# Git Workflow Reference — Conventional Commits

> This file applies to ALL phases.
> Git discipline is a non-negotiable skill for internship readiness.

---

## Why This Matters

In any team (and in your internship), people will:
- Read your commit history to understand what you changed
- Review your PRs before merging
- `git blame` to find who wrote a specific line and why

If your commits say "update", "fix bug", "asdf" — nobody can understand the
history. **Conventional Commits** solve this.

---

## Conventional Commit Format

```
<type>(<scope>): <short description>

[optional body — explain WHY, not WHAT]

[optional footer — e.g., Closes #123]
```

### Types

| Type | When to use | Example |
|------|-------------|---------|
| `feat` | New feature or functionality | `feat(user): add registration endpoint` |
| `fix` | Bug fix | `fix(order): prevent negative total amount` |
| `refactor` | Code change that doesn't add features or fix bugs | `refactor(service): extract OrderMapper from OrderService` |
| `test` | Adding or fixing tests | `test(user): add unit tests for UserService` |
| `docs` | Documentation changes | `docs: update README with setup instructions` |
| `chore` | Build, config, dependency changes | `chore: add springdoc-openapi dependency` |
| `style` | Formatting, whitespace (no logic change) | `style: fix indentation in UserController` |
| `perf` | Performance improvement | `perf(query): add index for findByEmail` |

### Scope (optional but recommended)

The scope is the module or feature area:
- `feat(user)` — User module
- `fix(auth)` — Authentication
- `refactor(order)` — Order module
- `test(payment)` — Payment tests

### Rules

1. **Type is always lowercase**: `feat`, not `Feat` or `FEAT`
2. **Description starts lowercase**: `feat: add ...`, not `feat: Add ...`
3. **No period at the end**: `feat: add user endpoint`, not `feat: add user endpoint.`
4. **Use imperative mood**: "add", "fix", "remove" — not "added", "fixed", "removed"
5. **Keep under 72 characters** for the first line

---

## Commit Granularity

### 🔴 BAD: One giant commit
```
feat: implement user management
- Added User entity
- Added UserRepository
- Added UserService
- Added UserController
- Added DTOs
- Added tests
- Added Swagger documentation
```

### ✅ GOOD: Small, focused commits
```
feat(user): add User entity with JPA annotations
feat(user): add UserRepository with email lookup
feat(user): add UserService with create and findById
feat(user): add UserController with REST endpoints
feat(user): add CreateUserRequest and UserResponse DTOs
test(user): add unit tests for UserService
docs(user): add Swagger annotations to UserController
```

**Rule of thumb**: One commit = one logical change. If you can't describe it
in one short sentence, it's probably too big.

---

## Branch Strategy (simple)

For a solo project or internship:

```
main          ← always working, deployable
  └── feat/xxx  ← feature branch, merged via PR
  └── fix/xxx   ← bugfix branch
  └── refactor/xxx ← refactoring branch
```

### Workflow
```bash
# Start a new feature
git checkout -b feat/add-order-endpoint

# Work, commit small and often
git add .
git commit -m "feat(order): add Order entity with status enum"
git commit -m "feat(order): add OrderService with create logic"

# Push and create PR
git push origin feat/add-order-endpoint
# → Create Pull Request on GitHub
# → Merge after review
```

---

## .gitignore for Spring Boot

```gitignore
# Build output
target/

# IDE
.idea/
*.iml
.vscode/
*.swp

# Environment
.env
*.env

# OS
.DS_Store
Thumbs.db

# Logs
*.log

# Mentor skill (optional — keep or ignore based on preference)
# .mentor/SESSION_LOG.md
# .mentor/AUDIT_REPORT.md
# .mentor/ROADMAP.md
```

---

## Git Commands Every Intern Must Know

| Command | Purpose |
|---------|---------|
| `git status` | See what's changed |
| `git add -p` | Stage changes interactively (review each hunk) |
| `git commit -m "..."` | Commit with message |
| `git log --oneline -10` | See last 10 commits |
| `git diff` | See unstaged changes |
| `git branch` | List branches |
| `git checkout -b name` | Create and switch to new branch |
| `git push origin branch` | Push branch to remote |
| `git pull origin main` | Update from main |
| `git stash` | Temporarily save uncommitted changes |
| `git stash pop` | Restore stashed changes |

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| `git add .` without reviewing | Use `git add -p` to review each change |
| Commit message "fix" or "update" | Use conventional format: `fix(scope): description` |
| Committing `.env` or `target/` | Add to `.gitignore` before first commit |
| One huge commit for everything | Break into small, logical commits |
| Working directly on `main` | Always use feature branches |
| Never pulling before pushing | `git pull origin main` before starting work |
| Committing commented-out code | Remove dead code, don't comment it |

---

## Checklist

- [ ] `.gitignore` configured (no `target/`, `.env`, `.idea/`)
- [ ] All commits follow Conventional Commit format
- [ ] No commit contains unrelated changes (keep commits focused)
- [ ] Student uses feature branches, not direct commits to `main`
- [ ] Student can explain: `git log --oneline` output makes sense to a stranger
- [ ] Student knows: `git add -p`, `git stash`, `git diff`
