# T49 - Design tokens and global styles

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T49
- **Type:** deployment
- **Owner:** Irimies Vasile
- **Requirement(s):** 
- **Use case(s):** 
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T49-design-tokens-and-global-styles`
- **PR link(s):** 
- **Tests:** (test classes/files, filled at the end)

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 2 — Design tokens and global styles

**Description:** Implement the design tokens so no component hardcodes colors, sizes or fonts.
Figma: page `Design System`, frame `Foundations` (4:40) and the variable collections `Primitives`, `Semantic`, `Spacing & Radius` (read them through `get_variable_defs`).
Include:
- the blue palette (#223A5E, #355982, #4D79A8, #6E9ECC, #9FC0E3, #D0E1F2, #F3F8FD)
- semantic tokens (bg/page, bg/surface, bg/brand, text/primary|secondary|muted|inverse, border/default|strong|focus, accent, status success/warning/danger fg+bg)
- spacing (4–48)
- radii (8, 12, 16, 24, pill)
- shadows `Shadow/Card` and `Shadow/Raised`
- font Plus Jakarta Sans (400–800) with the 11 text styles (Display/Hero, Heading H1–H3, Body Large/Default/Medium/Strong, Caption Default/Strong, Label/Button).

Add a global reset and set the page background to `bg/page`.
Done when a demo page side by side with the Foundations frame is visually identical.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [ ] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [ ] The task's stated dependency prerequisites are merged before implementation.
- [ ] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** 
- **Related completed tasks:** 
- **Constraints / assumptions:** 

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

### Steps

1. 

### Files to create / modify

### Tests to write

### Risks / open questions

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** 
- **Result:** 
- **Failure reason:** 

### Green (after implementation)

- **Result:** 
- **Full suite of the affected repo:** 

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
|   |                               |                |                              |     |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** 
- **Alternatives considered:** 
- **Limitations:** 

## Status

- [ ] Task read and context gathered
- [ ] Plan written
- [ ] **Gate 1:** plan approved by user
- [ ] Tests written and confirmed failing (red)
- [ ] Implementation done, tests passing (green)
- [ ] Implementation reviewed by user
- [ ] **Gate 2:** approved for commit and push
- [ ] Committed and pushed
- [ ] PR created
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [ ] Moved to `changes/completed/`


