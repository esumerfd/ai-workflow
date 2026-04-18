---
name: workflow:setup
description: Verify environment variables are set correctly before starting any workflow phase.
---
# Skill: /workflow:setup

**Environment Setup**

Verify that the developer's environment is correctly configured before starting any workflow phase. Check required environment variables, validate paths, and guide the developer through any missing configuration.

Run this once when first using the workflow, or any time a phase fails to locate the workspace.

---

## What This Skill Checks

| Variable | Required | Purpose |
|----------|----------|---------|
| `WORKFLOW_HOME` | Required | Root directory where all feature workspaces are created |

---

## Step 1 — Check WORKFLOW_HOME

Run: `echo $WORKFLOW_HOME`

**If set and the directory exists:**
- Report: "WORKFLOW_HOME is set to `<value>` and the directory exists. ✓"
- Proceed to Step 3.

**If set but the directory does not exist:**
- Report: "WORKFLOW_HOME is set to `<value>` but the directory does not exist."
- Ask: "Should I create it?"
- If yes: `mkdir -p $WORKFLOW_HOME`
- Proceed to Step 3.

**If not set:**
- Proceed to Step 2.

---

## Step 2 — Configure WORKFLOW_HOME

`WORKFLOW_HOME` is not set. Help the developer choose a location and persist it.

Ask:
1. "Where would you like to store workflow workspaces? This directory will hold one subdirectory per feature. Common choices:
   - `~/workflows` — simple, top-level
   - `~/work/workflows` — alongside other work directories
   - A custom path of your choice"

2. Once a path is chosen, confirm: "I'll set `WORKFLOW_HOME=<path>`. Should I add this to your shell profile so it persists across sessions?"

3. If yes — detect the shell profile file:
   - Check if `~/.zshrc` exists → use it
   - Else check if `~/.bashrc` exists → use it
   - Else check if `~/.bash_profile` exists → use it
   - If none found: report the path and tell the developer to add it manually

4. Show the exact line that will be added:
   ```
   export WORKFLOW_HOME="<chosen-path>"
   ```
   Ask: "Add this line to `<profile-file>`?"

5. If confirmed: append the line to the profile file. Report: "Added to `<profile-file>`. Run `source <profile-file>` or open a new terminal to activate it."

6. Create the directory: `mkdir -p <chosen-path>`

---

## Step 3 — Verify Write Access

Confirm the developer can write to `WORKFLOW_HOME`:

Run a quick write test:
```
touch $WORKFLOW_HOME/.workflow-setup-check && rm $WORKFLOW_HOME/.workflow-setup-check
```

- If it succeeds: "Write access confirmed. ✓"
- If it fails: "Cannot write to `$WORKFLOW_HOME`. Check directory permissions." — stop and report the error.

---

## Step 4 — Summary Report

Print a summary of the environment state:

```
Workflow Environment
--------------------
WORKFLOW_HOME  <value>   ✓ set and writable
```

If everything is configured correctly:
> "Environment is ready. Start a new feature with `/workflow:explore <feature-name>`."

If anything was just configured and requires a shell reload:
> "Environment configured. Run `source <profile-file>` (or open a new terminal), then start with `/workflow:explore <feature-name>`."

---

## Rules

- **Never write to a profile file without showing the exact line and getting confirmation.**
- **Never overwrite an existing `WORKFLOW_HOME` setting** — if the variable is already in the profile file, report it and ask if they want to change it.
- **Detect the shell profile; do not assume.** Check in order: `.zshrc`, `.bashrc`, `.bash_profile`. Report which one was found.
- **Stop on write failure.** If `WORKFLOW_HOME` is not writable, report the error clearly and stop — do not proceed to a workflow phase.
