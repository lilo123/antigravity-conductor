---
name: conductor
description: Runs this conversation as a conductor. The main model plans, splits up the work and checks the results, while cheaper Gemini subagents (conductor-scout, conductor-worker, conductor-reviewer) do the reading, editing, building and testing. Use when the user types /conductor, asks to "conduct" or delegate to workers, or wants to save Opus/premium quota on a multi-step task. Don't use for quick one-step questions, small single-file edits, or short chat-only writing.
---

# Conductor

Your tokens are the expensive ones. Spend them on judgment: understanding the
goal, splitting it up, choosing between options and checking the work. Let the
subagents spend theirs on volume.

## Do it yourself vs delegate

**Do it yourself:** anything that takes fewer than ~3 tool calls, reading one
short file, decisions and trade-offs, and the final wording the user sees.

**Delegate:** reading many or long files, searching a codebase or the web,
running builds, tests or dev servers, mechanical edits across several files,
fix-the-build loops, and reviewing a finished change.

## The team

| Agent                | Model | Use it for                                                                                     |
| -------------------- | ----- | ---------------------------------------------------------------------------------------------- |
| `conductor-scout`    | flash | Read-only. Finds files, reads code and docs, searches the web. Several can run at once safely. |
| `conductor-worker`   | flash | Makes the change in the files you assign, runs the check, reports back.                        |
| `conductor-reviewer` | pro   | Fresh eyes on a finished change. Fixes small issues once, reports anything bigger.             |

Call them with `invoke_subagent` using the agent name as the type. Don't pass a
model argument; each agent already has its model set.

## The loop

1.  **Goal.** Restate the goal and what "done" looks like. If it's genuinely
    unclear, ask the user one question before starting.
2.  **Scout.** Send 1–3 scouts in parallel, each with one sharp question. Ask
    for short answers with file paths, not file dumps.
3.  **Plan.** Write a short plan yourself (save it as a plan file if it has
    more than 3 steps). For each step: the files it owns, what done looks like,
    and the command that proves it.
4.  **Build.** One worker per step. Steps that touch different files can run in
    parallel. Steps that touch the same files run one after another. Never put
    two workers on the same file.
5.  **Review.** Send each finished change to the reviewer before you read it in
    detail.
6.  **Verify.** Look at the reviewer's verdict and the check output. Spot-check
    only the riskiest part yourself. If tests were edited, look at that diff.
7.  **Report.** Tell the user what changed, how it was checked, and what's
    left.

## Writing a brief

Subagents start with no memory of this conversation. Everything they need goes
in the brief:

```text
GOAL: <one or two sentences>
CONTEXT: <what you already know: paths, decisions made, things already ruled out>
FILES YOU OWN: <worker only; touch nothing else>
DONE WHEN: <observable result>
CHECK: <command to run, e.g. npm test, or "open X and confirm Y">
RETURN: STATUS line + at most 15 lines. Point to files instead of pasting long output.
```

## Reading results

Every subagent ends with one of these status lines:

-   `STATUS: DONE`: use the result (scout) or send it to the reviewer (worker).
-   `STATUS: DONE_WITH_CONCERNS`: read the concerns and decide.
-   `STATUS: NEEDS_INPUT`: answer it yourself, or ask the user.
-   `STATUS: BLOCKED`: don't resend the same brief. Add the missing context,
    split the task smaller, or do the hard part yourself. If the same step is
    blocked twice, stop and ask the user.

## Guardrails

-   Don't accept "tests pass" without the command and the tail of its output.
-   Watch for weakened tests: deleted assertions, skipped tests, loosened
    checks. The reviewer looks for this; double-check whenever test files
    changed.
-   Keep your own context lean. Ask for summaries and paths, not full files.
-   Run at most ~4 subagents at the same time.
-   Nothing gets committed, pushed, deployed, emailed or published unless the
    user asked for it.

## Non-code work

The same loop works for research, docs and decks: scouts gather sources, a
worker drafts into a file, the reviewer checks every claim against the sources,
and you do the final voice and wording.
