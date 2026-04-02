Make sure you're following instructions in ~/.codex/AGENTS.md as well as workspace AGENTS.md.

---

# Vocabulary

- **Workspace project** - a Cargo package, a Gradle project, in other words an individually packagable, publishable and reusable artifact that is local to the workspace.
- **Subproject** - a non-root workspace project.
- **Aggregating directory** - a directory that is itself not a workspace project, but contains workspace projects.
- **Module** - a `mod` module in its own file in Rust, a source file in Java

# Documentation Contract

## Design

Every workspace project must include a design spec at `doc/design.md`.

Aggregating directories may omit `doc/`, but they may opt in with a shared design doc when decisions must be documented across multiple child workspace projects.

A workspace project's `doc/design.md` is owned by that workspace project.

- It may define decisions about the workspace project itself.
- It may define the workspace project's own public boundary contract: what it guarantees, what it requires, and what inputs/outputs are valid.
- It may mention workspace projects it depends on only as assumptions or constraints of the APIs it consumes.
- It must not define internal policy for workspace projects it depends on.
- It must not define architecture or behavior policy for workspace projects that use it.
- It must not justify decisions based on unrelated systems outside this workspace project.
- The `## Non-goals` section is exempt from these scope limits.

Cross-crate contract placement rule:
- Keep a contract in a workspace project's `doc/design.md` only when the contract is owned by that workspace project's boundary.
- If a contract sets shared rules between peer workspace projects, sibling workspace projects, parent-level orchestration, or anything not owned by one workspace project's boundary, put it in the nearest common parent `doc/design.md`.

Aggregating directory's `doc/design.md` files are only for those shared cross-workspace-project contracts. Do not include decisions that are internal to a single crate there

Child crate `doc/design.md` files must be self-contained crate-boundary contracts.

Do not include AGENTS/process-level guidance in crate design docs.

If a rule is shared across sibling crates, place it only in the nearest common parent `doc/design.md` and avoid duplicating it in child docs unless needed to define child-owned behavior.

## `doc/design.md` Structure

First mandatory section: `# Goals`.

State, briefly, the high-level problem the workspace project exists to solve (what, not how). You may include `## Non-goals` to narrow scope. All later decisions must derive from these goals.

Second mandatory section: `# Decisions`.

List design decisions derived from `# Goals` without contradiction. Use `##` subsections when needed for organization.

List only decisions about the workspace project's target state. Exclude any decisions tied to migration steps, transitional/ongoing work, or past/current-state references.

## Parent Design Consultation (Process Rule Only)

When working on a workspace project, the assistant must consult parent `doc/design.md` files.

This is a workflow requirement for the assistant, not documentation content.

Do not add process reminders in crate docs such as:
- "consult parent design.md"
- "inherits parent contract"
- "implements parent contract"

unless the operator explicitly asks for that wording.

# Planning Contract

`doc/plan.md` is optional by default.

Plan file state semantics:

- Missing `doc/plan.md`: no implementation work has ever been planned for this workspace project.
- Present and non-empty `doc/plan.md`: there is active/pending planned work.
- Present and empty `doc/plan.md`: this workspace project had planned work in the past, and all phases are complete.

Before implementation starts for a workspace project, planned work must be captured in `doc/plan.md`.
If the file is missing, create it first from `doc/design.md`s, then implement.

Required `doc/plan.md` structure (applies when the file is non-empty):

- `# Scope`
- one or more phase sections in the format `# Phase N: <description> (pending|wip|finished)`

Example: `# Phase 1: Make foo do bar (pending)`.

During implementation, if a phase could not be completed due to any reasons that make the assistant stop, add a description of the issue to that phase, so that later sessions would know what needs to be solved for that phase in order to continue.

And tighten hygiene so the semantics stay unambiguous:

## Plan Hygiene

- Do not create placeholder `doc/plan.md` files for crates with no planned work.
- Keep in `doc/plan.md` only active and pending phases plus a minimal completed summary for immediate context.
- When all phases are fully finished, empty `doc/plan.md` and keep the file present.
- Do not delete `doc/plan.md` once created; missing should continue to mean “never planned.”
- When a plan is marked complete, verify linked plans and workspace-root `doc/plan.md` references still resolve after cleanup.

## Cross-Crate Planning Contract

If planned work spans multiple workspace projects, track that work in the root `doc/plan.md`.

The root `doc/plan.md` must track readiness across all involved workspace projects so the operator can resume from the latest recorded milestone without relying on a separate cross-project plan file.

Keep only minimal cross-project tracking information in the root `doc/plan.md`. Avoid copying detailed content from workspace project's `doc/plan.md` files.

# Diary

Write `doc/diary.md` when an error is discovered in either design.md or plan.md decisions, for example:

- a design decision turned out to be wrong for some reason
- a technical solution fails and an alternative must be tried

Diary format requirements:

- one date section per day as `# Month Day, Year`
- a brief narrative of what happened

The priority for diary entries is to document what did *not* work so future sessions do not repeat failed approaches.

`doc/diary.md` may be absent if no implementation failures occurred yet.

# Authority And Conflict Resolution

All decisions and code implementation must stem from `doc/design.md`.

Implementation must strictly follow `doc/plan.md`.

If an interactive operator command contradicts anything in `doc/`, stop and resolve the contradiction before continuing.

# Filesystem Policies

Honor `.gitignore` files and rules when doing anything with the project's filesystem.

Source files that exceed roughly 500 lines should be split into focused internal modules when reasonably possible, while preserving public API shape and behavior.

# Environmental Facts

Environment-specific facts that are only true for a particular local agent environment go into `ENV.md`. `ENV.md` must not be committed to VCS. If it's discovered to be part of a VCS, stop and treat it as a fatal error.

# Implementation Policies

Stop after a phase of the root `doc/plan.md` is done and ask the operator to review the code and/or live-test the code when appropriate.

# Sub-agent Coordination

Do use subagents when you need to explore code to keep the main context clear of data that's not needed in it.

When using multi-agent workflows:

- Prefer sub-agents for read-heavy tasks such as exploration, test execution, triage, review, and summarization.
- Do not assign multiple sub-agents to edit the same files or the same crate in parallel.
- The main agent owns shared/root artifacts and final integration, including `doc/plan.md`, workspace `AGENTS.md`, and any parent `doc/design.md` that defines cross-project contracts.
- A sub-agent should stay within its assigned workspace project unless explicitly told otherwise.
- If a sub-agent discovers a needed shared-contract change, it should report it in its handoff; the main agent should apply the shared change.
- Sub-agents should return concise summaries of findings, files touched, commands run, and proposed follow-up changes.

# For Cargo projects only

Workspace members must use `workspace = true` in their dependencies section - I want dependency versions
and paths centralized in the root Cargo.toml and for the workspace members to defer to that.

Each crate's API surface must be discoverable at a glance via documentation in `lib.rs` that shows compiling examples.

All unit tests must be placed in the crate-root `tests/` directory, not in main source files.

Use `cargo-nextest` for tests, never `cargo test`.
