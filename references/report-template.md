# Report Template & Writing Guidelines

## File naming

Save to `<project-root>/tmp/ruby-clean-code-review-<branch>-<YYYYMMDD>.md`.

Tell the user the exact path where the file was saved.

---

## Report structure

```markdown
# Ruby Clean Code Review
**Branch:** <branch_name>
**Base:** <base_branch>
**Date:** <YYYY-MM-DD>
**Files analyzed:** <N> Ruby files

---

## Summary / Resumen

| Category / Categoría | Violations / Hallazgos |
|----------------------|------------------------|
| Variables            | X |
| Methods              | X |
| Classes / SOLID      | X |
| Naming               | X |
| Testing              | X |
| Error Handling       | X |
| **Total**            | **X** |

---

## Findings / Hallazgos

### 🔴 V3 — Magic Numbers / Números mágicos

**File:** `app/services/payroll_calculator.rb:42`

**¿Qué está pasando? / What's happening**
[2-4 sentences in Spanish explaining the problem as if to a junior dev.
Use concrete terms. Avoid being condescending.]

**¿Por qué importa? / Why it matters**
[1-2 sentences on the real-world consequence: harder maintenance,
bugs, confusion for new team members.]

**Código actual / Current code:**
```ruby
# exact lines from the diff
```

**Mejora sugerida / Suggested fix:**
```ruby
# improved version respecting the project's naming conventions
```

> 💡 **Principio / Principle:** V3 — Searchable Names
> El número `525_960` solo existe en la cabeza del autor. Una constante
> con nombre documenta la intención y facilita el cambio en un solo lugar.

---

(one section per finding, ordered by severity: 🔴 → 🟡 → 🟢)

---

## Lo que hiciste bien / What you did well 🌟

- [specific thing done well, not generic praise]
- [another specific thing]

---

## Próximos pasos / Next steps

1. [Most impactful fix]
2. [Second fix]
3. [Optional / low priority]
```

---

## Severity levels

- 🔴 **High** — Breaks a SOLID principle, introduces a real maintenance risk, or
  hides a logic error
- 🟡 **Medium** — Naming or style issue that adds cognitive load but doesn't
  break logic
- 🟢 **Low** — Minor polish; good to fix but won't block a PR

---

## Tone guidelines

- Explain as if teaching a junior dev: clear, encouraging, never condescending.
- Every finding header shows BOTH the technical principle name (English) AND
  a plain-language label (Spanish): `V3 — Magic Numbers / Números mágicos`.
- Always show the actual diff lines, then the improved version.
- The improved version MUST respect the project's naming conventions loaded in
  Step 3 (domain vocabulary, no banned abbreviations, etc.).
- "Lo que hiciste bien" must name specific patterns observed, not generic
  praise like "good code".
- If zero violations found:

  > ¡Excelente trabajo! No encontré violaciones a los principios de Clean Code
  > en estos cambios. Tu código es claro, mantenible y sigue las convenciones
  > del proyecto. 🎉
  >
  > **Lo que hiciste bien:**
  > - [specific good practices observed]
