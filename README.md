AI number 1 ne jo bola hai 
Comprehensive Project Test Report

Executive Summary

This trading bot dashboard project has been thoroughly analyzed for bugs, security vulnerabilities, and UI/UX issues. The analysis covers static code inspection, security audit, and architectural review. Critical findings include CORS misconfiguration, state persistence issues, UI/backend data mismatches, and command execution gaps.




1. CRITICAL SECURITY VULNERABILITIES

1.1 CORS Misconfiguration - WebSocket Origin Bypass

Severity: CRITICAL | File: gateway/ws.go (Line 17)

Go


CheckOrigin: func(r *http.Request ) bool { return true },



Issue: The WebSocket upgrader accepts connections from ANY origin. This allows:

•
Cross-site WebSocket hijacking (CSWSH)

•
Unauthorized dashboard access from malicious websites

•
Real-time data theft (portfolio, signals, mode state)

Impact: An attacker can:

1.
Host a malicious website

2.
Connect to the trading bot's WebSocket from any browser

3.
Receive live portfolio, mode, and signal data

4.
Send commands if CSRF tokens are not properly validated

Recommendation:

Go


CheckOrigin: func(r *http.Request ) bool {
    origin := r.Header.Get("Origin")
    return origin == "https://trusted-domain.com" || origin == "http://localhost:5173"
},






1.2 Insecure File Permissions - Operator State Persistence

Severity: HIGH | File: services/monitoring/operator.go (Line 277 )

Go


if err := os.WriteFile(tmp, raw, 0o644); err != nil {



Issue: Operator state (queue, approvals, notifications, timeline) is written with world-readable permissions (644). This file may contain:

•
Trade approval decisions

•
Command history

•
User actions

•
Sensitive operational state

Impact:

•
Any user on the system can read operator state

•
Audit trail can be compromised

•
Compliance violations (if regulated)

Recommendation:

Go


if err := os.WriteFile(tmp, raw, 0o600); err != nil {  // owner read/write only






1.3 Silent State Corruption Handling

Severity: MEDIUM | File: services/monitoring/operator.go (Lines 285-295)

Go


// load restores persisted operational state at startup. Any read/parse error
// is silently ignored → fresh state (never crashes on a corrupt file).
func (o *operator) load() {
    raw, err := os.ReadFile(o.persistPath)
    if err != nil {
        return  // silent ignore
    }
    var state persistedOperatorState
    if err := json.Unmarshal(raw, &state); err != nil {
        return  // silent ignore - no logging!
    }



Issue: Corrupted state files are silently discarded without any logging or alerting. This can mask:

•
Disk I/O failures

•
Permission issues

•
Data loss events

•
Security incidents

Impact:

•
Lost audit trail

•
Undetected system failures

•
Compliance violations

Recommendation: Log all errors:

Go


if err != nil {
    log.Printf("ERROR: failed to load operator state: %v", err)
    return
}






1.4 No Input Validation on Command Parameters

Severity: MEDIUM | File: services/monitoring/operator.go (Lines 657-680)

Go


func (o *operator) execute(item protocol.CommandQueueItem) protocol.CommandResult {
    kind := item.Kind
    symbol := symbolOf(item.Params)  // No validation!
    qty := int64(numOf(item.Params, "qty", 1))  // No bounds checking!



Issue: Command parameters are extracted without validation:

•
Symbol can be any string (no universe validation)

•
Quantity has no upper bounds

•
No type checking on params

Impact:

•
Invalid trades could be placed

•
Excessive order quantities possible

•
Injection attacks via symbol field

Recommendation: Validate all parameters:

Go


symbol := strings.ToUpper(symbolOf(item.Params))
if !isValidSymbol(symbol) {
    return protocol.CommandResult{OK: false, Reason: "INVALID_SYMBOL"}
}
if qty <= 0 || qty > MAX_ORDER_QTY {
    return protocol.CommandResult{OK: false, Reason: "INVALID_QUANTITY"}
}






2. CRITICAL UI/BACKEND MISMATCHES

2.1 Phase State Mapping Bug

Severity: HIGH | File: dashboard/src/mission/MissionProvider.tsx (Lines 316-325)

TypeScript


function toPhase(p: string): SystemPhase {
  switch (p) {
    case "OFF": return "OFF";
    case "STARTING": return "STARTING";
    case "READY": return "READY";
    case "RUNNING": return "READY";  // BUG: RUNNING collapsed to READY!
    case "PAUSED": return "PAUSED";
    case "EMERGENCY": return "EMERGENCY";
    default: return "OFF";
  }
}



Issue: Backend state RUNNING is mapped to frontend READY, collapsing two distinct states into one.

Impact:

•
UI shows "READY" when system is actually "RUNNING"

•
Users cannot distinguish between ready-to-start and actively-trading states

•
Operator may attempt to start an already-running system

•
Confusion about actual system status

Recommendation: Use distinct phase names:

TypeScript


case "RUNNING": return "RUNNING";






2.2 Hardcoded Symbol Universe Mismatch

Severity: MEDIUM | File: dashboard/src/workspace/SymbolPane.tsx (Lines 7-8)

TypeScript


const SYMBOLS = ["RELIANCE", "TCS", "HDFCBANK", "INFY", "WIPRO", "HDFC", "ICICIBANK"];



Issue: Dashboard has hardcoded symbol list, but backend supports dynamic symbols from ingestion service.

Impact:

•
UI cannot select symbols not in hardcoded list

•
Backend may support symbols UI doesn't show

•
Operator cannot trade symbols available in backend

•
Inconsistent with system design

Recommendation: Fetch symbols dynamically:

TypeScript


const [symbols, setSymbols] = useState<string[]>([]);
useEffect(() => {
  fetch("/api/symbols").then(r => r.json()).then(d => setSymbols(d.data));
}, []);






2.3 WebSocket Snapshot Data Underutilization

Severity: MEDIUM | File: gateway/ws.go (Lines 81-135) & dashboard/src/mission/MissionProvider.tsx

Issue: WebSocket broadcasts only 4 fields (portfolio, mode, scan, signals) every 5 seconds, but dashboard polls /api/operator/workspace separately every 4 seconds.

Go


// Gateway broadcasts only these 4 fields
out := map[string]any{
    "portfolio": nil,
    "mode": nil,
    "scan": nil,
    "signals": nil,
}



Impact:

•
Redundant network traffic (polling + WebSocket)

•
Stale operator workspace data (4s poll vs 5s broadcast)

•
Inconsistent data sources within same pane

•
Higher latency for operator actions

Recommendation: Either:

1.
Broadcast full workspace snapshot over WebSocket, OR

2.
Remove polling and rely on WebSocket




3. LOGIC BUGS AND DESIGN ISSUES

3.1 Command Kind Declared But Not Implemented

Severity: MEDIUM | File: services/monitoring/operator.go (Lines 84-153)

Issue: The knownCommandKinds set includes commands that are accepted but not fully implemented:

Go


"start_ai", "stop_ai", "pause_ai", "resume_ai",  // Implemented
"risk_reset", "approve_trade", "reject_trade",   // Implemented
"restart_bot", "reconnect_data", "reconnect_broker",  // Implemented
"pause_analysis", "resume_analysis",  // Implemented
"clear_queue", "refresh_dashboard", "export_logs"  // Implemented



But in the execute() function (lines 820-920), some commands have stub implementations:

•
"start_ai" (line 820): Just calls go o.runBoot() without validation

•
"restart_bot" (line 868): Simulates restart with hardcoded 800ms delay

•
"reconnect_data" (line 878): Simulates reconnect with hardcoded 900ms delay

Impact:

•
Commands appear to work but don't actually execute

•
Operator may think system is reconnected when it's not

•
No real broker/data reconnection occurs

•
Misleading success responses

Recommendation: Either implement fully or mark as "simulation-only"




3.2 Control Mode Not Enforced at Gateway

Severity: MEDIUM | File: gateway/main.go (Lines 288-310)

Go


// handleOrderPlacement enforces the operator control mode at the gateway for
// buy intents (section 11): in SEMI_AUTO / MANUAL the backend refuses any
// autonomous buy unless the operator explicitly overrode it.
func (g *gateway) handleOrderPlacement(w http.ResponseWriter, r *http.Request ) {
    // ... code ...
    if strings.EqualFold(body.Side, "buy") {
        mode, err := g.fetchControlMode()
        if err == nil && (mode == "SEMI_AUTO" || mode == "MANUAL") && !body.OperatorOverride {
            // Only rejects if override is false
            httputil.WriteErr(w, r, serviceName, "ORDER_REJECTED_CONTROL_MODE", http.StatusConflict )
            return
        }
    }
    g.forwardPost(w, r, "order", "/orders", raw)
}



Issue: The control mode check only applies to BUY orders. SELL orders bypass this check entirely.

Impact:

•
Operator in MANUAL mode can force-sell without approval

•
Inconsistent control mode enforcement

•
Risk management bypass for sell orders

Recommendation:

Go


if (strings.EqualFold(body.Side, "buy") || strings.EqualFold(body.Side, "sell")) {
    // Apply control mode check to both buy and sell
}






3.3 Sentiment Cap Configuration Not Validated

Severity: LOW | File: services/analysis/main.py (Lines 36-50)

Python


def _load_config():
    global _SENTIMENT_CAP
    for base in (os.environ.get("CONFIG_DIR", "."), "."):
        path = os.path.join(base, "config", "defaults", "risk.yaml")
        if os.path.exists(path):
            with open(path) as f:
                raw = yaml.safe_load(f) or {}
            ss = (raw.get("signal_score") or {})
            _SENTIMENT_CAP = float(ss.get("sentiment_hard_cap", 0.2))
            return
    _SENTIMENT_CAP = 0.2



Issue: No validation that sentiment_hard_cap is in valid range [0.0, 1.0].

Impact:

•
Invalid cap values could break signal scoring

•
Negative or >1.0 values would skew signal calculations

Recommendation:

Python


cap = float(ss.get("sentiment_hard_cap", 0.2))
_SENTIMENT_CAP = max(0.0, min(1.0, cap))  # Clamp to [0, 1]






4. DEPENDENCY VULNERABILITIES

4.1 NPM Dependencies - High Severity Issues

Severity: HIGH | File: dashboard/package.json

Vulnerabilities Found:

1.
esbuild ≤0.24.2 (Moderate)

•
CSWSH vulnerability: allows any website to send requests to dev server

•
Affects: vite ≤6.4.2



2.
nanoid <3.3.17 (High)

•
Custom generators can loop indefinitely when size is zero

•
DoS vulnerability



Recommendation:

Bash


npm audit fix
npm install esbuild@latest nanoid@latest






5. PERFORMANCE & SCALABILITY ISSUES

5.1 WebSocket Broadcast Inefficiency

Severity: LOW | File: gateway/ws.go (Lines 50-79)

Go


// broadcastLoop pushes a live snapshot to all WebSocket clients on an interval.
func (g *gateway) broadcastLoop(ctx context.Context) {
    ticker := time.NewTicker(5 * time.Second)
    defer ticker.Stop()
    for {
        select {
        case <-ctx.Done():
            return
        case <-ticker.C:
            snap := g.snapshot()
            if snap == nil {
                continue
            }
            b, _ := json.Marshal(snap)  // Marshals for EVERY client
            g.wsMu.Lock()
            for c := range g.wsClients {
                // Sends same JSON to all clients
                if c.conn == nil {
                    continue
                }
                c.wsMu.Lock()
                if err := c.conn.WriteMessage(websocket.TextMessage, b); err != nil {
                    c.wsMu.Unlock()
                    continue
                }
                c.wsMu.Unlock()
            }
            g.wsMu.Unlock()
        }
    }
}



Issue: JSON is marshaled once but sent to all clients. If a send fails, the error is silently ignored.

Impact:

•
Stale connections can accumulate

•
Memory leaks if clients disconnect ungracefully

•
No monitoring of broadcast failures

Recommendation: Remove dead connections:

Go


if err := c.conn.WriteMessage(websocket.TextMessage, b); err != nil {
    c.conn.Close()
    g.wsMu.Lock()
    delete(g.wsClients, c)
    g.wsMu.Unlock()
}






6. MISSING ERROR HANDLING

6.1 Unhandled HTTP Request Errors

Severity: MEDIUM | File: gateway/main.go (Lines 269-282)

Go


func (g *gateway) forwardMode(w http.ResponseWriter, r *http.Request, method, payload string ) {
    target, err := url.Parse(g.monitoringURL)
    if err != nil {
        httputil.WriteErr(w, r, serviceName, reasoncodes.ServiceUnhealthy, http.StatusBadGateway )
        return
    }
    var body io.Reader
    if payload != "" {
        body = bytesReader(payload)
    }
    req, _ := http.NewRequestWithContext(r.Context( ), method, target.String()+"/internal/mode/change", body)
    // ^ ignores error!
    req.Header.Set("Content-Type", "application/json")
    client := &http.Client{Timeout: 5 * time.Second}
    resp, err := client.Do(req )
    if err != nil {
        httputil.WriteErr(w, r, serviceName, reasoncodes.ServiceUnhealthy, http.StatusBadGateway )
        return
    }
    defer resp.Body.Close()
    data, _ := io.ReadAll(resp.Body)  // ^ ignores error!
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(resp.StatusCode)
    _, _ = w.Write(data)  // ^ ignores error!
}



Issue: Multiple error returns are ignored with _.

Impact:

•
Silent failures in mode changes

•
Incomplete responses sent to client

•
Difficult debugging




7. TESTING GAPS

7.1 No Unit Tests for Critical Functions

Severity: MEDIUM

Missing Tests:

•
signal.compute_signal() - Core signal calculation

•
operator.execute() - Command execution logic

•
gateway.snapshot() - WebSocket data assembly

•
Risk evaluation functions

Recommendation: Add pytest/Go test coverage for:

•
Signal scoring edge cases

•
Command execution with invalid parameters

•
Approval state transitions

•
Control mode enforcement




8. DEPLOYMENT & CONFIGURATION ISSUES

8.1 Rust Build Fails - Toolchain Version Mismatch

Severity: HIGH | File: go.mod

Plain Text


go 1.25  // Requires Go 1.25, not available in standard repos



Issue: Project requires Go 1.25 which is not released/available in standard package managers.

Impact:

•
Cannot build on standard Ubuntu systems

•
Blocks deployment

•
Rust services cannot be compiled

Recommendation: Use stable Go version (1.22 or 1.23)




9. AUDIT & COMPLIANCE ISSUES

9.1 No Audit Trail for State Mutations

Severity: MEDIUM | File: services/monitoring/operator.go

Issue: Operator state changes (queue modifications, approvals, etc.) are persisted but not all mutations are audited.

Impact:

•
Compliance violations (if regulated)

•
Cannot trace who made which changes

•
Audit trail gaps




10. UI/UX ISSUES

10.1 No Loading States for Async Operations

Severity: LOW | File: dashboard/src/mission/MissionProvider.tsx

Issue: Workspace polling happens every 4s but no loading indicators shown during fetches.

Impact:

•
UI appears frozen during slow network

•
Users don't know if action was received

•
Poor user experience




SUMMARY TABLE

Category
Count
Severity
Critical Security
2
CRITICAL
High Security
2
HIGH
Medium Issues
8
MEDIUM
Low Issues
3
LOW
Total
15
-







RECOMMENDED FIXES (Priority Order)

1.
IMMEDIATE (Critical):

•
Fix WebSocket CORS to whitelist specific origins

•
Change operator state file permissions to 0o600

•
Add input validation to command parameters



2.
HIGH PRIORITY:

•
Fix phase state mapping (RUNNING → READY bug)

•
Enforce control mode for SELL orders

•
Update Go version requirement to 1.22/1.23

•
Fix npm vulnerabilities (esbuild, nanoid)



3.
MEDIUM PRIORITY:

•
Implement proper error logging for state corruption

•
Add full command implementations or mark as stubs

•
Fetch symbols dynamically instead of hardcoding

•
Consolidate WebSocket vs polling data sources



4.
LOW PRIORITY:

•
Add loading states to UI

•
Improve error handling in gateway

•
Add unit tests for critical functions






CONCLUSION

The project has a solid architecture but contains several critical security and logic issues that must be addressed before production deployment. The most urgent fixes are the CORS misconfiguration and state file permissions, which could lead to unauthorized access and data breaches.







AI 2 ne jo bola ye wo hai 
FULL TEST REPORT — AI Stock Trading Bot (paper V1)
Date: 2026-08-08 | Env: sandbox, all 6 services + dashboard built from source and run live Build: Go OK, Rust release OK, Python OK, dashboard npm OK (0 build errors)

1. Frontend <-> Backend connectivity
22/22 endpoints used by dashboard/src/api.ts returned 200 with valid JSON envelope {ok,data,reason_code}: PASS
Vite proxy /api + /ws -> gateway: PASS (dashboard served 200, proxied API 200)
WebSocket /ws: 8 frames in 32s, gaps 5.00s steady, ~17KB snapshot with config_hash, config_version, mode, portfolio, scan, signals(50): PASS
Gaps: /api/operator/events and /api/operator/timeline return data: null (frontend types say array) -> .map() risk if UI not guarded.
2. Backend features visible on frontend
PASS: boot sequence (10 stages all ok -> RUNNING), control-mode AUTO/SEMI_AUTO/MANUAL, kill-switch, mode 3/2/1 changes, diagnostics tiers 1-3, audit trail, freshness, hard-rules, signals, risk heat, portfolio/positions/orders, operator queue/notifications/approvals, approvals+trades stale flags. Order validation: negative qty, zero qty, unknown symbol, bad side, fractional qty -> all rejected with ORDER_REJECTED_VALIDATION: PASS Kill-switch: buys blocked (ORDER_REJECTED_KILL_SWITCH), exits allowed, order/monitoring state now in sync: PASS (old desync bug fixed) Invalid operator command -> 422 INVALID_COMMAND synchronously: PASS (old async bug fixed) Partial fill transparency: requested_qty=100000, filled_qty=27, partial=true: PASS (fixed)

3. Bugs still open
HIGH-1 No mark-to-market: after 25s of live ticks, position last_price frozen at fill price, pnl=0.0, unrealized_pnl=0.0, equity effectively static. Live PnL on the dashboard is wrong. HIGH-2 No persistence: order service restart wiped everything — cash back to 1,000,000, positions 0, orders 113 -> 0. Nothing recoverable after a crash. HIGH-3 Risk cap breached under concurrency: 100 parallel buys -> portfolio heat 6.47% vs hard limit 6.00%. Check-then-fill is not atomic; limits can be exceeded in a burst. MED-4 Stale flags misreport: while order service was down, workspace trades.stale stayed false. MED-5 Boot hard-fails to OFF if gateway probe fails; only reason shown is "probe failed or timed out". LOW-6 RISK_NO_SIGNAL is returned for neutral-direction symbols with the same 404-ish shape as errors.

4. Security tests (10 pass / 10 fail)
PASS: no wildcard CORS; security headers present (X-Content-Type-Options, X-Frame-Options, Referrer-Policy, CSP); path traversal, encoded traversal, SQLi-style and XSS payloads all safely rejected; 5MB body -> 413; malformed JSON -> 400; no secrets leaked in /api/config. FAIL (critical): NO AUTHENTICATION ANYWHERE — anyone who can reach the gateway can read the portfolio, place orders, change mode, and toggle the kill-switch. Same for the WebSocket. FAIL: no rate limiting — 200 requests in 0.2s, all 200. FAIL: internal services bind 0.0.0.0 and are directly reachable (8081-8085). Direct POST to order service :8084 filled a trade while control-mode was MANUAL, i.e. gateway-level controls are fully bypassable. FAIL: no TLS; plain HTTP only.

5. Load / power test
Reads: 600 requests / 20 threads -> 626 rps, p50 13ms, p95 31ms, max 574ms, 0 errors.
Orders: 100 concurrent -> 133 ops/s, p50 21ms, p95 230ms, 0 crashes; outcomes 36 filled, 63 RISK_HEAT_MAX_REACHED, 1 RISK_NO_SIGNAL.
Stability: no service crashed under load; WS cadence stayed 5s.
6. Verdict — real-life readiness
Excellent as a paper-trading demo/simulator. NOT ready for real money. Blockers before live: (1) authentication + TLS + bind internal services to 127.0.0.1, (2) persistence with crash recovery, (3) atomic risk accounting so heat/limits cannot be breached concurrently, (4) real mark-to-market, (5) rate limiting, (6) real broker adapter + reconciliation.
