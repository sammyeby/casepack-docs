# TODO: the public self-host runbook

Status: Active · Branch: `docs/self-host-runbook` · PR: [#11](https://github.com/sammyeby/casepack-docs/pull/11) · Plan: [plan.md](plan.md)

Skills: `incremental-implementation`, `frontend-ui-engineering` (the sidebar entry), `code-review-and-quality`.

- [x] The runbook page, from the QA runs' verified steps.
- [x] Sidebar entry.
- [x] `self-hosting.md`: replace the publication gate with a pointer; correct the prerequisites.
- [x] `npm run check`, `npm run build`, and a look at the rendered page.
  - `astro check`: 0 errors, 0 warnings, 0 hints. `astro build` completes; `/self-host-runbook/` is built.
  - Every in-page anchor the new links use exists in the built HTML (`#2-configure`, `#6-sign-in-and-invite-users`, `#7-check-status-and-renewal`, `#installs-created-before-2026-11-26`, `#not-yet-validated`, Self-Hosting's `#backup-restore-and-redeploy`).
  - A full-page screenshot of `astro preview` (Chromium, 1280 px): the sidebar entry under Reference, 3 asides, 2 tables and 16 code blocks render.
- [x] PR [#11](https://github.com/sammyeby/casepack-docs/pull/11), to merge after casepack#29.
