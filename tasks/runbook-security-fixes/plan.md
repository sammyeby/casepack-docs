# Plan: the self-host runbook after the security fixes

Checklist: [todo.md](todo.md)

## What and why

The security review of 2026-11-26 changed what the self-host bundle exposes. The bundle PR (`casepack` branch `fix/security-review-bundle`) makes these changes:

- **B-3:** PostgreSQL is no longer published on the host.
- **B-5:** sign-in on `8081` goes through `keycloak-edge`, which passes only the `casepack` realm. The Keycloak admin console moves to `127.0.0.1:8082`.
- **B-2:** the realm requires HTTPS for public client addresses (`sslRequired: external`). The bundle is plain HTTP, so it belongs on a trusted network only.
- **B-4:** the database now lives on its volume.

Without these docs changes, the public runbook would be wrong: it lists port 5432, and it sends operators to `http://localhost:8081/admin`, which now returns 404.

Source: `casepack-api` `tasks/fix-self-host-install/todo.md`, item 12 (casepack-docs line), on branch `chore/fix-self-host-install`.

## Approach

`self-host-runbook.md`:
- A caution under "Before You Start": plain HTTP, so trusted network only until TLS is supported.
- The ports table: drop the 5432 row, add `8082` (admin console, this machine only). The 8081 row says it serves sign-in only.
- The admin console URL, `http://localhost:8082/admin/`, through `ssh -L 8082:localhost:8082 <server>` from another machine. It goes in the secret-rotation caution and the SMTP steps.

`self-hosting.md`, Backup, Restore, and Redeploy:
- How to dump PostgreSQL now that it isn't published: `pg_dumpall` inside the container. It reads the password from the container's own environment, and one dump holds both databases and their roles.
- Restore stays "not yet validated".

Greenfield, so there are no notes for installs made before these fixes (owner, 2026-11-26).

## Risks

- **Documenting the bundle before its PR merges.** Mitigation: merge this after the bundle PR, and take every command from the bundle branch and the QA run.
- **A backup command that doesn't work with the hardened PostgreSQL image.** It asks for a password even on the local socket. Mitigation: the exact command was run against `ghcr.io/sammyeby/postgresql:17.12-alpine` with the bundle's init script. Its output had both databases and both roles.

## Verification

- `npm run check` and `npm run build`.
- The changed sections render, and their anchors still resolve.
- The URLs and ports match the bundle branch and QA run 4.
