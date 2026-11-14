# pstack in Claude Code

pstack skills use Cursor names. In Claude Code, read this file once per session, then map each Cursor name in a skill with the tables below. `<pstack>` means the folder that holds `skills/` and `docs/`. If a Cursor field has no row, drop it and say so. Never pass a field or value the tool schema rejects.

## Tools

| Cursor | Claude Code |
| --- | --- |
| `Task` | `Agent` |
| `AskQuestion` | `AskUserQuestion`. In `-p` mode it is absent, so ask in plain text. |
| Open a todolist | Use the session's task-list tool: `TodoWrite`, `TaskCreate`/`TaskUpdate`, or the host's own plan tool such as `update_plan`. Use whichever the session offers. Otherwise keep the list in your reply and update it at each phase. |
| Run another pstack skill (`/name` or "the **name** skill") | Most pstack skills set `disable-model-invocation: true`, so the `Skill` tool refuses them and the session's skill list omits them. Try `Skill` with `pstack:<name>` only when the list shows it. Otherwise `Read` `<pstack>/skills/<name>/SKILL.md` in full and follow it as if it had loaded, including its own adapter line. Do not tell the user it is unavailable. When you tell the user to type it, write `/pstack:<name>`, which works because the block only stops the model. |
| `Grep`, `Glob` | `Bash` with `rg` when the session has no `Grep` or `Glob`. |

## Subagent fields

| Cursor | Claude Code |
| --- | --- |
| `subagent_type: generalPurpose` | `subagent_type: "general-purpose"` |
| `poteto-agent`, `comment-sicko` | `pstack:poteto-agent`, `pstack:comment-sicko` |
| `readonly: true` | `subagent_type: "Explore"`. It keeps `Bash`, so the brief also says "do not edit files", and you check `git status` afterward. |
| `readonly: false` | `general-purpose`. It keeps the session's MCP servers. |
| `environment: "cloud"` | `isolation: "worktree"`. Name the branch or commit the worker checks out. |
| `environment: "local"` | Omit. |
| `cloud_base_branch: <b>` | Tell the worker to check out `<b>` first. |
| `run_in_background: true` | Omit. Subagents run in the background by default and notify you when they finish. |
| `resume` | `SendMessage` with the agent's id or name, when the session offers it. That continues the same agent with its context, whether it is still running or has finished, so a steer needs no new spawn. Only when `SendMessage` is absent, spawn a new agent and paste the earlier result into its brief. |
| `file_attachments` | List the paths in the brief. |

## Models

| Cursor | Claude Code |
| --- | --- |
| `claude-<family>-<version>-<effort>[-fast]`, such as `claude-opus-5-5-xhigh` | `model: "opus"` and `effort: "xhigh"`. `model` takes only `opus`, `sonnet`, `haiku`, `fable`. Drop `-fast`. |
| A `grok-*`, `gpt-*`, or other non-Claude slug | A single role (`swarm workers`, a code delegate): omit `model` so the agent runs on the session model, and say so. A seat in a panel list (`arena runners`, `arena cross-judge pool`, `architect runners`, `interrogate reviewers`): see Panels below. |
| `auto` or `inherit-parent` | Omit `model`. |
| The `pstack-models.mdc` rule, `~/.cursor/rules/pstack-models.mdc` | `~/.claude/rules/pstack-models.md`. Claude Code loads it into every new session, so its role lines are already in context. If they are not, read the file. If the file or the role's line is missing, use the skill's default slug. |
| A role value `<model> <effort>`, such as `opus high` | `model: "opus"` and `effort: "high"`. |

## Panels

Panels exist to get different models' blind spots. Claude Code only runs Claude models, so the adapter keeps panels diverse across Claude model names, not across vendors.

- **Family is the model name.** For the family checks in `arena` and `swarm`, `opus`, `sonnet`, `fable`, and `haiku` are four families. Learn the parent's model from the session's model line in the system prompt. If you cannot tell, say that the family check did not run.
- **Replace a non-Claude seat in a panel** with the first of `sonnet`, `fable`, `opus`, `haiku` that no other seat in that panel already uses, at the seat's effort capped at what the `Agent` schema lists. Use `haiku` only when the three stronger ones are taken. Say which model replaced which slug.
- **A panel of `inherit-parent` or `auto` seats** is the one case that stays on the parent model. Do not spread it.
- **Cross-judge.** The `arena` judge must not be the parent's model. Choose the first pool entry whose mapped model differs from the parent's. When the whole pool maps to the parent's model, switch the judge to the strongest other model the `Agent` schema lists and say so. Skip the judge only when no other model is listed.
- **`swarm` model race.** Name each arm's model up front, drawn from the same four names, one arm per model, and run extra arms on repeats of the same model.

## setup-pstack

Claude Code loads every `.md` file in `~/.claude/rules/` into each new session, so `setup-pstack` writes its rule there. Follow its steps with these changes:

- Detect. The detected models are the `model` values that the `Agent` tool schema lists, each with an effort from its `effort` values. Write a real model as `<model> <effort>`, such as `opus xhigh`. The effort is the last token, so the budget step works as written.
- Defaults. Map each default slug with the rows above: `claude-opus-5-5-xhigh` is `opus xhigh`, and `grok-4.7-xhigh-fast` is `inherit-parent` for a single role, and the next free Claude model for a panel seat (see Panels).
- Write. Write `~/.claude/rules/pstack-models.md` with no frontmatter, then the comment lines, the `# budget` line, and one line per role from step 5. Overwrite the whole file. Never put the role lines in `CLAUDE.md` or `AGENTS.md`.

## Paths

| Cursor | Claude Code |
| --- | --- |
| `.cursor/skills/`, `~/.cursor/skills/` | `.claude/skills/`, `~/.claude/skills/` |
| `~/.cursor/plugins/`, `.cursor-plugin/plugin.json` | `~/.claude/plugins/`, `.claude-plugin/plugin.json` |
| `.cursor/rules/*.mdc`, always-applied rules | `CLAUDE.md`, or `AGENTS.md` when no `CLAUDE.md` exists. Claude Code never reads `.mdc` files. The model rule is the exception. See Models. |
| `https://github.com/cursor/plugins/blob/main/pstack/<path>` | `<pstack>/<path>` when that file exists. The plugin ships the skills, `docs/guide/`, and `README.md`. Read the local copy and never fetch it. Give the user the URL when they need a link. |
| Transcripts in `~/.cursor/projects/<slug>/agent-transcripts/` | `~/.claude/projects/<slug>/<session>.jsonl`. The slug is the workspace path with each `/` and `.` changed to `-`, so `/Users/you/proj` is `-Users-you-proj`. The newest `.jsonl` directly in that folder is the current session. Its `<session>/subagents/` folder holds subagent transcripts. Read only this workspace's folder. |

## Cursor-only features

Claude Code has no Cursor Automations, Bugbot, Cursor cloud agents, or built-in PR tool. When a step needs one, put one line in your reply, `Skipped in Claude Code: <step>. <reason>.`, then go on to the next step. Do not stop, ask, or retry. When a later step needs what the skipped step made, such as a URL or a key, leave a placeholder and name it in the line.

| Cursor | Claude Code |
| --- | --- |
| Cursor Automations: routine wakes and recurring checks | Map to Claude Code scheduling when the session offers it: `ScheduleWakeup` or `/loop` for a self-paced check, `CronCreate` or the `schedule` skill for a cron or one-time run. Say which one you used. Do not poll in a loop the harness already tracks. |
| Cursor Automations: `update_state` routine calls, `api2.cursor.sh/automations` webhooks, and `SendToUser` secret requests | Skip with the line. |
| Bugbot | Bugbot comments on a GitHub PR are ordinary review threads, so triage them with the forge command that the step gives. Skip a step that starts, waits for, or configures Bugbot when the PR has no Bugbot check or comment. |
| Cursor cloud agents | `environment: "cloud"` maps to a worktree, as in Subagent fields. Skip a step that needs the cloud agent itself, such as its dashboard status or its own machine. |
| A built-in PR tool | Use the forge command that the step gives, such as `gh pr create`. That is not a skip, so write no line. |
