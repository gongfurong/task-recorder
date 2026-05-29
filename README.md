# task-recorder

English | [Simplified Chinese](README.zh-CN.md)

`task-recorder` is a portable AI skill that turns long or important AI work into a managed, resumable task.

It makes the assistant ask before executing, keep a durable Markdown task record, preserve goal/scope history, append stage-by-stage progress, record decisions and verification, and leave enough context for a future AI session to continue without rereading the full conversation.

It is designed for AI CLIs and assistants that support Markdown skills, including Claude Code and OpenCode.

## Why This Skill Is Useful

AI coding sessions often fail in predictable ways: context gets long, requirements shift, progress is scattered in chat, the assistant forgets previous decisions, or a new session cannot safely continue. `task-recorder` solves that by making the task record the durable handoff point.

Key benefits:

- Reliable continuation: start a new conversation and continue from `.tasks/*.md` instead of re-explaining everything.
- Safer execution: the assistant must present understanding and a plan before executing meaningful work.
- Less context waste: current status stays at the top, while history is appended by stage.
- Better change tracking: each stage records its own changes, verification, result, and handoff notes.
- Scope-safe evolution: extensions and revisions do not overwrite earlier goals or completed stages.
- Durable outputs: substantial work should be saved to files or other durable artifacts, not only printed in chat.
- Multilingual by design: user-facing records follow the user's language while preserving exact code, paths, commands, logs, and technical terms.
- Portable: the core behavior lives in one `SKILL.md` file.

## What It Records

Task records are written under `.tasks/` after the user confirms the plan.

The record uses a lightweight structure:

- `AI Continuation Instruction`: tells future assistants how to resume safely.
- `Metadata`: status, timestamps, workspace, record path, branch/commit references.
- `Overview`: compact current source of truth: status, overall goal, current stage, next step, blockers, key artifacts.
- `Goal And Scope History`: original goal, active goal, added scope, superseded scope.
- `User Requirements`: cumulative constraints, preferences, and non-goals.
- `Expected Deliverables`: files or external artifacts expected from the task.
- `Key Decisions`: important decisions and rationale.
- `Stage Log`: append-only stage history. Each stage owns its request, scope relationship, plan, progress, changes, verification, result, and handoff.
- `Resume Notes`: short handoff for the next session.
- `Final Summary`: written only when the whole task is completed or failed.

## Activation Modes

### Strong Activation

Strong activation starts the task-recorder flow immediately.

Use an explicit task prefix at the beginning of the message:

```text
task: implement a CLI parser
Task: refactor the auth flow
start task: organize a release workflow
continue task: .tasks/unity-fsm.md
read task: .tasks/release-plan.md
```

Providing a `.tasks/*.md` task record also activates recovery behavior.

### Weak Activation

Weak activation is opt-in only. The assistant should ask whether to use `task-recorder` first, and must not create records or execute through the skill until the user agrees.

Weak activation applies when the work is complex or resumable, or when the user semantically asks to preserve progress, create/maintain a task record, convert ongoing work into a managed task, or continue later.

Example:

```text
Please preserve the current progress so we can continue in a new chat later.
```

Expected assistant behavior:

```text
This looks like work that can benefit from task recording.

1. Use task-recorder - create a managed record after confirmation.
2. Continue normally - no task record.

Reply with 1 or 2.
```

### No Activation

Simple Q&A, casual chat, one-step edits, and ordinary explanations should proceed normally unless the user explicitly requests task recording.

## Typical Workflows

### New Managed Task

1. User starts with `task:` or another task prefix.
2. Assistant inspects only what is necessary.
3. Assistant presents understanding, plan, expected deliverables, and task record path.
4. User confirms or revises.
5. Only after confirmation does the assistant create `.tasks/task-name.md` and execute.
6. The assistant updates the record at meaningful checkpoints.

### Complex Work Without Prefix

1. User asks for complex or resumable work without a task prefix.
2. Assistant asks whether to use `task-recorder`.
3. If user opts in, the normal confirmation workflow begins.
4. If user declines, the assistant proceeds normally without task records.

### Mid-Conversation Adoption

If work started without this skill and the user later asks to preserve progress, the assistant should ask whether to adopt the current work into `task-recorder`.

After opt-in, the assistant summarizes inferred context, marks uncertainties, asks for confirmation, then creates a record. Earlier unrecorded work is recorded as a best-effort reconstruction, not as fully tracked history.

### Continue From Existing Record

Give the assistant a `.tasks/*.md` record or path:

```text
continue task: .tasks/unity-fsm.md
```

The assistant should summarize status, goal, completed work, next step, and blockers before continuing.

## Installation

Runtime installation only requires `SKILL.md`. The rest of this repository is documentation for humans and GitHub.

Recommended workflow:

1. Clone this repository so you can read the docs and pull updates later.
2. Copy or symlink `SKILL.md` into your target skills directory.

Minimal workflow:

1. Download or copy only `SKILL.md`.
2. Place it in the target skills directory.

README, license, and changelog files do not need to be installed into the skills directory. If you copy the whole `task-recorder` folder into a skills directory, most skill loaders will simply use `SKILL.md`, but installing only `SKILL.md` is the cleanest option.

### Claude Code

Copy or symlink `SKILL.md` to:

```text
~/.claude/skills/task-recorder/SKILL.md
```

### OpenCode

Copy or symlink `SKILL.md` to:

```text
~/.config/opencode/skills/task-recorder/SKILL.md
```

On Windows, the global OpenCode path is usually:

```text
C:\Users\<you>\.config\opencode\skills\task-recorder\SKILL.md
```

Restart the CLI after installing or updating the skill.

## Repository Files

- `SKILL.md`: the actual skill file to install.
- `README.md`: English documentation.
- `README.zh-CN.md`: Chinese documentation.
- `CHANGELOG.md`: notable changes.
- `CONTRIBUTING.md`: contribution guidelines.
- `LICENSE`: MIT license.

## Testing The Skill

After installing and restarting your CLI, try:

```text
task: implement a small utility and keep a task record
```

Expected behavior:

- The assistant presents understanding and a plan.
- It waits for confirmation before creating `.tasks/*.md`.
- After confirmation, it creates a task record and executes.

Weak activation test:

```text
Please preserve the current progress so we can continue in a new chat later.
```

Expected behavior:

- The assistant asks whether to enable `task-recorder`.
- It does not create a record until you opt in.

## Design Principles

- Ask before meaningful execution.
- Weak activation is opt-in only.
- Keep current state at the top of the record.
- Append stage history instead of overwriting old goals, plans, and results.
- Record key facts, decisions, verification, and handoff notes.
- Avoid noisy tool-call logs.
- Prefer durable artifacts for substantial outputs.
- Preserve exact code, commands, paths, logs, and error messages.

## License

MIT. See [LICENSE](LICENSE).
