# Work Units: Issue / Task / TODO / Ticket

## Purpose

`bonsai/solve` defines four levels of work so that a goal can be decomposed into work that an Agent can actually execute.

```text
Goal
  ↓
Issue
  ↓
Task
  ↓
TODO
  ↓
Ticket
  ↓
Agent
  ↓
Action
  ↓
Event
  ↓
Status
  ↓
Verify
  ↓
Solve
```

## Definitions

| Concept | Definition | Primary question | Completion condition |
|---|---|---|---|
| **Issue** | A problem, objective, or state that needs to be solved | Why / What needs solving? | The problem is resolved and verified |
| **Task** | A coherent unit of work required to solve an Issue | What work must be done? | The intended work product exists |
| **TODO** | A concrete next piece of work for a human or Agent | What do I do next? | The activity is completed |
| **Ticket** | An executable work package prepared for an Agent | What exactly should the Agent execute? | Execution produces an observable result/Event |

## Issue

An **Issue** is the unit of a problem to be solved. It has a goal, context, current status, and acceptance/verification criteria.

An Issue can contain multiple Tasks.

```text
Issue
├── Task A
├── Task B
└── Task C
```

An Issue is not an implementation instruction. It represents the reason and desired outcome for the work.

## Task

A **Task** is a coherent group of work contributing to an Issue.

A Task should be small enough to plan and track, but large enough to represent a meaningful unit of work.

```text
Issue: migrate issue-related knowledge

Task: inventory existing issue documentation
Task: classify documents
Task: migrate selected documents
Task: verify references
```

## TODO

A **TODO** is a concrete action that can be taken next by a person or Agent.

TODOs are operational rather than conceptual.

```text
TODO:
- inspect wiki/issue.md
- compare definitions with bonsai/solve
- identify duplicate status definitions
```

A TODO does not necessarily contain enough structured information to be executed autonomously.

## Ticket

A **Ticket** is a TODO transformed into an explicit, executable work package for an Agent.

A Ticket should specify at least:

```yaml
id: TICKET-001
target: bonsai/solve
action: inspect
input:
  path: docs/work-units.md
expected_output:
  type: report
acceptance:
  - file was inspected
  - findings are recorded
actor: agent
```

The Ticket is the boundary between planning and Agent execution.

## TODO vs Ticket

The distinction is intentional:

```text
TODO
「READMEを確認する」

        ↓ formalize

Ticket
「bonsai/solve の README.md を読み、
定義されている Status と Work Unit の差分を報告する」
```

Therefore:

```text
TODO   = やること
Ticket = Agentが実行できる形にした仕事
```

A TODO may remain human-oriented. A Ticket must be sufficiently explicit for an Agent to execute and report an observable result.

## Relationships

```text
Goal
  │
  └── Issue
        │
        ├── Task
        │     ├── TODO
        │     └── TODO
        │
        └── Task
              └── TODO
                    ↓ formalize
                  Ticket
                    ↓
                  Agent
                    ↓
                  Action
                    ↓
                  Event
                    ↓
                  Status
```

Issue, Task, TODO, and Ticket are therefore different layers, not synonyms.

## Boundary with Action / Event / Status

`bonsai/solve` already models `Action`, `Event`, `Status`, and `Skill`. Work units describe **what needs to be done**; execution concepts describe **what was done and what happened**.

```text
Work planning                         Execution
─────────────────                    ─────────────────
Issue                                 Agent
  ↓                                    ↓
Task                                 Action
  ↓                                    ↓
TODO                                 Event
  ↓                                    ↓
Ticket                               Status

             Ticket → Action → Event → Status
```

The execution result must be observable. An Agent must not mark an Issue as solved merely because it attempted a Ticket.

## Core rule

> Issue is the problem. Task is the work. TODO is the next concrete activity. Ticket is the executable package.

The solve loop then connects the work model to the existing status machine:

```text
open
  → understood
  → planned
  → executing
  → implemented
  → verified
  → solved
```

A failed or blocked Ticket produces an Event and leaves the Issue at the appropriate observed Status; it must not be forced into `solved`.
