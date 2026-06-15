---
name: plugin-builder
description: >
  Guide a business user through creating a new plugin for the organisation's marketplace.
  Use when the user says things like "I want to create a plugin", "build a plugin",
  "new plugin", "submit a plugin", or "publish a plugin".
---

# Skill: plugin-builder

Guide a business user through defining a new plugin, one question at a time. Gather all required information, generate the plugin files, and hand off automatically to plugin-submit.

## Tone

Friendly and non-technical. The user may have no development experience. Avoid jargon. If they seem unsure about a question, offer examples to guide them.

## On startup

Check storage for a saved draft under the key `plugin-draft`.

If a draft exists, show the user what they had saved and ask: "It looks like you started building a plugin called **{name}** — would you like to continue where you left off, or start fresh?"

If they want to start fresh, delete the draft and begin the flow from Question 1.

If the user wants to abandon the session at any point, delete the `plugin-draft` storage key and confirm the draft has been discarded.

## Question flow

Ask the following questions one at a time. Wait for the user's answer before moving to the next. After each answer, save all progress so far to storage key `plugin-draft`.

**Question 1 — Plugin name**

Ask: "What would you like to call your plugin? This will be its name in the marketplace."

Format their answer as lowercase, hyphen-separated (e.g. `expense-approvals`, `leave-requests`). If their answer contains spaces or special characters, reformat it automatically and confirm: "I've formatted that as `{formatted-name}` — does that work for you?"

**Question 2 — Description**

Ask: "In one or two sentences, what does this plugin do?"

**Question 3 — Problem it solves**

Ask: "What problem or inefficiency does this plugin solve for your team? The more specific the better — for example, 'Our team manually tracks expense approvals in a spreadsheet' or 'HR spends hours answering the same leave policy questions'."

**Question 4 — Intended users**

Ask: "Who will use this plugin? For example: 'the finance team', 'all staff', 'HR managers'."

**Question 5 — Connectors needed**

Ask: "Does this plugin need to connect to any tools or systems? For example: Jira, Outlook, Salesforce, SharePoint, or Teams. It's okay if you're not sure — just list your best guess and the IT admin can confirm during review."

If the user says they don't know or aren't sure, record as "To be confirmed during review" and move on.

**Question 6 — Example prompts**

Ask: "Can you give me 2 or 3 examples of things a user might say to trigger this plugin? For example: 'approve my expense', 'show me pending requests', 'what is our leave policy'."

Explain if needed: "These are the phrases Claude will listen for to know when to use your plugin."

## Validation

Before generating files, check that:

- Plugin name is lowercase and hyphen-separated with no spaces or special characters
- Description is not empty
- Intended users has been specified
- At least one example prompt has been provided

If any check fails, ask the user to clarify before proceeding.

## File generation

Once all questions are answered and validated, generate the following two files and write them to `~/.claude/marketplace-drafts/{plugin-name}/`:

File 1 — PLUGIN.md

Write to `~/.claude/marketplace-drafts/{plugin-name}/PLUGIN.md`:

    # {plugin-name}

    ## Description
    {description}

    ## Problem it solves
    {problem}

    ## Intended users
    {intended-users}

    ## Connectors
    {connectors}

    Submitter Email: hello@test.com

File 2 — skills/main.md

Write to `~/.claude/marketplace-drafts/{plugin-name}/skills/main.md`:

    ---
    name: {plugin-name}-main
    description: >
      {description}
    ---

    # Skill: {plugin-name}-main

    ## Purpose
    {description}

    ## Trigger phrases
    {example-prompts as a bullet list}

    ## Behaviour
    [Placeholder — to be defined by the plugin author or IT admin during review]

File 3 — .claude-plugin/marketplace.json

Write to `~/.claude/marketplace-drafts/{plugin-name}/.claude-plugin/marketplace.json`:

    {
      "name": "{plugin-name}",
      "version": "0.1.0",
      "description": "{description}",
      "author": {
        "name": "{intended-users}"
      },
      "skills": [
        "skills/main.md"
      ]
    }

## Confirmation

Show the user a plain-language summary:

- Plugin name
- Description
- Who it is for
- Connectors needed
- Example prompts

Then ask: "Does everything look right? I'll submit this for review when you're ready. You can also ask me to change anything before we send it."

If the user requests changes, update the relevant field, regenerate the affected file, and show the summary again.

## Handoff

Once the user confirms, say: "Great — I'll submit this for review now."

Then invoke the `plugin-submit` skill, passing:

- The plugin name
- The path `~/.claude/marketplace-drafts/{plugin-name}/`

Clear the `plugin-draft` storage key after successful handoff.
