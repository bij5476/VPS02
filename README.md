# VPS02 — Windows test machine over Tailscale

This repository contains a manually started GitHub Actions workflow that creates a fresh Windows machine, joins it to your Tailnet, enables Remote Desktop, and removes it when the job ends.

## Important limit

GitHub-hosted Windows jobs have a maximum job duration of 360 minutes. The workflow therefore offers sessions up to **350 minutes** to leave time for setup and automatic cleanup. It cannot provide an 8-hour session. For eight hours or more, use a paid VPS or a self-hosted runner.

## One-time setup

### 1. Add repository secrets

In **Settings → Secrets and variables → Actions**, create these repository secrets:

| Secret | Purpose |
| --- | --- |
| `TS_OAUTH_CLIENT_ID` | Tailscale OAuth client ID |
| `TS_OAUTH_SECRET` | Tailscale OAuth client secret |
| `RDP_USERNAME` | New local Windows account name |
| `RDP_PASSWORD` | Strong password for that account |

Create the OAuth client in the Tailscale admin console with permission to create auth keys and with the `tag:ci` tag allowed. Keep the password and all Tailscale credentials in GitHub Secrets; never commit them to this repository.

### 2. Restrict Tailnet access

In your Tailscale ACL/grants, allow only your own devices to reach `tag:ci` on TCP port `3389`. This makes the RDP service reachable only over your Tailnet.

## Starting a machine

1. Open the repository's **Actions** tab.
2. Choose **Ephemeral Windows over Tailscale**.
3. Select **Run workflow** and choose a duration.
4. Wait for the job to complete its setup steps.
5. In the Tailscale app or admin console, find the new device named `vps02-<run-id>`.
6. Use Windows Remote Desktop to connect to that device's Tailscale IP or MagicDNS name with the `RDP_USERNAME` and `RDP_PASSWORD` you stored.

Only one machine is allowed at a time. Starting another run cancels the previous one. When the job finishes, GitHub destroys the virtual machine and the Tailscale node is removed automatically.

## Security notes

- The workflow uses manual dispatch only; it does not run for pushes or pull requests.
- It does not print the RDP password or the Tailscale address in workflow logs.
- This repository is public. Do not put credentials in files, workflow inputs, issues, or commits.
