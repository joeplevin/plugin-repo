---
name: plugin-notify
description: >
  Notify the business user of the outcome of their plugin submission.
  Triggered automatically by GitHub Actions webhook when a pull request
  is merged or closed. Do not invoke this skill manually.
---

# Skill: plugin-notify

Handle the outcome of a plugin submission PR. Triggered by GitHub Actions
when a PR is merged (approved) or closed without merging (rejected). Send
the appropriate email to the business user.

## On invoke

Expect the following from the GitHub Actions webhook payload:
- `action` — either `merged` or `rejected`
- `plugin-name` — name of the plugin (parsed from PR title)
- `pr-url` — URL of the pull request
- `pr-body` — full body of the pull request
- `rejection-reason` — the IT admin's closing comment (only present if rejected)

If `action` is missing or unrecognised, log the error and stop. Do not
send any email.

## Parse submitter email

Extract the submitter email from `pr-body` by finding the line:

    Submitter Email: {email}

Use this as the `to` address for the notification email. If no email is
found, log the error and stop. Do not send a fallback email to an unknown
address.

## Merged path

If `action` is `merged`:

Send an email to the submitter with:

    Subject: Your plugin has been approved: {plugin-name}

    Body:
    Great news! Your plugin has been approved and is now live
    in the marketplace.

    Plugin: {plugin-name}
    View it here: {pr-url}

    You can start using it straight away in Cowork.

## Rejected path

If `action` is `rejected`:

Extract the rejection reason from `rejection-reason`. If it is empty or
missing, use the fallback: "No reason was provided by the reviewer."

Send an email to the submitter with:

    Subject: Plugin submission update: {plugin-name}

    Body:
    Unfortunately your plugin submission was not approved.

    Plugin: {plugin-name}
    Reviewer comments: {rejection-reason}

    You can view the full review here: {pr-url}

    If you would like to make changes and resubmit, run the
    plugin-builder skill again and your previous answers will
    be saved as a starting point.

## After sending

Log the outcome (merged or rejected) and the plugin name for audit
purposes. Do not log the submitter email.
