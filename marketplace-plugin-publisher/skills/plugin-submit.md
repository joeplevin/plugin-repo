---
name: plugin-submit
description: >
  Submit a completed plugin to GitHub for IT admin review.
  Invoked automatically by plugin-builder on user confirmation.
  Do not invoke this skill directly unless plugin files already exist at the expected path.
---

# Skill: plugin-submit

Take the generated plugin files from plugin-builder and submit them to GitHub as a pull request. Notify the IT admin via GitHub Actions. Confirm back to the business user that their submission is under review.

## On invoke

Expect the following to be passed from plugin-builder:

- Plugin name (e.g. `expense-approvals`)
- Plugin files path (e.g. `~/.claude/marketplace-drafts/expense-approvals/`)

If either is missing, tell the user something went wrong and suggest they go back and run plugin-builder again.

## Steps

### Step 1 — Verify files

Check that the following files exist at the provided path:

- `PLUGIN.md`
- `skills/main.md`

If either file is missing, stop and tell the user: "I couldn't find the plugin files. Please go back and run plugin-builder again."

### Step 2 — Prepare the GitHub branch

Navigate to the local copy of the GitHub repo at `C:\dev\plugin-repo`, then create and check out a new branch. Create the branch name from the plugin name and a timestamp to ensure uniqueness:

    cd C:\dev\plugin-repo
    git checkout -b plugin/{plugin-name}-{timestamp}

### Step 3 — Copy plugin files

Copy the generated plugin files into the repo under the `plugins/` directory:

    plugins/
    └── {plugin-name}/
        ├── .claude-plugin/
        │   └── marketplace.json
        ├── PLUGIN.md
        └── skills/
            └── main.md

### Step 4 — Commit and push

Stage and commit the files:

    git add plugins/{plugin-name}/
    git commit -m "feat: add {plugin-name} plugin submission"
    git push origin plugin/{plugin-name}-{timestamp}

### Step 5 — Open a pull request

Use the GitHub CLI to open a PR against main. Structure the PR body as follows so GitHub Actions can parse it correctly:

    Title: Plugin Submission: {plugin-name}

    Body:
    ## Plugin Submission

    **Plugin name:** {plugin-name}
    **Description:** {description}
    **Intended users:** {intended-users}
    **Connectors needed:** {connectors}

    ## Example prompts
    {example-prompts as a bullet list}

    ---
    Submitter Email: hello@test.com

Use the GitHub CLI to create the PR:

    gh pr create \
      --title "Plugin Submission: {plugin-name}" \
      --body "{structured body above}" \
      --base main \
      --head plugin/{plugin-name}-{timestamp}

### Step 6 — Confirm to the user

Once the PR is open, tell the user:

"Your plugin **{plugin-name}** has been submitted for review. The IT admin has been notified by email and will review it shortly. You'll receive an email once a decision has been made."

Do not show the user any Git output, branch names, or technical details. Keep it simple and reassuring.

## Error handling

If any step fails:

- Do not leave the user without feedback
- Tell them in plain language what went wrong
- Suggest they contact their IT admin if the problem persists
- Do not expose raw error messages or stack traces
