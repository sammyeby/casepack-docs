# TODO: the self-host runbook after the security fixes

Status: Active · Branch: `docs/runbook-security-fixes` · PR: [#12](https://github.com/sammyeby/casepack-docs/pull/12) · Plan: [plan.md](plan.md)

Skills: `incremental-implementation`, `code-review-and-quality`.

- [x] `self-host-runbook.md`: a trusted-network caution, the ports table (no 5432; 8082 on this machine only), and the admin console URL and SSH tunnel in the secret-rotation and SMTP steps.
- [x] `self-hosting.md`: back up PostgreSQL with `pg_dumpall` inside the container.
- [x] `npm run check`, `npm run build`, and a look at the changed sections.
  - `astro check`: 0 errors, 0 warnings, 0 hints. `astro build` completes.
  - The anchors the new links use exist in the built HTML: `#before-you-start`, `#not-yet-validated`, `#6-sign-in-and-invite-users`.
  - The backup command was run as written (through `exec` and `sh -c`) against `ghcr.io/sammyeby/postgresql:17.12-alpine` with the bundle's `init-keycloak-db.sh`. The dump had both roles and both databases, mode `0600`. Without `PGPASSWORD`, this image refuses the local socket.
  - The admin console URL and ports match QA run 4 on bundle `6dc40c9`: `localhost:8082/admin/` redirects to the console, `8081/admin/` gives 404.
- [x] PR [#12](https://github.com/sammyeby/casepack-docs/pull/12), to merge after [casepack#33](https://github.com/sammyeby/casepack/pull/33).
