# Problem Statement

While `obra/superpowers` is popular, it does not attempt to solve the same problems as this approach, which are:
- Be able to lock-in architectural project-wide and team/org-wide decisions
- Automatically keep codebase consistent with the architectural design
- Automatically keep architectural design itself consistent
- Automatically perform design and code quality control
- Make heavy refactoring easy

# AIPMv1 Solution


## Design docs

Enter design docs. They become a part of your project repo. They also become the source for truth in the sense that if the code contradicts anything in them, the code is wrong.

The hierarchy of authority goes like this: `design.md` -> `plan.md` derived from `design.md` -> source code derived from `plan.md`.

The source code can never contradict `plan.md` or `design.md`, `plan.md` can never contradict `design.md`, and no `design.md` files can contradict other `design.md` files.

The filesystem hierarchy of `design.md` files is the following:
```
workspace/
  doc/design.md - defines decisions that apply to all project members
  member1/
    doc/design.md - defines decisions that are specific to member1, inherits all parent design.md
  aggregating_dir_1/
    doc/design.md - defines decisions that are specific to all subprojects in this directory, inherits all parent design.md
    member2/
      doc/design.md - defines decisions that are specific to member2, inherits all parent design.md
    member3/
      doc/design.md - defines decisions that are specific to member3, inherits all parent design.md
```

There's a bunch of automatically enforced policies for what things `design.md` docs can and cannot mention to keep them free from spaghetti logic, copy/paste decisions, drift, etc.

One important thing to understand about design docs: if something is not in them, that means you don't care how that part gets implemented and it can change at any time.

## Plans

When design docs are locked in, it's time to plan an impl. There is a bit of a chicken and egg problem in the sense that some design decisions are only possible to lock in after you try to plan an implementation. This is fine. The entire process is iterative: you write whatever thoughts/wishes you've got into an initial `design.md`, then ask the AI what decisions it thinks needs to be considered before an implementation is started, and it will give you a lot of food for thought, and you can pick and choose which of that neeeds to go into the design doc and which you don't care about.

If you do care about something to be done in a specific way, this has to be written down in the design doc and the pre-planning stage can help you figure out what that is.

After you settle down with design docs for real, ask the AI to write down implementation plans for it. These plans will always be within the constraints of the design docs.

The plans will also be split into phases. Phases exist to solve two problems:
1. Give you an opportunity to steer development between phases. A phase is done, you can review the code, maybe run the app for some visual verification, give feedback to the AI if you want something done differently. Again, at that point you might realize a bit more stuff should go into the design docs and not be left to AI's random choice.
2. Have work split into smaller, individually implementable pieces to be able to do solve it in a single context window.

## Subagents

Subagents are great for efficient use of the main context window - the main agent offloads exploratory work to subagents, where subagents hand off the useful and very short summary of their investigation back to the main context window. It's often a difference between being able to plan a task in a single context window or not, if the AI has to explore a lot of code.

## Environmental facts

Workspace root `ENV.md` is used to store things that the agent needs, but are only true for that particular environment, such as credentials, hostnames or whatever else. AIPMv1's instructions forbid committing that file and if it's discovered to be part of a VCS, that is an immediate and fatal error.

## Example workflows

### For a new project

Drop `AGENTS.md` into the project root. Run Codex CLI configured for gpt-5.4/xhigh.

You: I want to build a GUI client for the game Achaea. Let's create a design.md for that.

AI Creates the `doc/design.md` with the `# Goals` filled.

You: I want it to be built in Rust. And here are the UI ideas I've got: (removed for brevity).  Add these UI ideas to the design doc.  And to implement them, what popular, battle-tested crates would you suggest? I want good runtime performance and low memory footprint.

AI suggests crates, lists their tradeoffs.

You: Let's try `iced` then. Pin that down.

AI adds `iced` as the chosen GUI library to `design.md`.

You: What else do we need to figure out before implementing the client?

AI mentions GMCP, MXP, terminal emulation support, telnet protocol support and many other things. You talk this over for hours, periodically asking the AI to write down agreed-on decisions to `design.md`.

You: Anything else we have to pin down before proceeding with an impl?

AI: not really, all keys decisions have been made.

You: Go ahead and plan an impl then.

AI creates a bunch of `plan.md` in your project.

You: Go ahead and implement the next top level plan.md unfinished phase.

AI does that. You review the code. Optionally steer (which leads to updates to design.md and plan.md files).

You continue until all plans are finished.

### For an existing project that you want to onboard to AIPMv1

Drop `AGENTS.md` in there. Depending on how familiar you are with the codebase, either do a full AI-based reverse engineering of code into the design docs, or a hybrid approach: for example, spell out `# Goals` for each workspace member manually, but have the decisions reverse-engineered by AI.

Then do `design.md` cleanups - drop all the things that you don't care about from them. For example, what specific library must be used to implement something, or names of internal types that you don't really care about because they're not part of a public API boundary, etc.  Just to cut down on the noise.

### For an existing project that you don't want to onboard to AIPMv1

Create a new workspace, drop `AGENTS.md` in there. Add `ENV.md` that explains where the target project(s) are located. Run your agent.

Follow the same process described earlier - design docs, plans.
