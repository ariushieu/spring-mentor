# .mentor

AI mentor skill for **IT students** learning Java Spring Boot, targeting their
first backend internship.

Place this folder at the root of your Spring Boot project. When an AI agent
(Antigravity, Claude Code, or any compatible agent) starts a session in this
project, it reads these files and acts as a senior engineer mentor.

---

## How It Works

**First session**
The agent scans the entire codebase — `pom.xml`, Spring Boot config, controllers,
services, repositories, entities, tests. It detects SOLID violations, generates a
personalized `ROADMAP.md`, and writes a detailed `AUDIT_REPORT.md`. Nothing is
pre-written — everything comes from your actual code.

**Every session after that**
The agent reads `SESSION_LOG.md`, knows exactly where you left off, reports
progress and understanding scores, and continues from the next task.

---

## File Structure

```
.mentor/
├── SKILL.md                         ← Agent instructions (core behavior)
├── ROADMAP.md                       ← Generated after first scan
├── AUDIT_REPORT.md                  ← Generated SOLID violations report
├── SESSION_LOG.md                   ← Session memory + understanding scores
├── README.md                        ← This file
└── references/
    ├── git-workflow.md              ← Git + Conventional Commits (all phases)
    ├── solid-audit-protocol.md      ← SOLID scanning methodology (Phase 1–2)
    ├── testing-strategy.md          ← Testing guide — JUnit 5 + Mockito (Phase 2–3)
    ├── roadmap-template.md          ← Structure guide for roadmap generation
    ├── clean-architecture.md        ← Architecture guide (Phase 3)
    ├── api-documentation.md         ← Swagger/OpenAPI guide (Phase 3)
    ├── docker.md                    ← Docker guide (Phase 4)
    ├── cicd-vps.md                  ← VPS + CI/CD guide (Phase 5)
    └── production-checklist.md      ← Production readiness checklist
```

`ROADMAP.md` and `AUDIT_REPORT.md` are intentionally blank until the first session.

---

## Starting a Session

Say to the agent:

> "Start today's session."

Or if it is the very first time:

> "Begin mentoring. Scan the project."

---

## 10-Week Goal

| Phase | Weeks | Topic |
|-------|-------|-------|
| 1 | 1–2 | Project scan + SOLID audit |
| 2 | 3–4 | Refactor to SOLID + unit testing |
| 3 | 5–6 | Clean Architecture + integration testing |
| 4 | 7–8 | Docker |
| 5 | 9–10 | VPS + CI/CD via GitHub Actions |

**End state**: Push code → auto-deploy to VPS. HTTPS live. Tested. Explainable architecture.
