# T52 - Base UI components: layout and data display

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T52
- **Type:** deployment
- **Owner:** Irimies Vasile
- **Requirement(s):** 
- **Use case(s):** 
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T52-base-ui-components-layout-and`
- **PR link(s):** 
- **Tests:** (test classes/files, filled at the end)

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 5 — Base UI components: layout and data display

**Description:** Implement the shared building blocks used by many screens:
- `AuthLayout` (split layout used by Login and Create account: left brand panel with gradient, headline, three feature rows and decorative circles; right form panel; frames `Login` 13:10 and `Create account` 13:62). Both auth screens use it, so they stay independent from each other
- `Tabs` in two styles (pill/segmented as in frame `My registrations` 14:355, and underline as in the event workspace frame 15:412)
- `Table` (header row, rows, bottom dividers, fixed and fill columns, as in `Org – Events` 15:10)
- `Modal` (centered card, `Shadow/Raised`, dark backdrop as in `Org – Delete event confirmation` 16:1061, focus trap, Esc to close)
- `InfoBanner` (info and warning tones)
- `Avatar` (initials from first and last name, sizes 36/48/80)
- `ProgressBar` (capacity)
- `DateBlock` (month + day tile)
- `InfoTile` (icon box + label + value, as in `Event details` 13:417)
- `PageHeader` (title, subtitle, right slot)
- `Icon` wrapper (calendar, pin, clock, users, building, bell, check, x, plus, send, lock, trash, edit, info, alert, chevrons, arrow-left, search, ticket, megaphone, tasks, shield, logout).

**Depends on:** 2, 4.

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


