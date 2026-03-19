# 🌱 Spring Mentor

AI-powered mentor skill for IT students learning **Java Spring Boot**, targeting their first backend internship.

## What is this?

A `.mentor` folder that you drop into any Spring Boot project. When an AI agent (Antigravity, Claude Code, Cursor, etc.) opens the project, it becomes a **senior engineer mentor** that:

1. **Scans** your entire project — detects tech stack, folder structure, dependencies
2. **Audits** your code against SOLID principles — finds real violations with file/line references
3. **Checks intern readiness** — missing JWT auth? No pagination? No tests? It adds real tasks
4. **Generates a personalized roadmap** — 5 phases, 10 weeks, every task references your actual code
5. **Teaches you** — never writes code for you, guides you step by step

## Quick Start

```bash
# 1. Clone this repo
git clone https://github.com/ariushieu/spring-mentor.git

# 2. Copy .mentor into your Spring Boot project
cp -r spring-mentor/.mentor /path/to/your-spring-boot-project/

# 3. Open project in your AI-powered editor

# 4. Say "Start session" — the mentor takes over
```

## What You'll Learn (10-week roadmap)

| Phase | Weeks | Topics |
|-------|-------|--------|
| 1 | 1–2 | SOLID principles — find & fix real violations in your code |
| 2 | 3–4 | Refactoring + Unit testing (JUnit 5, Mockito) |
| 3 | 5–6 | Clean Architecture + API docs (Swagger) + Integration tests |
| 4 | 7–8 | Docker (multi-stage build, compose, health checks) |
| 5 | 9–10 | CI/CD pipeline + VPS deployment |

## Features

- 🔍 **Deep project scanning** — two-pass strategy, works on projects of any size
- 📊 **SOLID audit** with severity ratings (HIGH / MEDIUM / LOW)
- 🎯 **Intern readiness gap analysis** — detects 9 missing critical features (JWT, N+1, Pagination, Validation, etc.)
- 📝 **Conventional Commits** enforcement — Git discipline from day 1
- ☕ **Modern Java guidance** — Records, Switch Expressions, Optional (Java 17+/21+)
- 🧪 **Testing strategy** — unit → repository → controller → integration
- 📄 **API documentation** — Swagger/OpenAPI setup and annotation
- 🐳 **Docker** — multi-stage builds, JVM container flags, Spring profiles
- 🚀 **CI/CD** — GitHub Actions with auto-rollback on failed health check
- 📈 **Progress tracking** — 1–5 understanding scores for 20 concepts
- 💾 **Session memory** — incremental state saves, session compaction, resume from exact position

## File Structure

```
.mentor/
├── SKILL.md                         ← Core agent instructions
├── ROADMAP.md                       ← Generated after first scan
├── AUDIT_REPORT.md                  ← Generated SOLID violations report
├── SESSION_LOG.md                   ← Session memory + scores
├── README.md                        ← Internal docs
└── references/
    ├── git-workflow.md              ← Conventional Commits guide
    ├── solid-audit-protocol.md      ← SOLID scanning methodology
    ├── testing-strategy.md          ← JUnit 5 + Mockito + Spring Test
    ├── roadmap-template.md          ← Roadmap structure guide
    ├── clean-architecture.md        ← Spring Boot layered architecture
    ├── api-documentation.md         ← Swagger/OpenAPI guide
    ├── docker.md                    ← Docker for Spring Boot
    ├── cicd-vps.md                  ← VPS + CI/CD pipeline
    └── production-checklist.md      ← Production readiness checklist
```

## How It Works

```
You: "Start session"
  ↓
Agent reads SKILL.md → checks SESSION_LOG.md
  ↓
First time? → Deep scan → SOLID audit → Gap analysis → Generate roadmap
Returning?  → Read state → Resume from last task
  ↓
Teaches via: explain concept → show bad code → ask student → review → fix together
  ↓
After each task: commit (conventional), update SESSION_LOG, move to next task
```

## Requirements

- A Java Spring Boot project (Maven or Gradle)
- An AI coding agent (Antigravity, Claude Code, Cursor, etc.)
- Java 17+ recommended (for modern features guidance)

## Tips

- **Add `.mentor/` to your project's `.gitignore`** — it's a personal learning tool, not source code
- The mentor communicates in **Vietnamese** but writes all files in **English**
- If the agent drifts, say: *"Follow the roadmap, resume from SESSION_LOG"*
- You can modify the roadmap — the agent will log and respect changes

## License

MIT
