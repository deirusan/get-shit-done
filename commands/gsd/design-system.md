---
name: gsd:design-system
description: Establish project-wide design foundation through conversation
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - WebFetch
  - AskUserQuestion
---

<objective>

Establish the visual foundation for the entire project through conversational discovery. Creates `.planning/DESIGN-SYSTEM.md` that all UI work in the project respects.

**What it does:**
1. Optionally analyze visual references (images, URLs) for aesthetic patterns
2. Conversationally discover design preferences (colors, typography, spacing, components)
3. Detect project framework for appropriate patterns
4. Generate a comprehensive design system document

**Output:** `.planning/DESIGN-SYSTEM.md` — the visual foundation that `/gsd:discuss-design` and planning phases load as context

</objective>

<execution_context>
@~/.claude/gsd-extensions/workflows/design-system.md
@~/.claude/gsd-extensions/references/ui-principles.md
@~/.claude/gsd-extensions/templates/design-system-template.md
</execution_context>

<context>

**When to use:**
- Starting a new project with UI
- Before first UI phase
- When suggested after `/gsd:new-project`
- When visual direction needs definition

</context>

<process>

Follow the workflow completely. Key phases:

1. **Check existing** — If DESIGN-SYSTEM.md exists, offer update/replace/cancel
2. **Detect framework** — React, SwiftUI, Python, HTML/CSS
3. **Visual references** — Optionally analyze user-provided images/URLs
4. **Aesthetic direction** — Conversationally discover style preferences
5. **Color exploration** — Build or capture color palette
6. **Typography** — Font selection and scale
7. **Component style** — Border radius, shadows, borders
8. **Spacing system** — Base unit and scale
9. **Generate** — Create DESIGN-SYSTEM.md

</process>

<success_criteria>
- [ ] User's aesthetic vision understood
- [ ] Visual references analyzed (if provided)
- [ ] Color palette defined with tokens
- [ ] Typography scale established
- [ ] Spacing system determined
- [ ] Component patterns documented
- [ ] Framework-appropriate values included
- [ ] DESIGN-SYSTEM.md created
- [ ] User knows how system integrates with GSD
</success_criteria>
