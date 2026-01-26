---
name: gsd:discuss-design
description: Design phase-specific UI through conversation before planning
argument-hint: "<phase>"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - WebFetch
  - AskUserQuestion
  - Task
---

<objective>

Design phase-specific UI elements through conversation, then generate visual mockups for review. Ensures design decisions are made before implementation time is spent.

**What it does:**
1. Load design system (if exists) as visual foundation
2. Conversationally design components needed for the phase
3. Generate framework-appropriate mockups (React, SwiftUI, HTML, Python)
4. Iterate until user approves the visuals
5. Create DESIGN.md that the planner automatically loads

**Output:**
- `.planning/phases/{phase}/{phase}-DESIGN.md` — component specs for planner
- `.planning/phases/{phase}/mockups/` — visual previews

</objective>

<execution_context>
@~/.claude/gsd-extensions/workflows/discuss-design.md
@~/.claude/gsd-extensions/references/ui-principles.md
@~/.claude/gsd-extensions/references/framework-patterns.md
@~/.claude/gsd-extensions/templates/phase-design-template.md
</execution_context>

<context>
Phase number: $ARGUMENTS (required)

**Load design system (if exists):**
@.planning/DESIGN-SYSTEM.md

**Load roadmap:**
@.planning/ROADMAP.md
</context>

<process>

Follow the workflow completely. Key phases:

1. **Parse phase** — Validate phase number, load phase details
2. **Load design system** — Use as foundation if exists
3. **Detect framework** — React, SwiftUI, Python, HTML for mockup format
4. **Visual references** — Optionally analyze phase-specific references
5. **Component discovery** — Identify and spec all UI components
6. **Layout discussion** — How components are arranged
7. **Interaction discussion** — States, transitions, feedback
8. **Generate mockups** — Spawn design specialist agent
9. **Review loop** — Iterate until approved
10. **Create DESIGN.md** — Specs for planner

</process>

<success_criteria>
- [ ] Phase requirements understood
- [ ] Design system loaded as context (if exists)
- [ ] All UI components identified and specified
- [ ] Component states documented (loading, error, empty, etc.)
- [ ] Layout and responsive behavior defined
- [ ] Framework-appropriate mockups generated
- [ ] User approved mockups visually
- [ ] DESIGN.md created for planner consumption
- [ ] User knows next steps (plan or edit)
</success_criteria>
