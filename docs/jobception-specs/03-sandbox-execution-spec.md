# Sandbox Execution Spec

- **Version:** 0.2
- **Status:** Draft
- **Owner:** Tech Lead / Security
- **Last Updated:** 2026-04-10
- **Depends On:** 03-system-architecture.md, 04-security-spec.md
- **Referenced By:** 03-evaluation-spec.md, 07-containerization-spec.md, 07-scalability-spec.md

## Purpose
Define the runtimes in which candidates complete bounty work. The sandbox layer is product-critical (it IS the bounty experience), security-critical (we run untrusted candidate code), and ops-critical (it's the highest-spend, spikiest workload). It deserves its own spec.

JobCeption uses **three complementary runtimes** depending on bounty type, all wrapped behind a single internal service (`sandbox-orchestrator`) so the rest of the platform sees one abstraction.

## Runtime Selection

| Bounty Type | Runtime | Rationale |
|---|---|---|
| Code challenge (algo, data structures, single-file solutions) | **Judge0** | Fast, sandboxed, 60+ languages, mature, self-hostable, free OSS. Industry standard for competitive coding platforms. |
| Project / multi-file build / "real dev environment" | **GitHub Codespaces** OR **StackBlitz SDK** | Full IDE in browser, zero setup, scales to many concurrent users. Codespaces for backend-heavy work, StackBlitz for frontend/Node. |
| Custom environment (recruiter uploads a Docker template) | **Docker on Kubernetes** (in-house orchestrator) | Maximum flexibility for niche stacks, recruiter-defined environments. |

The orchestrator picks the runtime based on the bounty's `runtime_kind` field, set when the bounty template is created.

## Runtime 1: Judge0 (MVP default)

### What it is
Judge0 is an open-source online judge. We self-host it on EKS. Candidates submit code via the orchestrator, Judge0 compiles and runs the code in an isolated sandbox against hidden test cases, and returns pass/fail plus performance metrics (time, memory).

### Why it's the MVP default
- **Mature isolation:** Judge0 runs each submission in an isolated cgroup/namespace with strict resource limits.
- **60+ languages out of the box:** C, C++, Java, Python, Go, Rust, JS, etc.
- **Hidden test cases:** the orchestrator stores test cases server-side; only pass/fail and performance are returned to the candidate.
- **Self-hostable, free, OSS:** matches the platform's cost philosophy.
- **Fast:** typical execution in <2s for most code submissions.

### Architecture
```
Candidate Browser
   ⇅ WebSocket (editor events, anti-cheat telemetry)
Sandbox Orchestrator (stateful, K8s-hosted)
   ⇅ HTTP
Judge0 cluster (self-hosted on EKS)
   ↳ Judge0 Server (REST API)
   ↳ Judge0 Workers (isolated execution, autoscaled)
   ↳ Postgres (Judge0's own — separate from the app DB)
   ↳ Redis (Judge0's queue)
Submission Capture
   ↳ source + test results → S3 → SHA-256 → BountySubmission row
```

### Submission flow
1. Candidate opens a Judge0-backed bounty → orchestrator creates a session row.
2. Candidate edits code in a Monaco editor in the browser. Auto-save every 30s.
3. Candidate clicks "Run" → orchestrator submits to Judge0 with the candidate's code, the language ID, stdin (sample test), and resource limits.
4. Judge0 returns stdout, stderr, time, and memory.
5. Candidate clicks "Submit" → orchestrator runs the code against ALL hidden test cases sequentially.
6. Orchestrator computes pass count, total time, total memory.
7. Source + test results uploaded to S3 with SHA-256.
8. BountySubmission row written; status `submitted`.
9. EvaluationService picks it up async.

### Resource limits per execution
- CPU time: per-bounty, default 5s.
- Wall time: 10s.
- Memory: 256MB default, configurable up to 1GB.
- Stack: 64MB.
- Output size: 10MB max.
- Number of processes: 50.

These are configured per-bounty by the recruiter at template creation.

### Hidden test cases
- Stored in S3, encrypted at rest, never sent to the client.
- Orchestrator fetches them server-side at submission time.
- Test case visibility flags: `sample` (visible to candidate during work), `hidden` (only used at finalize), `validation` (used only after submission, for grading).

### Anti-cheat at this layer
- Number of "Run" invocations logged.
- Time between paste events and runs.
- Code length deltas between auto-saves (sudden large jumps flagged).
- Source captured at every save → diffed → "code evolution" trail for review.

## Runtime 2: GitHub Codespaces / StackBlitz (project bounties)

### What it is
For bounties that require a real multi-file environment — say, "build a working REST API with tests" or "fix this React app" — Judge0 is too narrow. We use **GitHub Codespaces** (for backend, system, polyglot work) or **StackBlitz WebContainers** (for Node/frontend, runs entirely in the browser).

### Why both?
- **Codespaces:** real Linux VMs in the cloud, full devcontainer support, any language, any tool. Scales well, integrates with GitHub. Cost: per-minute Codespace billing.
- **StackBlitz:** runs in-browser via WebContainers, near-instant boot, zero per-user infra cost, but limited to Node-compatible runtimes.

The orchestrator picks based on the bounty template's declared stack.

### Codespaces flow
1. Bounty template defines a `devcontainer.json` and a starter repo.
2. Candidate joins → orchestrator forks the starter repo into a one-off candidate workspace.
3. Orchestrator launches a Codespace via the GitHub API on the candidate's workspace.
4. Candidate edits and runs code in the in-browser VS Code instance.
5. Auto-commit every 5 minutes via the orchestrator (git snapshots).
6. Submit → orchestrator stops the Codespace, snapshots final state to S3, runs the bounty's grading script in a clean Judge0-style runner.
7. BountySubmission written with the full git history attached.

### StackBlitz flow
1. Bounty template defines a starter project (package.json + source).
2. Candidate joins → orchestrator embeds the StackBlitz SDK in the page with the starter loaded.
3. All execution is client-side via WebContainers — no server cost.
4. Auto-save snapshots stream back to the orchestrator.
5. Submit → orchestrator pulls the final project state, runs grading.

### Real-time collaboration (post-MVP)
For bounty types that involve pair work or interview-style live sessions, **Yjs** (CRDT library) provides real-time collaborative editing on top of Monaco. Deferred to Phase 3.

## Runtime 3: Docker on Kubernetes (custom environments)

### What it is
For niche bounties where neither Judge0 nor Codespaces fits — e.g., "evaluate this trained ML model in a TensorFlow environment with specific GPU requirements" — the recruiter uploads a **Dockerfile and a starter image** at template creation. Candidates spin up isolated containers from this template.

### Architecture
- Recruiter uploads Dockerfile via the bounty template UI.
- Image is built by the platform (not the recruiter), scanned with Trivy, signed with Cosign.
- Image is pushed to a private ECR namespace under the bounty.
- At session start, orchestrator launches a pod from the signed image in the sandbox node group.
- Pod has the same isolation profile as Judge0 workers: default-deny egress, resource limits, no privileged capabilities, non-root user.
- Submit → orchestrator snapshots the workspace volume to S3 and tears down the pod.

### Constraints
- Image size cap: 5GB.
- No GPU at MVP. (GPU bounties are a Phase 4 consideration.)
- Recruiter cannot specify arbitrary entrypoints; the platform-provided entrypoint runs the candidate's session and the grading hook.
- All images rebuilt monthly with security patches.

## Common Layer: Anti-Cheat Telemetry
Captured per session regardless of runtime:
- Tab visibility events (browser focus/blur).
- Paste events with size and content hash.
- Keystroke timing entropy (sliding window) — see `09-trust-and-reputation-spec.md` for the biometrics rules.
- Source diffs at every auto-save (the "code evolution trail").
- For Judge0: number of runs, time between runs.
- For Codespaces: git commit cadence and content.
- LLM-call detection: known endpoint domains in network logs (Codespaces and Docker runtimes only).

Telemetry is stored as `anti_cheat_flags` JSON on the submission row. **It is signal, not verdict** — no auto-disqualification; it surfaces in the Performance Report and admin tools.

## Common Layer: Submission Integrity
- Submission artifacts stored as a tarball.
- SHA-256 computed server-side and stored on the submission row.
- S3 uses bucket versioning + object lock.

## Common Layer: Failure Modes
- **Judge0 worker crash:** orchestrator retries the test run on another worker.
- **Codespace failure:** orchestrator detects via heartbeat; offers candidate a resume from the last git commit.
- **Custom Docker pod crash:** orchestrator offers a resume from the last volume snapshot.
- **Total subsystem outage:** marketplace browsing unaffected; new joins paused with a banner explaining ETA.

## Scaling
- **Judge0 workers:** HPA on queue depth. Spot instances for burst.
- **Codespaces:** scaled by GitHub. Cost is per-minute, so the orchestrator enforces a per-bounty cap on Codespace minutes.
- **Custom Docker:** warm pool not feasible (each bounty has a different image); cold start tolerated (~30s) with a clear loading state in the UI.

## Observability
- Per-session trace.
- Per-runtime SLO: Judge0 99.5% successful executions; Codespaces 99% successful sessions; custom Docker 98%.
- Alert on: queue depth, escape attempts (target 0), Codespaces minute spend approaching cap.

## Security Controls
- All runtimes: non-root, default-deny egress, resource limits, no host devices, no privileged capabilities.
- Judge0: runs in its own node group with stricter NetworkPolicy.
- Codespaces: GitHub manages isolation; we trust their boundary but never embed secrets in the dev container.
- Custom Docker: signed images only, admission controller verifies signatures, periodic vulnerability scans.
- All admin actions on the fleet are audit-logged.

## Deferred / Future
- **Firecracker microVMs.** The original specification proposed Firecracker as the primary isolation layer. This is preserved as a Phase 4+ option if and only if (a) Judge0/Codespaces/Docker prove insufficient for a real bounty type or (b) a security review determines stronger isolation is required. Firecracker is a major operational investment and is NOT MVP scope.
- **GPU bounties** (Phase 4).
- **Real-time collaborative bounties via Yjs** (Phase 3).

## Open Questions
- Per-bounty Codespaces minute cap: what's the right ceiling?
- Should StackBlitz be the default for any frontend bounty even if Codespaces would also work? (StackBlitz is dramatically cheaper but more limited.)
- How do we handle a bounty that needs internet access for legitimate reasons (e.g., calling a public API)?

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
- 2026-04-10 — v0.2 — Full rewrite. Pivoted from custom Firecracker microVMs to Judge0 + GitHub Codespaces/StackBlitz + Docker-on-K8s. Firecracker preserved as Phase 4+ option.
