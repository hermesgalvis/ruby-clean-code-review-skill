---
name: ruby-clean-code-review
description: >
  Analyzes Ruby/Rails code changes in the current branch against Clean Code
  principles adapted to Ruby (https://github.com/uohzxela/clean-code-ruby).
  Generates a bilingual (Spanish explanations + English technical terms)
  teaching-oriented Markdown report with specific fix suggestions.
  Invoke when: (1) a developer runs /ruby-clean-code-review before opening a
  PR, (2) someone asks to review Ruby code against clean code principles,
  (3) code review prep on a feature or bugfix branch.
  Only works in Ruby/Rails projects (requires Gemfile). Global skill —
  available in any project.
allowed-tools: Bash, Read, Write
---

# Ruby Clean Code Review

## Important constraint

**Read-only by default.** This skill analyzes and reports — it does NOT modify
any source file. Only apply fixes to the project's code if the user explicitly
asks for it after reading the report (e.g., "fix finding V3" or "apply all
suggested fixes"). When in doubt, ask before touching any file.

## Step 1 — Verify environment

Run both checks:
```bash
[ -f Gemfile ] && echo "HAS_GEMFILE" || echo "NO_GEMFILE"
git rev-parse --git-dir 2>/dev/null && echo "IS_GIT" || echo "NOT_GIT"
```

- `NO_GEMFILE` → stop and tell the user:
  > **Esta skill es exclusiva para proyectos Ruby / Ruby on Rails.**
  > No encontré un `Gemfile` en el directorio actual. Ejecuta
  > `/ruby-clean-code-review` desde la raíz de tu proyecto Ruby.

- `NOT_GIT` → stop and tell the user:
  > **Este directorio no es un repositorio git.**
  > Inicializa uno con `git init` o navega a la raíz del proyecto.

## Step 2 — Determine what to analyze

```bash
git branch --show-current
git status --short
```

**If `git branch --show-current` returns empty string** (detached HEAD):
> Estás en estado "detached HEAD". Crea o cambia a una rama con
> `git checkout -b <nombre-rama>` y ejecuta la skill de nuevo.
Stop here.

**If branch is `master` or `main`:**
- No uncommitted changes → stop:
  > **No hay nada que analizar en `master`/`main`.**
  > Esta skill revisa cambios que aún no están en la rama principal.
  > Opciones: crea una rama (`git checkout -b mi-feature`) o realiza
  > cambios locales sin commitear y ejecuta la skill de nuevo.
- Has uncommitted changes → get changes with both commands and combine:
  ```bash
  git diff -- '*.rb'
  git ls-files --others --exclude-standard -- '*.rb'
  ```
  `git diff` covers modified tracked files. `git ls-files --others` lists
  new untracked files — read each one with the Read tool to get its content.
  Label all findings as **"cambios sin commitear en master"**. Skip Steps 3a–3b.

**Any other branch:**

3a. Detect base branch:
```bash
git remote show origin 2>/dev/null | grep 'HEAD branch' | awk '{print $NF}'
```
If the command fails or returns empty, use `master` as default and note it.

3b. Check if branch is behind:
```bash
git fetch origin --quiet 2>&1; git status -sb
```
If the status shows `behind`:
- **Ask the user** before merging:
  > Tu rama está desactualizada frente a `<base_branch>`.
  > ¿Quieres que actualice la rama ahora con `git merge origin/<base_branch>`?
  > (S/n)
- If yes → run `git merge origin/<base_branch> --no-edit 2>&1`.
  - If output contains `CONFLICT` → warn the user to resolve conflicts
    manually and stop.
  - Otherwise → confirm update to user and continue.
- If no → continue with the diff as-is and note in the report that the
  branch may be outdated.

3c. Get committed changes vs base:
```bash
git diff origin/<base_branch>...HEAD -- '*.rb'
```
3d. Get uncommitted changes (if any):
```bash
git diff -- '*.rb'
```
Combine both. If the combined diff is empty:
> No hay cambios en archivos `.rb` para analizar en esta rama.
Stop here.

**Large diff warning:** if the diff exceeds ~400 lines, inform the user that
the analysis will cover all files but output may be long. Do not truncate the
analysis.

## Step 3 — Load project conventions

Check and read (in order) if they exist:
1. `.claude/CLAUDE.md` or `CLAUDE.md` — extract naming rules, domain vocab,
   banned abbreviations.
2. `.agents/rules/ruby-rails.md` — Ruby/Rails code conventions.
3. `.agents/rules/testing.md` — testing patterns (read only if test files
   appear in the diff).

Note the conventions found; use them to enrich N1 findings. If no files
exist, apply standard Rails community conventions.

## Step 4 — Analyze

Read `references/principles.md` now.

Analyze the combined diff against all principles in that file. For each
violation found, record:
- Principle code (e.g., V3, M5, C1)
- File path and line number
- The exact lines from the diff
- A suggested fix respecting the project's naming conventions

**Only flag changed lines.** Do not report on code that wasn't modified.

## Step 5 — Generate report

Read `references/report-template.md` now.

Build the report following the structure and tone guidelines in that file.
Save it and tell the user the exact path.
