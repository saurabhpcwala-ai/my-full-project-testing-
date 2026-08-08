Thought
User's tokens are almost exhausted. They want to hand off to another AI. They need a detailed document describing the project state and what remains to be done — the remaining parts (2, 3, 4, 6) of the fix plan — so they can paste it into another AI.

The user wants me to create a handoff document with details. The FIX_PLAN.md already exists and is the source of truth. But the user wants something to paste into another AI with details.

Let me think about what's most useful. The FIX_PLAN.md already contains all parts. But the user wants a handoff doc they can paste into another AI. I should create a comprehensive handoff document that includes:
1. Project context (what it is, stack, ports, commands)
2. Current state (what's been done: PART 1, PART 5 complete + setup + testing)
3. What remains (PART 2, 3, 4, 6 + by-design items)
4. Golden rules
5. Critical context (code locations, line numbers, key decisions)
6. Verification commands

Let me create a HANDOFF.md file in /workspace. This is a doc creation — the user explicitly asked me to make something ("mujhe kuch banake do"). So creating a documentation file is appropriate here.

Let me also consider: the user wants the doc to be paste-able into another AI. So it should be self-contained, in a mix of Hinglish/English similar to FIX_PLAN.md style, detailed.

Let me check current state first — what's verified, what's live, etc. I have all this in context. Let me also grab a quick snapshot of current state to make it accurate.

Let me create a comprehensive handoff document. I should make it thorough since it's going to another AI. Let me structure it:

1. PROJECT OVERVIEW (what this is)
2. STACK + PORTS + COMMANDS
3. PROGRESS SO FAR (what's done and verified)
4. REMAINING PARTS (2, 3, 4, 6) with full details
5. GOLDEN RULES
6. CURRENT LIVE STATE
7. KEY CODE LOCATIONS
8. BY-DESIGN (do not fix)
9. VERIFICATION COMMANDS

The FIX_PLAN.md already has most of this. But the handoff doc should be standalone and paste-ready. Actually, the smartest approach: point the other AI to FIX_PLAN.md AND summarize everything in the handoff doc itself, so it's self-contained.

Let me write HANDOFF.md in /workspace. Keep it detailed but organized. The user wants "details ke saath".

Let me quickly verify current live state for accuracy before writing.
08-08 14:08:11
Token bachane ke liye complete handoff document banata hoon jo dusre AI ko paste kar sakte ho. Pehle current state quickly capture karta hoon.

08-08 14:08:11

Run command "for p in 8080 8081 8082 8083 8084 8085 5173; do echo -n "$p:"; curl -s -o /dev/null -w "%{http_code} " http://localhost:$p/health 2>/dev/null || echo -n "DOWN "; done; echo; curl -s http://localhost:8080/api/operator/workspace | python3 -c " import json,sys d=json.load(sys.stdin)['data'] print('phase:',d['system']['phase'],'| mode:',d['system']['control_mode'],'| kill_switch:',d['system'].get('kill_switch')) "; echo "---git---"; cd /workspace && git log --oneline -3 2>/dev/null; git status --porcelain 2>/dev/null | head -3"
$ for p in 8080 8081 8082 8083 8084 8085 5173; do echo -n "$p:"; curl -s -o /dev/null -w "%{http_code} " http://localhost:$p/health 2>/dev/null || echo -n "DOWN "; done; echo; curl -s http://localhost:8080/api/operator/workspace | python3 -c "
import json,sys
d=json.load(sys.stdin)['data']
print('phase:',d['system']['phase'],'| mode:',d['system']['control_mode'],'| kill_switch:',d['system'].get('kill_switch'))
"; echo "---git---"; cd /workspace && git log --oneline -3 2>/dev/null; git status --porcelain 2>/dev/null | head -3
8080:200 8081:200 8082:200 8083:200 8084:200 8085:200 5173:200 
phase: RUNNING | mode: AUTO | kill_switch: False
---git---
bf37c23 Initial commit
?? FIX_PLAN.md
?? README.md
?? TEST_REPORT.md
08-08 14:08:18
Thought
Now let me create a comprehensive handoff document. The user wants a detailed document they can paste into another AI. Let me create HANDOFF.md (or EXT_AI_HANDOFF.md) that contains:

1. What the project is
2. Current progress (parts done: PART 1, PART 5 + testing)
3. What remains (PART 2, PART 3, PART 4, PART 6 + by-design)
4. How to start the stack, ports, commands
5. Golden rules
6. Detailed instructions for each remaining part with code locations

The FIX_PLAN.md already has most of this. The best handoff approach: point the other AI to FIX_PLAN.md, TEST_REPORT.md, and give a comprehensive paste-ready document with all context.

Let me write a detailed handoff doc. It should be self-contained since it will be pasted into another AI. I'll write it in Hinglish like FIX_PLAN.md so it's consistent, but include all technical details.

Let me create the file.
08-08 14:08:19
Ab complete handoff document banata hoon — self-contained, paste-ready, Hinglish + technical details ke saath.

08-08 14:08:19

Edit file "/workspace/HANDOFF_EXT_AI.md"
HANDOFF_EXT_AI.md
# HANDOFF — Is project ko aage badhane wale AI ke liye COMPLETE CONTEXT
 
> **Yeh file kisi dusre AI session mein paste karo.** Isme sab kuch hai: project kya hai, kya ho chuka hai, kya karna baki hai (details ke saath), kaise verify karna hai, aur golden rules.
>
> Saath mein ye bhi paste karo (same directory mein hain):
> - `/workspace/FIX_PLAN.md` — **SOURCE OF TRUTH**. Har part ki detailed fix instructions + STATUS TRACKER + MASTER LIST (11 problems). Har part shuru karne se pehle ye padho.
> - `/workspace/TEST_REPORT.md` — pichli testing session ka full evidence (jo verified hai wo DOBARA mat karo).
 
---
 
## 1. PROJECT KA OVERVIEW (ye kya hai)
 
AI Stock Trading Bot — **"share-anyone"** (GitHub repo `saurabhpcwala-ai/share-anyone`). Multi-service paper-trading system:
- Signals generate karta hai (technical indicators + sentiment) → risk engine decide karta hai (allow/reduce/reject) → order service execute karta hai (paper orders) → monitoring operator layer (approvals/queue/notifications) → React dashboard.
- **Yeh PAPER/SIMULATION hai** — koi real money nahi.
 
## 2. STACK + PORTS
 
| Port | Service | Tech | Directory |
|------|---------|------|-----------|
| 8080 | gateway (API + WS + control-mode gate + kill-switch proxy) | Go | `/workspace/gateway` |
| 8081 | ingestion (market data ticks) | Go | `/workspace/services/ingestion` |
| 8082 | analysis (signals) | Python | `/workspace/services/analysis` |
| 8083 | risk (decision engine) | Rust | `/workspace/services/rust/risk` |
| 8084 | order (positions/cash/orders/kill-switch) | Rust | `/workspace/services/rust/order` |
| 8085 | monitoring (operator layer: queue/approvals/notifications/audit) | Go | `/workspace/services/monitoring` |
| 5173 | dashboard (React/Vite) | TS | `/workspace/dashboard` |
 
**Shared protocol/structs:** `/workspace/internal/protocol/protocol.go` (Go) aur `/workspace/services/rust/core/src/protocol.rs` (Rust). Reason codes: `/workspace/internal/reasoncodes/reasoncodes.go`.
 
## 3. CORE COMMANDS (BAAR-BAAR CHAHIYE)
 
```bash
# Rust PATH (cargo nahi milta to)
export PATH="$HOME/.cargo/bin:$PATH"
 
# Full build (Go + Rust + Python check + dashboard)
cd /workspace && ./scripts/build-all.sh
 
# Sirf Rust build/test
cd /workspace/services/rust && cargo build --release && cargo test --release
 
# Stack start (BACKGROUND terminal se, ~55s)
cd /workspace && ./scripts/run-all.sh
 
# Boot system (start ke baad zaroori — phase OFF → STARTING → RUNNING)
curl -X POST http://localhost:8080/api/operator/boot/start -d '{}'
 
# Health check
for p in 8080 8081 8082 8083 8084 8085; do curl -s -o /dev/null -w "$p %{http_code}\n" http://localhost:$p/health; done
 
# Place paper order (partial fill test karne ke liye bada qty do)
curl -s -X POST http://localhost:8084/orders -H "Content-Type: application/json" -d '{"symbol":"ADANIPORTS","side":"buy","type":"market","qty":100000}'
 
# Workspace state (operator layer ka snapshot)
curl -s http://localhost:8080/api/operator/workspace
```
 
**IMPORTANT environment vars jo order/monitoring/gateway restart pe chahiye:**
```bash
export CONFIG_DIR=/workspace; export MARKET_ALWAYS_OPEN=1
export MONITORING_URL=http://localhost:8085; export INGESTION_URL=http://localhost:8081
export ANALYSIS_URL=http://localhost:8082; export RISK_URL=http://localhost:8083
export ORDER_URL=http://localhost:8084
```
(⚠️ `MARKET_ALWAYS_OPEN=1` ke bina buy orders `ORDER_REJECTED_MARKET_CLOSED` reject honge.)
 
**Background terminal tooling:** Servers ko background terminal mein chalao (`run-all.sh` background mein). Pehle wala run-all abhi bhi chalu hai (sab ports 200). Agar kisi service ka binary update karo to us service ko kill + restart karo (background terminal kill/create). **`pkill`/`killall` use mat karo.**
 
## 4. PROGRESS SO FAR (kya ho chuka hai — VERIFIED)
 
| Part | Status | Kya kiya |
|------|--------|----------|
| Testing session | DONE | 40+ tests (README) + 17 new tests (Python analysis 27/27, config, kill-switch+mode, concurrency, emergency_stop, WS cadence). Full evidence `TEST_REPORT.md`. |
| Heat FP boundary bug (#12) | FIXED | `risk/src/state.rs` +1e-6 epsilon (lines 260, 369). |
| **PART 1** — silent partial fill (#1,#10) | ✅ DONE | `Order` struct mein `requested_qty`/`filled_qty`/`partial` (`core/src/protocol.rs`), 3 construction sites (`order/src/state.rs`), frontend (types.ts, PortfolioView, ObservabilityPane, BrainView). Verified: buy 100000 → requested 100000/filled 80/partial true. |
| **PART 5** — operator fixes (#4,#6,#11) | ✅ DONE | 5a invalid command (`INVALID_COMMAND` + `knownCommandKinds`), 5b kill-switch desync (hydration else-branch + frontend operator-command path), 5c separate stale states (`DataChunk[T]`). Verified live. |
 
**FIX_PLAN.md STATUS TRACKER:** PART 1 = `[x]`, PART 5 = `[x]`. Baki (PART 2,3,4,6) = `[ ]`.
 
## 5. JO PARTS BAKI HAIN (aage inhe karo)
 
Order of priority: **PART 3 → PART 2 → PART 4 → PART 6** (sab HIGH pehle, phir MED/LOW). Har part ke FULL details `FIX_PLAN.md` mein hain — **har part ke section ko dhyan se padho, actual code inspect karo, assume mat karo.**
 
### PART 3 — HIGH — Persistence: order/portfolio state (order service → disk file)
- **Problem:** Order service restart → positions [], cash 1M, orders 0 (sab in-memory).
- **Files:** `/workspace/services/rust/order/src/main.rs` + `state.rs`
- **Kya karna:** JSON snapshot file (e.g. `/workspace/order_state.json`) par order/position CHANGE ke baad `persist()` (atomic write: temp+rename). Startup pe load (fail → silent fresh). **Kill-switch ko is file mein mat persist karo — wo by-design safety feature hai (PART 3 mein CLEAR STATEMENT hai).**
- **Verify:** buy → file bane → service kill+restart → positions/orders/cash restore.
- `cargo build --release` + `cargo test`.
 
### PART 2 — HIGH — Mark-to-market / PnL accuracy
- **Problem:** Position `last_price` sirf buy/sell fill pe set hota hai, live price se kabhi update nahi → `pnl` hamesha 0.0, equity frozen.
- **Files:** `/workspace/services/rust/order/src/state.rs` + `main.rs` (periodic task)
- **Kya karna:** `reprice_positions()` async fn (har open position ka `last_price(&symbol)` se live price fetch), periodic (~5-10s) call. **Long/short PnL formula alag-alag** (FIX_PLAN.md line 108+ dekho). Repricing memory mein; file I/O har tick par NAHI.
- **Verify:** buy TCS, wait 30-60s → `/api/portfolio` mein `last_price` live, `unrealized_pnl != 0`.
 
### PART 4 — HIGH — Persistence: operator state (monitoring → disk file)
- **Problem:** Monitoring operator state (queue/approvals/notifications/events/timeline) restart pe wipe.
- **Files:** `/workspace/services/monitoring/operator.go` + `operator_http.go` + `main.go`
- **Kya karna:** JSON file (`/workspace/operator_state.json`) par mutate-operation ke baad persist (debounce ~500ms OK). Startup load. **Mode/phase/kill-switch persist mat karo — by design** (system boot pe OFF/READY start hona chahiye).
- **Verify:** SEMI_AUTO mein approval pending → restart → approvals wapas.
 
### PART 6 — LOW/MED — Perf: WS broadcast + mode hysteresis
- **6a:** `gateway/ws.go` snapshot() 4 serial HTTP calls → parallel karo (goroutines), partial-data safe (failed keys null/stale marker). Zero-value data valid nahi broadcast karna.
- **6b:** `monitoring/main.go` metricsLoop mode auto-change (3→1) → operator override ke baad `manualOverrideUntil` timestamp (~10 min) se suppress karo.
- **Verify:** WS gaps ~5s consistent; mode override ke baad auto-change suppressed.
 
## 6. BY DESIGN (FIX MAT KARO — koi problem nahi)
 
- **#8 Kill-switch reboot pe persist** — safety feature, intentional. Kabhi remove/disable mat karo.
- **#9 Approvals sirf SEMI_AUTO/MANUAL** — AUTO mein autonomy, by design.
- **RoR reduce partial** — PART 1 mein visibility se handled; logic change mat karo.
- **Heat FP** — already fixed.
 
## 7. CURRENT LIVE STATE (is handoff ke time)
 
- Stack UP: sab ports 200 (8080-8085 + dashboard 5173).
- System: **phase=RUNNING, mode=AUTO, kill-switch=RELEASED (False)**.
- Order service mein koi position nahi (pehle tests ke baad restart hua tha).
- Dashboard preview: `https://5173-92bfc68a2b374c8a.monkeycode-ai.live`
 
## 8. GOLDEN RULES (hamesha yaad rakho)
 
1. **Project ko kabhi TODO/BEECH MAI ATKA hua mat chhodo.** Har part = build + test + working state.
2. **DOBARA TEST MAT KARO** jo `TEST_REPORT.md` mein verified hai. Sirf apne part ki changes verify karo.
3. **Doosre parts ke files mat chhedo** (har part ki apni file list hai).
4. Har part ke baad `./scripts/build-all.sh` + relevant tests.
5. Part complete hone pe FIX_PLAN.md STATUS TRACKER mein `[ ]` → `[x]` karo + implementation notes add karo.
6. **Golden rule (har part se pehle):** "Inspect the existing structs, handlers, locks, and call paths. Do not assume field names or endpoint behavior from the plan."
7. Backward compat: purane response fields mat hatao; agar shape change karo to saare consumers update karo (grep karke dhundho).
8. Long-running servers ko background terminal mein chalao; `pkill`/`killall` prohibited.
 
## 9. IMPORTANT GOTCHAS (dusre AI ne seekha)
 
- **`Order` struct Rust mein `core/src/protocol.rs` mein hai** (order crate mein nahi) — shared crate. Usme fields add karne par risk crate bhi build hota hai (risk `Order` construct nahi karta, safe).
- **Workspace response shape ab badal gaya hai:** `trades`/`approvals` ab array nahi, `{items, stale, error}` objects hain (`DataChunk[T]`). Frontend mein `.items` se access karo. `ObservabilityPane`/`SymbolPane`/`MissionProvider` already update ho chuke hain.
- **Kill-switch frontend path ab operator command hai** (`/api/operator/command {"kind":"kill_switch","params":{"active":...}}`), direct `/kill-switch` POST nahi. GET `/api/kill-switch` abhi bhi valid hai (read).
- **Invalid command ab HTTP 422 + `INVALID_COMMAND`** return karta hai (enqueue nahi hota). Valid kinds ki list `operator.go` mein `knownCommandKinds` map hai.
- **Market closed:** buy test karne se pehle `MARKET_ALWAYS_OPEN=1` env var confirm karo.
- **Kill-switch release ke baad system EMERGENCY phase mein ho to** `{"kind":"resume_ai"}` command se RUNNING pe wapas lao.
- Audit/trades data monitoring mein order service se hydrate hota hai — order service down ho to `trades.stale=true` milega (by design ab).
 
08-08 14:08:50
Ho gaya yaar. Handoff document ready hai.

/workspace/HANDOFF_EXT_AI.md — paste-ready handoff
Isme sab kuch detail ke saath hai:

Project overview — kya hai, kaunsa repo
Stack + ports table — 7 services, tech, directories
Core commands — build/start/boot/health/order + zaroori env vars (MARKET_ALWAYS_OPEN=1 gotcha included)
Progress so far — testing + Heat fix + PART 1 + PART 5 (sab verified)
Baki parts — PART 3, 2, 4, 6 details ke saath (priority order), verify criteria
By design — kya fix nahi karna
Current live state — RUNNING, AUTO, kill-switch released
Golden rules — 8 rules
Gotchas — 7 critical learnings (Order struct core mein hai, workspace shape ab {items,stale,error}, kill-switch path, etc.)
Dusre AI ko kya paste karna hai: /workspace/HANDOFF_EXT_AI.md + /workspace/FIX_PLAN.md + /workspace/TEST_REPORT.md. Teen files ka path batao ya content paste karo.

Note: pehle se verified tests dobara mat karna — handoff file mein clear likha hai. Sab kuch saved hai, token khatam hone pe chinta nahi.
