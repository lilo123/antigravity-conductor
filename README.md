# Conductor plugin for Antigravity

A premium model (e.g. Claude Opus) plans the work and checks the results.
Cheaper Gemini subagents do the reading, editing, building and testing.

## What's inside

-   `skills/conductor`: the playbook the main model follows (`/conductor`).
-   `agents/conductor-scout`: read-only researcher (flash).
-   `agents/conductor-worker`: implementer (flash).
-   `agents/conductor-reviewer`: fresh-eyes reviewer (pro).

## Install

```bash
git clone https://github.com/lilo123/antigravity-conductor ~/.gemini/config/plugins/conductor
```

Restart Antigravity, open Customizations and check that **Conductor** shows up
with 1 skill and 3 agents.

## Use

Pick Claude Opus as the model, then start your request with `/conductor`.

## Update

```bash
git -C ~/.gemini/config/plugins/conductor pull
```
