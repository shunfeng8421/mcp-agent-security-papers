# Runtime-Confirmed Vulnerabilities in the AI-Agent Execution Control Plane

**Companion document to Paper #24.** All findings below were **runtime-confirmed** against real, isolated server instances (loopback, 127.0.0.1, no credentials, no real assets), following responsible disclosure. Each was reported to the owning repository.

*Last verified: 2026-10-03 (all four issues OPEN at owning repos).*

---

## 1. `truong51972/agent-exec-gateway` — Unauthenticated Arbitrary Code Execution + Policy Bypass
- **Issue:** [#2](https://github.com/truong51972/agent-exec-gateway/issues/2)
- **Root cause:** FastAPI app with **no authentication** on any `/api/*` route (no `Depends`, no middleware).
- **Runtime proof (loopback, no creds):**
  - `GET /api/hosts` → `200` (host inventory exposed)
  - `POST /api/policies` → `201` (attacker installs an allow-all policy)
  - `POST /api/approvals/{id}/approve` → self-approves any pending approval (defeats human-approval gate)
  - `POST /api/executions` → `200`, `decision=allow`; the SSH executor ran `echo __AEG_RCE_VERIFY_44e6f8__` on the configured remote host and the marker was echoed back in stdout.
- **Impact:** Any network-reachable attacker can register hosts, rewrite policy, self-approve, and execute arbitrary commands on remote targets. The component is *named* an "execution gateway" meant to gate code execution, yet ships as an unauthenticated execution primitive.
- **CWEs:** CWE-306 (missing auth), CWE-285 (policy bypass), CWE-78 (command injection / arbitrary exec).

## 2. `nagasai17bce-rgb/agent-sandbox-runtime` — Unauthenticated Arbitrary Python Execution
- **Issue:** [#1](https://github.com/nagasai17bce-rgb/agent-sandbox-runtime/issues/1)
- **Root cause:** `/v1/run` accepts arbitrary Python source and executes it, with **no authentication**; the provided `Dockerfile` binds the container to `0.0.0.0`.
- **Runtime proof (loopback, no creds):** `POST /v1/run` with a payload echoing a marker + hostname returned both in stdout.
- **Impact:** A network-reachable attacker can execute arbitrary Python inside the "sandbox" container — i.e., the thing named "sandbox" is an unauthenticated code-execution primitive.
- **CWEs:** CWE-306, CWE-94 (code injection).

## 3. `JestonBatson/governed-policy-engine` — Policy Rewrite + Approval Bypass
- **Issue:** [#1](https://github.com/JestonBatson/governed-policy-engine/issues/1)
- **Root cause:** No authentication on the control endpoints; `Dockerfile` binds `0.0.0.0`.
- **Runtime proof (loopback, no creds):**
  - `PUT /v1/policies` → `200` (attacker rewrites the entire policy set)
  - `POST /v1/approvals` → `200` (attacker self-grants any approval)
  - `GET /api/hosts` → `200` (config leakage)
- **Impact:** The engine *named* for "governance" lets any reachable attacker rewrite policy and grant approvals, defeating the human-approval gate.
- **Note:** `AUDIT_SIGNING_KEY` missing causes a `503` — that is a deployment requirement (fail-closed), not the vulnerability; the core write endpoints are the issue.
- **CWEs:** CWE-306, CWE-284 (improper access control), CWE-200 (info disclosure).

## 4. `ishakking642-eng/e2b-mcp-server` — Hardcoded API Key + Unauthenticated Services on 0.0.0.0
- **Issue:** [#3](https://github.com/ishakking642-eng/e2b-mcp-server/issues/3)
- **Root cause:** A hardcoded Daytona API key committed to source, plus MCP and guard HTTP services bound to `0.0.0.0` with no authentication.
- **Runtime proof:** Static (hardcoded secret in repo) + service startup on `0.0.0.0` without auth gate.
- **Impact:** Secret leakage (CWE-798) enables use of the Daytona account; unauthenticated services on 0.0.0.0 extend the reachable attack surface.
- **CWEs:** CWE-798 (hardcoded credential), CWE-306.

---

## Methodology (all findings)
1. **Source review** of freshly-published repos (0-star, created ~2026-10-01/02) named for agent execution control (gateway / sandbox / policy / approval).
2. **Runtime confirmation** on loopback: start the service in an isolated venv, bind to 127.0.0.1, drive the unauthenticated endpoints, verify a real effect (marker echoed / status 200 on write / config read-back).
3. **No credentials, no real assets, no external hosts** touched. Every test is reproducible from this document.
4. **Disclosure** via GitHub issue to the owning repo; state recorded on 2026-10-03.

## Control group (hardened, NOT vulnerable)
Seven implementations reviewed and found to enforce authentication / fail closed (paper #24 §control): web-agent-gateway (pairing token + dual approval), agent-ssh-gateway (access control + token store + audit log), Agent-Security-MCP-Gateway (CSRF + disposable approval + scope), AI-Agent-Security-Gateway (scope extractor + audit + evaluator), ib-mcp (fail-closed when no auth config), mcp-confirm (claim-once + request-state boundary), computeMCP (HMAC auth + per-client ACL + remote-container isolation).

## Why it matters
The four vulnerable components are **named for the exact security property they fail to provide**. "Gateway", "sandbox", "policy", and "approval" are trust anchors a deploying operator will assume are safe; each ships as an unauthenticated, reachable execution/control primitive. This is a *trust-inversion* pattern distinct from generic insecure code — the naming itself manufactures false assurance.
