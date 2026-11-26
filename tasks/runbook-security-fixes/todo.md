# TODO: the self-host runbook after the security fixes

Status: Active · Branch: `docs/runbook-security-fixes` · Plan: [plan.md](plan.md)

Skills: `incremental-implementation`, `code-review-and-quality`.

- [ ] `self-host-runbook.md`: a trusted-network caution, the ports table (no 5432; 8082 on this machine only), and the admin console URL and SSH tunnel in the secret-rotation and SMTP steps.
- [ ] `self-hosting.md`: back up PostgreSQL with `pg_dumpall` inside the container.
- [ ] `npm run check`, `npm run build`, and a look at the changed sections.
- [ ] PR, to merge after the bundle PR.
