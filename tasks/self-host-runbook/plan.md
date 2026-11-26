# Plan: the public self-host runbook

## What and why

A design partner installing CasePack self-host needs step-by-step instructions. Today the [Self-Hosting](../../src/content/docs/self-hosting.md) page explains the model but withholds the commands and promises a "pinned operator runbook" during onboarding, which doesn't exist.

The owner decided on 2026-11-26 to publish the runbook here instead. Source: `casepack-api` `tasks/fix-self-host-install/todo.md`, item 9.

## Approach

- A new page, `self-host-runbook.md`, under Reference next to Self-Hosting. It covers only steps the self-host QA has run against the released bundle (QA runs 4 and 5, 2026-11-25/26):
  - clone, configure `.env` (including the per-install secrets from casepack#29), start;
  - create an enrollment code in the portal, `./activate.sh --code-file`;
  - sign in; SMTP and invitations; status and renewal;
  - upgrading by version pin; replacing the old shared defaults on an existing install.
- Everything not yet validated is listed as such, with "contact us": your own domain with TLS, restore, rollback, identity replacement, a licensing outage beyond the 30-day credential.
- `self-hosting.md`: the "Enrollment Publication Gate" section becomes a pointer to the runbook. The prerequisites are corrected: the issuer key ships in `.env.example`, and SMTP is required to invite users.
- The bundle README's matching "withheld" wording changes in a separate casepack PR.

## Risks

- **Describing something that isn't true of the product.** Mitigation: every command and output is taken from the QA runs and the bundle at the commit the page names.
- **Publishing before casepack#29 merges** would document variables the bundle doesn't require yet. Mitigation: merge this after #29.

## Verification

- `npm run check` and `npm run build`.
- The page renders and appears in the sidebar (`npm run dev`).
- Each command is checked against the bundle's `main` after #29 merges.
