---
title: Self-Host Runbook
description: Install, enroll, run and upgrade a CasePack self-hosted deployment with Docker Compose or Podman.
---

This runbook takes a CasePack self-hosted deployment from a fresh server to signed-in users, and covers renewal and upgrades. It is for the design-partner preview; read [Self-Hosting](/self-hosting/) first for what the preview supports.

:::note[What this runbook covers]
Every step here runs in our self-host QA against the published bundle: install, enrollment, first sign-in, inviting users, credential renewal, and upgrading between releases. [Not yet validated](#not-yet-validated) lists what isn't covered yet. For those, contact us and we'll do it with you.
:::

## Before You Start

You need:

- A Linux server (or a Mac for evaluation) with **Docker Engine 24+ and Docker Compose v2**, or **Podman with `podman compose`**.
- `git` and `openssl`.
- At least 15 GB of free disk for the images and your data.
- Outbound HTTPS from the server to `https://licensing.samuelumeh.com`.
- An **SMTP server**. CasePack sends user invitations through it, and won't invite users without it.
- A **CasePack Self-Hosted entitlement**, and an account on the [licensing portal](https://licensing.samuelumeh.com/portal).

:::caution[Trusted network only]
The bundle serves plain HTTP. Until TLS is supported, run it only on a network you trust, such as your office LAN or a VPN. Keycloak refuses HTTP sign-in from public client addresses.
:::

The bundle uses these ports on the server:

| Port | Service | Who needs to reach it |
|---|---|---|
| 3000 | Web app | Your users' browsers |
| 8080 | API | Your users' browsers |
| 8081 | Keycloak sign-in (only the `casepack` realm) | Your users' browsers |
| 8333 | Evidence storage (S3) | Your users' browsers, for evidence uploads and downloads |
| 8082 | Keycloak admin console | Only you. It listens on the server itself (`127.0.0.1`). |

PostgreSQL isn't published on the server. To reach the admin console from another machine, open an SSH tunnel and then go to `http://localhost:8082/admin/`:

```bash
ssh -L 8082:localhost:8082 <your-server>
```

## 1. Get the Bundle

```bash
git clone https://github.com/sammyeby/casepack.git
cd casepack
```

## 2. Configure

Create your configuration from the template and make it readable only by you:

```bash
cp .env.example .env
chmod 600 .env
```

Edit `.env` and fill in these settings. Generate each secret separately. Never reuse one, and never use an example value.

| Setting | Value |
|---|---|
| `DB_PASS`, `KC_DB_PASS`, `KC_ADMIN_PASS` | One `openssl rand -hex 32` each |
| `S3_ACCESS_KEY` | `openssl rand -hex 16` |
| `S3_SECRET_KEY` | `openssl rand -hex 32` |
| `CASEPACK_KEYCLOAK_SA_CLIENT_SECRET` | `openssl rand -hex 32` |
| `CASEPACK_SELF_HOST_BOOTSTRAP_ADMIN_EMAIL` | The first CasePack administrator's email address |
| `CASEPACK_SELF_HOST_BOOTSTRAP_ADMIN_INITIAL_PASSWORD` | A temporary password. You choose a new one at first sign-in. |
| `KC_SMTP_HOST`, `KC_SMTP_PORT`, `KC_SMTP_FROM`, `KC_SMTP_USER`, `KC_SMTP_PASSWORD` | Your mail server. Adjust `KC_SMTP_STARTTLS`, `KC_SMTP_SSL` and `KC_SMTP_AUTH` if it needs to. |

Leave these as shipped:

- `CASEPACK_LICENSE_PUBLIC_KEY`: the public key CasePack uses to verify its licence credentials.
- `LICENSING_API_URL` and `LICENSING_PORTAL_URL`.
- `CASEPACK_API_VERSION` and `CASEPACK_SPA_VERSION`: the versions this bundle was tested with.

:::caution
Keycloak reads the SMTP settings and the service-account secret only on its **first** start. To change SMTP later, see [Sign In and Invite Users](#6-sign-in-and-invite-users). To rotate the service-account secret, open the Keycloak admin console (`http://localhost:8082/admin/`, realm **casepack**; see [the ports](#before-you-start) for the tunnel) at **Clients** → **casepack-user-manager** → **Credentials** and click **Regenerate**. Copy the new secret into `CASEPACK_KEYCLOAK_SA_CLIENT_SECRET` in `.env`, then run `docker compose up -d`.
:::

## 3. Start CasePack

```bash
docker compose up -d
```

With Podman, include the Podman override in every Compose command:

```bash
podman compose -f docker-compose.yml -f docker-compose.podman.yml up -d
```

The API is ready when its health check answers `UP`:

```bash
curl -s http://localhost:8080/actuator/health
```

Until it's enrolled, CasePack runs in **setup mode**: it creates no customer data, and nobody can sign in yet. The API log says so:

```bash
docker compose logs api | grep "entered SETUP"
```

## 4. Create an Enrollment Code

1. Sign in to the [licensing portal](https://licensing.samuelumeh.com/portal).
2. Under **CasePack Self-Hosted entitlement**, choose the deployment to enroll: **Primary**, or **Recovery/test** for a second deployment.
3. Click **Create enrollment code**.

The code works once and expires after **10 minutes**, so have the next step ready. Save it to a file without showing it on screen:

```bash
umask 077
read -rsp 'Enrollment code: ' code && printf '%s' "$code" > ~/casepack-enrollment-code; unset code; echo
```

Don't put the code in `.env`, in a command line or in a ticket.

## 5. Enroll

From the `casepack` directory:

```bash
./activate.sh --code-file ~/casepack-enrollment-code
rm ~/casepack-enrollment-code
```

`activate.sh` uses Docker Compose if it's installed, and Podman Compose otherwise. A successful enrollment prints:

```text
Enrollment complete.
Installation: inst_4d551e2545ea4a3bbb029df00171f749
Credential sequence: 1
Credential expires: 2026-12-26T08:13:31Z
Enrollment complete. The durable identity is in the licensing-state volume.
Next: docker compose up -d
If CasePack is already running, it applies the enrollment within a minute.
```

Your installation ID will differ. If CasePack is running, there's nothing more to do: within a minute the API applies the credential and creates the first administrator and workspace, without a restart. To watch for it:

```bash
docker compose logs -f api | grep -E "Applied a new Connected credential|Self-host bootstrap complete"
```

If the code expired before you used it, create a new one. If `activate.sh` reports any other error, send us its output and the result of [the status command](#7-check-status-and-renewal). It never contains the code.

## 6. Sign In and Invite Users

Open `http://localhost:3000` (or your server's address) and sign in with the administrator email and initial password from `.env`. Keycloak asks you to choose a new password.

To add people, use **Create User** on the users page. See [Users & Roles](/users-roles/) for the roles. Each person gets an invitation email to set their password and verify their address.

If SMTP isn't configured, CasePack refuses the invitation with **"Email isn't set up for this CasePack installation…"** and creates nothing. To set up email after the first start, use the Keycloak admin console:

1. Open `http://localhost:8082/admin/` on the server, or through the [SSH tunnel](#before-you-start), and sign in as `admin` with `KC_ADMIN_PASS`.
2. Switch to the **casepack** realm.
3. Open **Realm settings**, then the **Email** tab, and enter your mail server.

## 7. Check Status and Renewal

```bash
docker compose run --rm --no-deps api licensectl status
```

```text
Status: ENROLLED
Installation: inst_4d551e2545ea4a3bbb029df00171f749
Credential sequence: 2
Credential present: true
```

CasePack renews its credential automatically about once a day. Each credential is valid for up to 30 days, so the server can be offline from the licensing service for up to 30 days without losing write access. A renewal raises the **credential sequence** by one. The licensing portal shows each deployment's sequence and when it was last seen.

To renew now:

```bash
docker compose run --rm --no-deps api licensectl refresh
```

The running API applies the new credential within a minute, with no restart.

## 8. Upgrade

Your `.env` pins the versions you installed, so `git pull` alone doesn't upgrade CasePack. When we announce a release:

1. Back up first. See [Backup, Restore, and Redeploy](/self-hosting/#backup-restore-and-redeploy).
2. Get the new bundle files:

   ```bash
   git pull
   ```

3. Check whether the new `.env.example` adds settings your `.env` lacks:

   ```bash
   diff <(grep -oE '^[A-Z0-9_]+=' .env.example | sort) <(grep -oE '^[A-Z0-9_]+=' .env | sort)
   ```

   Add any setting listed with `<`, and read its comment in `.env.example`.
4. Set `CASEPACK_API_VERSION` and `CASEPACK_SPA_VERSION` in `.env` to the announced versions.
5. Pull the images and restart:

   ```bash
   docker compose pull
   docker compose up -d
   ```

An upgrade keeps the installation, its credential and your data: CasePack doesn't enroll again. Afterwards, `licensectl status` shows the same installation ID.

## Not Yet Validated

Not yet covered by our QA, so contact us before you rely on them:

- Serving CasePack on your own domain behind TLS. This involves `KEYCLOAK_HOSTNAME`, `API_URL`, `OIDC_AUTHORITY`, `CORS_ORIGINS`, `CASEPACK_KEYCLOAK_SPA_REDIRECT_URI` and `S3_PUBLIC_ENDPOINT`.
- Restoring from backup, and reconciling the licensing state afterwards.
- Rolling back to an earlier version.
- Replacing a deployment's identity, for example after losing the `licensing-state` volume.
- A licensing outage longer than the 30-day credential.
- Kubernetes/Helm, or more than one API replica.

## Related

- [Self-Hosting](/self-hosting/): what the preview supports, privacy, recovery states and backups.
- [Licensing & Access States](/licensing-access/)
- [Users & Roles](/users-roles/)
