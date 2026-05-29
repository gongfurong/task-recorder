---
name: task-recorder
description: Use for task-managed work and progress recording. Strong activate only when the message begins with an explicit task prefix such as task:, Task:, 任务:, 任务：, 开始任务:, 继续任务:, 读取任务:, or when a .tasks/*.md task record is provided. Weak activate for complex/resumable work or semantic intent in any language to track, preserve progress, create/maintain records, convert ongoing work into a managed task, or continue later. IMPORTANT: weak activation must only ask whether to use task-recorder first; do not create records, write files, or enter planning/confirmation until the user answers the opt-in prompt. Listed trigger words are routing hints, not exclusive matches. Supports multilingual task records, recovery, and continuation.
---

# Task Recorder

This skill makes the assistant maintain a durable task record while doing real work. The record is the handoff document for long tasks, interrupted sessions, and future conversations.

## Core Contract

- If activation is weak, the first response must be an opt-in question. Do not skip the opt-in question.
- Use this skill for work that benefits from managed execution, durable progress tracking, verification, decision records, or reliable continuation across sessions.
- Do not use this skill for simple Q&A, casual chat, tiny one-step answers, or explanations that do not require execution, unless the user explicitly asks to record the task.
- The default task record directory is `./.tasks/`.
- The task record is the durable source of truth. In-session todo tools can be used for working memory, but the markdown record must be updated at meaningful checkpoints.
- Do not create a task record before the user confirms the plan, unless the user explicitly asks to create a draft record.
- Weak activation is opt-in only. If the message does not begin with a valid task prefix and does not provide a task record, never create or update task records until the user explicitly chooses to enable task-recorder.
- Do not execute user-impacting project work before the user confirms the understanding and plan, except for safe read-only inspection needed to understand the request.
- If the task scope changes materially, pause, update the record, present the revised understanding and plan, and wait for confirmation before continuing.
- For task-recorder work, substantial deliverables must be saved to durable artifacts unless the user explicitly requests chat-only output.
- When continuing, modifying, or extending an existing task, preserve the original goal and plan history. Add the new goal as an extension, revision, or follow-up scope instead of overwriting the previous goal.

## Language Behavior

Support users in any language without requiring them to use English.

- Infer the working language from the user's task prefix and task content.
- Use the user's dominant language for user-facing responses, task records, progress notes, decisions, blockers, and summaries.
- Localize all user-facing template headings, confirmation prompts, option labels, status summaries, and placeholder text into the user's working language when displaying them or writing task records.
- Keep this skill document's examples and placeholders as implementation guidance, not literal text to show to users.
- When creating actual user-facing output or task records, replace every bracketed placeholder such as `[Localized ...]`; never leave template placeholder text in real records or responses.
- If the user explicitly requests a language, use that language.
- If the input is multilingual, use the language that best serves the user, usually the dominant natural language in the request.
- Keep professional technical terms, code identifiers, API names, file paths, commands, error messages, logs, package names, class names, function names, protocol names, and domain-standard terminology in their original form unless translation improves clarity.
- Do not translate code, commands, configuration keys, file paths, stack traces, or exact external messages.
- When recording changes, preserve exact filenames, symbols, command text, and verification output references.

## Trigger Behavior

Use three trigger levels: strong activation, weak activation, and no activation.

### Strong Activation

Strong activation starts the task-recorder flow immediately.

Activate strongly when either condition is true:

- the user's message begins with a task prefix;
- the user directly provides, attaches, pastes, or references a task record file/path/content that appears to be a `.tasks/*.md` task record.

Prefix matching is case-insensitive where the language has letter case. Both English `:` and Chinese `：` colons are valid.

Recognized prefix meanings:

- task start: `task:`, `task：`, `任务:`, `任务：`, `start task:`, `start task：`, `开始任务:`, `开始任务：`;
- task continuation: `continue task:`, `continue task：`, `resume task:`, `resume task：`, `继续任务:`, `继续任务：`;
- task record reading: `read task:`, `read task：`, `task record:`, `task record：`, `读取任务:`, `读取任务：`, `任务记录:`, `任务记录：`.

Equivalent multilingual task prefixes are allowed when they clearly appear at the very beginning of the message and end with `:` or `：`.

Do not strongly activate this skill when task-related words appear later in the message without a valid beginning prefix, unless the message provides a recognizable task record.

### Weak Activation

Weak activation does not create records or execute the task. It only asks whether the user wants to use the task-recorder workflow.

Weakly activate when the user has not used a valid task prefix and has not provided a task record, but the goal is likely complex enough to benefit from managed execution.

Use weak activation when the request appears to require planning, sequencing, tradeoff decisions, persistent outputs, verification, coordination across multiple artifacts or environments, or future resumption. Do not use weak activation for ordinary single-answer requests or small one-step changes.

Also weakly activate when the conversation is not currently using task-recorder and the user's semantic intent is to start tracking the current work, preserve progress, convert ongoing work into a managed task, make the work continuable later, or otherwise create durable task context, but does not use a strong task prefix. Do not rely on exact keywords for this. In this case, do not create a record immediately; ask whether to adopt the current work into task-recorder.

Non-prefix recording, tracking, preservation, continuation, or managed-task intent is weak activation, not strong activation. The first assistant response for weak activation must be only the opt-in prompt plus any minimal context needed to explain it. Do not create files, do not create `.tasks/*.md`, and do not enter the confirmation/planning flow until the user opts in.

For weak activation, respond in the user's language with a concise opt-in prompt:

```markdown
**[Localized heading: Task Management Recommended]**
[Localized sentence explaining that this looks like a multi-step task and can be managed with `task-recorder`.]

1. [Localized option: Use task-recorder (recommended)] - [localized short benefit]
2. [Localized option: Continue normally] - [localized short consequence]

[Localized instruction: Reply with 1 or 2.]
```

If the user chooses option 1 or otherwise agrees, follow the Required User Confirmation Flow. If the user chooses option 2 or declines, proceed normally without task records. A weak-trigger message itself is not consent; consent must come after the opt-in prompt.

### Mid-Conversation Adoption

When the user opts into task-recorder for work that has already started outside this skill:

- summarize the inferred task goal, completed work, current state, likely next step, known uncertainties, and proposed record path;
- clearly mark any inferred details that are not confirmed by the user;
- ask for confirmation before creating the task record;
- after confirmation, create the task record with the first `Stage Log` entry marked as adopted from prior conversation context;
- if the previous context is insufficient or ambiguous, ask concise clarification questions before creating the record;
- do not pretend that earlier unrecorded work was fully tracked. Record it as a best-effort reconstruction from available context.

Use this opt-in prompt for mid-conversation adoption:

```markdown
**[Localized heading: Enable Task Recording?]**
[Localized sentence: I can adopt the current work into `task-recorder` so it can be resumed later.]

1. [Localized option: Enable task-recorder] - [localized note: I will summarize the current context, ask for confirmation, then create a task record.]
2. [Localized option: Do not enable] - [localized note: Continue without creating a task record.]

[Localized instruction: Reply with 1 or 2.]
```

### No Activation

Do not activate or ask to activate for ordinary Q&A, casual chat, simple explanations, small one-step edits, simple command results, or review-only requests unless the user uses a strong trigger or provides a task record.

### Task Record Recovery Activation

If the user provides a task record directly, activate this skill even without a task prefix.

Recognize task records by signals such as:

- a path matching `.tasks/*.md`;
- markdown content with task-record sections such as Metadata, Overview, Stage Log, Goal And Scope History, User Requirements, Changes, Verification, Resume Notes, or Final Summary, localized into any language;
- a file that clearly describes a recorded task, status, progress, and next step.

When a task record is provided:

- read and summarize the task goal, status, progress, decisions, changes, verification, blockers, and resume notes;
- do not restart planning from scratch unless the record is missing, inconsistent, or the user requests a new scope;
- when the user asks to modify, extend, continue, or add scope to the task, preserve the existing goal and add a new scope entry that explains the relationship to the previous goal;
- if the task is `completed`, explain that it appears complete, summarize the result, and ask whether the user wants follow-up work or a new task;
- if the task is `failed`, `blocked`, `draft`, `confirmed`, or `in_progress`, explain the current state and guide the user toward the next safe action;
- if continuing requires a decision or scope change, ask before executing;
- if the next step is already clear and the record is confirmed or in progress, ask whether to continue from that next step.

## File Naming

Create task records under `./.tasks/` using:

```text
.tasks/task-slug.md
```

Rules:

- Derive the filename from the task name in the user's working language.
- Do not translate the task name to English just for the filename.
- Preserve professional terms, product names, framework names, and code identifiers when useful.
- Use a concise, readable filename. Non-English filenames are allowed when the filesystem supports them.
- Sanitize characters that are invalid for the current filesystem.
- Replace path separators and unsafe punctuation with `-` or omit them.
- Keep the slug short and stable.
- Do not add a date or time prefix by default.
- If a file already exists for a different task, append a timestamp suffix such as `_YYYYMMDD-HHMMSS`.
- If continuing an existing task, update the existing file instead of creating a new one.

## State Model

Use these task-level states:

- `draft`: understanding and plan are being prepared; not confirmed.
- `confirmed`: user approved the plan; execution may begin.
- `in_progress`: work is actively happening.
- `blocked`: work cannot continue without external input, access, dependency, or decision.
- `completed`: work and verification are complete.
- `failed`: work ended unsuccessfully and no immediate continuation is planned.

Use these item-level states in `Progress`:

- `[ ]` not started
- `[~]` in progress
- `[x]` completed
- `[!]` blocked or failed

## Required User Confirmation Flow

When a new task starts through strong activation, or after the user opts into task-recorder from weak activation, first inspect only what is needed to understand the request. Then present the current understanding, user requirements, draft plan, expected deliverables, and open questions without creating a task record yet.

Respond with this structure and wait for confirmation:

```markdown
**[Localized heading: Understanding]**
[Localized concise restatement of the goal, constraints, and expected result.]

**[Localized heading: Proposed Plan]**
1. [Localized step]
2. [Localized step]
3. [Localized step]

**[Localized heading: Confirmation Needed]**
- [Localized] After confirmation, the task record will be created at `.tasks/task-slug.md`; if a name collision occurs, append a timestamp suffix.
- [Localized] Main deliverables will be written to durable artifact locations; chat will only show summaries, locations, key decisions, and items needing confirmation.
- [Localized] During execution, I will update goal, plan, progress, decisions, changes, verification, and resume notes automatically.
- [Localized] Please confirm whether to execute this plan. If changes are needed, state them directly.
```

Do not create or modify task records or non-record project files before confirmation, except when the user explicitly asks to create a draft record or asks to continue an existing confirmed task record.

### Confirmation Revision Loop

During the confirmation phase, treat the plan as unconfirmed until the user gives a pure execution approval.

Pure execution approval is semantic, not keyword-based. It means the user's message only confirms or instructs execution, without adding requirements, constraints, preferences, questions, options, corrections, or scope changes. It can be expressed in any language or short approval form, including meanings like `ok`, `yes`, `go`, `do it`, `run`, `execute`, `start`, `confirm`, `approved`, `好`, `可以`, `执行`, `开始`, `确认`, and equivalent wording.

If the user adds any extra content during confirmation, do not execute yet. Instead:

- incorporate the new content into the requirements and plan;
- preserve prior goals and plans if this is a continuation, modification, or extension of an existing task;
- re-output the latest understanding and revised plan;
- ask for confirmation again;
- do not create or update the task record yet unless the user explicitly requested a draft record.

If the user message mixes approval with any additional requirement, constraint, preference, correction, question, or scope change, treat it as a revision, not approval. Re-present the latest understanding and plan instead of executing.

If the user's intent is ambiguous, ask a short clarification question instead of executing.

## Output And Artifact Policy

For task-recorder work, chat output should be a control surface, not the main storage for task results.

Substantial outputs must be saved to durable artifacts unless the user explicitly requests chat-only output or the environment makes persistence impossible. A durable artifact is any user-accessible location that can be reopened, edited, versioned, shared, or continued later.

Use durable artifacts for target outputs, supporting materials, substantial analysis, operational instructions, examples, or any content the user or a future assistant may need to revisit.

Before writing deliverables, include the proposed artifact locations in the confirmation plan unless they are already obvious from the user's request or existing project conventions.

In chat/CLI output, keep only:

- concise understanding and plan;
- decision prompts;
- progress summaries;
- file paths written or changed;
- verification results;
- blockers and next actions.

Do not dump long final deliverables into chat when they can be saved to a durable artifact. If the user explicitly asks for inline output, provide it inline and still offer to save it to a durable location when it is substantial.

## Decision Guidance

Be proactive and guiding, not passive.

When there are multiple reasonable approaches and the best choice is not obvious:

- Ask a concise question before executing.
- Present 2 to 4 options.
- Number every option.
- Include short pros and cons.
- Mark one option as recommended when there is a clear recommendation.
- Explain the recommendation in one sentence.
- Tell the user they can reply with the option number directly.
- Do not hide tradeoffs or silently choose a risky path.

Do not ask the user to decide when one option is clearly better than the alternatives. In that case, choose the clearly better option, briefly state why, and record the decision when it affects implementation, architecture, workflow, or user-visible behavior.

Use this format:

```markdown
**[Localized heading: Decision Needed]**
[Localized description of the decision needed.]

**[Localized heading: Options]**
1. [Option A] ([Localized label: Recommended, if applicable]): [localized benefit]; [localized risk/cost label]: [localized cost]
2. [Option B]: [localized benefit]; [localized risk/cost label]: [localized cost]

[Localized recommendation sentence explaining which option is recommended and why.]
[Localized instruction: Reply with the option number, or describe your own choice.]
```

Autonomous choice is allowed only when all are true:

- one option clearly dominates, or the choice is low risk;
- it follows existing project patterns;
- it is reversible;
- it does not change user-visible behavior beyond the confirmed scope;
- asking would add friction without improving correctness.

Ask for user choice only when the decision has meaningful tradeoffs, unclear priorities, irreversible consequences, cost/time implications, security/privacy implications, or visible product/design impact not already covered by the confirmed plan.

If user action is required outside the assistant's tool access, guide step by step. State exactly what the user should do, how to verify it, and what result to report back. Record that handoff in the current `Stage Log` entry and `Resume Notes`.

## Deliverable Depth

Produce the target deliverable, not just an explanation of it.

- Create or modify the actual target files or external artifacts whenever the goal can be directly materialized.
- Choose the most concrete maintainable artifact that fits the context, instead of stopping at a description of the artifact.
- Add companion documentation when it materially improves human or future-AI maintainability by explaining how to understand, use, extend, verify, or continue the work.
- Do not add companion documentation when the target artifact is already documentation, the task is a small localized change, or the documentation would add noise without improving maintenance.
- Produce only explanatory documentation when the user explicitly asks for documentation/design only, when implementation or target-file creation needs missing inputs/tools/access, or when the task is intentionally conceptual.
- If the right deliverable level is unclear, ask a concise decision question or propose a staged plan that creates the most useful artifacts first.
- Record the chosen deliverable level and artifact locations in the current `Stage Log` entry, `Key Decisions` when relevant, and `Resume Notes`.

## Goal And Scope Evolution

Task goals are append-only in spirit, not overwrite-only.

- When a task is continued, modified, extended, narrowed, or re-scoped, preserve the previous goal and describe the new scope as a relationship to it.
- Use relationship labels in the user's language, such as original goal, inherited goal, extension goal, revision, follow-up, replacement, or cancelled scope.
- Do not replace the `Goal` section with only the newest request when previous goals remain relevant.
- If a new request supersedes a previous goal, explicitly mark the old goal as superseded, cancelled, or no longer in scope, and explain why.
- If the previous goal was completed and the user asks for additional work, treat the new work as a follow-up or extension while keeping the completed goal visible.
- Update `Overview`, `Goal And Scope History`, the current `Stage Log` entry, `Key Decisions` when relevant, and `Resume Notes` so a future assistant can see both what was already achieved and what is newly requested.

## Record Layout Model

Use a lightweight append-only stage model.

- Keep the top of the file as the compact source of truth: overall task status, overall goal, current stage, next step, blockers, and key artifacts.
- Keep history in `Stage Log`, appended by stage. Each stage owns its own request, scope relationship, plan, progress, changes, verification, result, and handoff notes.
- When the task continues, expands, or changes, do not rewrite old stages. Add a new stage at the end and update only the top summary/current-stage fields.
- Avoid maintaining the same facts in many places. File changes belong primarily to the stage where they happened. The top summary may mention only important cumulative artifacts.
- Completed stages should be compact but sufficient for future readers: why the stage existed, what was planned, what changed, what was verified, what resulted, and what to know next.
- If a later stage supersedes earlier work, mark that relationship in the new stage and in the top goal evolution summary; do not delete the earlier stage.
- The record should allow a future assistant to quickly read the top section for current state, then scan stage entries for historical detail.

## Task Record Template

Every task record must use this structure. Keep it concise and factual.

```markdown
# [Localized heading: Task]: [Short task name]

## [Localized heading: AI Continuation Instruction]
[Localized instruction for future assistants, while preserving the exact skill name `task-recorder`: This file is a `task-recorder` task record. If the `task-recorder` skill/workflow is already active, continue using it. If it is not active and is available, load or activate it before continuing. Do not duplicate, paste, or re-inject the full skill instructions into the conversation or this file. Treat this file as the source of truth for the task context, summarize status, goal, completed work, next step, blockers/risks, and then guide continuation. If the user asks to continue, resume, inspect, or update the task, continue through the `task-recorder` recovery workflow. Do not restart planning from scratch unless the record is missing, inconsistent, or the user changes scope. Ask before continuing unless the user explicitly instructs execution and the next step is already confirmed.]

## [Localized heading: Metadata]
- [Localized label: Status]: draft
- [Localized label: Created]: YYYY-MM-DD HH:mm
- [Localized label: Updated]: YYYY-MM-DD HH:mm
- [Localized label: Workspace]: [path]
- [Localized label: Task Record]: .tasks/task-slug.md
- [Localized label: Related Branch]: [branch or n/a]
- [Localized label: Related Commits]: [hashes or n/a]

## [Localized heading: Overview]
- [Localized label: Status]: [latest task status]
- [Localized label: Overall Goal]: [localized overall goal, preserving original and active scope]
- [Localized label: Overall Summary]: [localized compact cumulative summary]
- [Localized label: Current Stage]: [localized current stage number/title/status]
- [Localized label: Current Stage Goal]: [localized current stage goal]
- [Localized label: Completed So Far]: [localized compact completed outcomes]
- [Localized label: Next Step]: [localized single best next step]
- [Localized label: Blockers/Risks]: [localized blockers or risks]
- [Localized label: Key Artifacts]: [localized important durable artifacts]

## [Localized heading: Goal And Scope History]
- [Localized label: Original Goal]: [localized original goal]
- [Localized label: Active Goal]: [localized active goal]
- [Localized label: Added / Extended Scope]: [localized added scope by stage, or n/a]
- [Localized label: Superseded / Removed Scope]: [localized removed or superseded scope with reason, or n/a]

## [Localized heading: User Requirements]
- [Localized cumulative requirement, constraint, preference, or explicit non-goal]

## [Localized heading: Expected Deliverables]
- [Localized target artifact, file, external destination, or output category]

## [Localized heading: Key Decisions]
- YYYY-MM-DD HH:mm - [Localized decision] - [Localized label: Rationale]: [localized why]

## [Localized heading: Stage Log]
### [Localized heading: Stage 1] - YYYY-MM-DD HH:mm - [localized short title]
- [Localized label: Request]: [localized original or continuation request]
- [Localized label: Scope Relationship]: [localized original / extension / revision / follow-up / replacement]
- [Localized label: Status]: [localized stage status]
- [Localized label: Plan]: [localized compact confirmed plan for this stage]
- [Localized label: Progress]: [localized compact progress for this stage]
- [Localized label: Blockers/Risks]: [localized blockers or risks for this stage, or n/a]
- [Localized label: Changes]:
  - [Localized label: Added]: `path or artifact` - [localized purpose]
  - [Localized label: Modified]: `path or artifact` - [localized summary]
  - [Localized label: Deleted]: `path or artifact` - [localized reason]
  - [Localized label: Renamed / Moved]: `old` -> `new` - [localized reason]
  - [Localized label: Commands / Configuration]: `command or config item` - [localized why/result]
- [Localized label: Verification]: [localized verification summary]
- [Localized label: Result]: [localized stage result]
- [Localized label: Handoff]: [localized what future assistants need to know]

## [Localized heading: Resume Notes]
- [Localized label: Current Status]: [localized one sentence]
- [Localized label: Next Step]: [localized single best next action]
- [Localized label: Watch Outs]: [localized risks, constraints, or important context]

## [Localized heading: Final Summary]
[Localized final summary. Fill only when completed or failed.]
```

## Update Discipline

Update the task record at meaningful checkpoints, not after every tiny action.

Required updates:

- after the user confirms the plan: create the task record, write the approved plan as the first `Stage Log` entry, update `Overview`, `Goal And Scope History`, `Expected Deliverables`, and set `Status: confirmed`;
- when continuing, modifying, extending, or re-scoping an existing task: append a new `Stage Log` entry and update only the top `Overview`, `Goal And Scope History`, `Expected Deliverables` if needed, `Key Decisions` if needed, and `Resume Notes`;
- before substantial execution begins: set `Status: in_progress` and update the current stage status/progress;
- after each meaningful work unit completes: update the current stage's progress/changes/verification/result as needed, then update the top `Overview` and `Resume Notes`;
- when a key technical decision is made: update `Key Decisions` and the current stage entry if relevant;
- when a command, test, build, lint, migration, or manual check is run: update verification in the current stage entry;
- when blocked: set `Status: blocked`, record the blocker in `Overview`, the current stage entry, and `Resume Notes`, and write the exact next user/assistant action;
- at completion: set `Status: completed`, ensure the top `Overview`, current stage entry, verification, and `Resume Notes` are current, then write `Final Summary`.

Avoid recording:

- routine tool calls with no lasting relevance;
- private chain-of-thought;
- secrets, tokens, credentials, or sensitive personal data;
- unrelated conversation;
- speculative conclusions not backed by inspection or confirmation.

## Git-Style Change Recording

Prefer real repository evidence.

If the workspace is a git repository:

- use `git status` to identify added, modified, deleted, renamed, and untracked files;
- use `git diff --stat` or targeted diffs to summarize changes;
- do not claim commits exist unless they were actually created;
- record the branch and commit hashes only when available and relevant;
- never revert unrelated user changes.

If the workspace is not a git repository:

- still record changes under Added, Modified, Deleted, Renamed / Moved, and Commands / Configuration;
- note `Related Branch: n/a`.

Stage-level change entries are concise change logs, not full pasted diffs. Include full diff snippets only when they are essential for handoff.

## Resume Behavior

When reading an existing `.tasks/*.md` record, immediately summarize the task state before doing more work:

```markdown
**[Localized heading: Task Record Loaded]**
- [Localized label: Status]: [Status]
- [Localized label: Goal]: [localized goal in one sentence]
- [Localized label: Completed]: [localized key completed items]
- [Localized label: Next Step]: [localized next step from Resume Notes]
- [Localized label: Blockers/Risks]: [localized issues or watch outs]

[Localized sentence: I can continue with the next step. If you want to adjust the goal or scope, please say so first.]
```

If `Status` is `draft`, ask for plan confirmation before execution.

If `Status` is `confirmed`, `in_progress`, or `blocked`, continue only after resolving any blocker or decision need.

If the record is stale or inconsistent, state the inconsistency, inspect the workspace, repair the record with factual updates, and ask for confirmation when the next action changes scope.

## Planning Quality Rules

Plans should be practical and minimal.

- Prefer the smallest correct change.
- Use existing project conventions before inventing new structure.
- Break tasks into enough steps to support handoff, but avoid artificial micro-steps.
- Include verification in the plan.
- Include cleanup when temporary files, generated artifacts, or experiments are expected.
- Mark assumptions explicitly.
- Ask only decision-relevant questions; do not block on trivial preferences.

## User Guidance Rules

When the user seems unsure, provide a guided path:

- explain what needs to be decided;
- explain why it matters;
- recommend a default;
- give exact next action choices;
- keep the explanation brief enough that the user can decide quickly.

When the user must do something manually:

- provide numbered steps;
- include the expected result;
- include what to send back;
- record the handoff in the task record.

## Completion Criteria

A task is complete only when:

- the confirmed goal is satisfied or the remaining gap is explicitly documented;
- relevant files, configuration, durable artifacts, or external destinations are updated;
- verification has been attempted and recorded;
- known issues and residual risks are documented;
- `Resume Notes` clearly says no next execution step remains, or states the next optional follow-up;
- `Final Summary` explains what changed and how to validate/use it.
