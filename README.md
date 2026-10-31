# BrainNet Viewer LAN for Unraid

[中文安装说明](INSTALL.zh-CN.md)

BrainNet Viewer visualizes brain surfaces, networks, volumetric maps and atlas
regions. This template deploys the published `mingruixia/brainnet-viewer:2.0.0`
image in LAN server mode. It does not rebuild or modify the application.

Each signed-in browser session has a separate process and workspace. Tabs in
the same browser profile share a session; use another profile or private window
for an independent workspace. The team password is shared, not a named-user
account system. Use this service on a trusted LAN.

## Before installation

1. Enable Docker on your Unraid server. Choose a fixed server IP and a free TCP
   port, normally **1031**. Restrict access to your trusted LAN.
2. Obtain a TLS certificate containing that IP in its Subject Alternative Name,
   and its matching private key. Client computers must trust its issuing CA.
   Do not bypass browser certificate warnings.
3. In a private directory, prepare `server.crt`, `server.key` and
   `access_password.txt`. The password must contain 16–256 UTF-8 bytes on one
   line. Generate it locally and share it through a private channel.
4. Set the files readable by container UID **10001**. For a new dedicated
   directory on the Unraid host, an administrator can use the commands below
   after placing the three files. Do not run them on a shared or unrelated folder.

```sh
bnvSecrets=/mnt/user/appdata/brainnet-viewer-lan/secrets
chown 10001:10001 "$bnvSecrets" "$bnvSecrets/server.crt" "$bnvSecrets/server.key" "$bnvSecrets/access_password.txt"
chmod 700 "$bnvSecrets"
chmod 400 "$bnvSecrets/server.crt" "$bnvSecrets/server.key" "$bnvSecrets/access_password.txt"
```

Keep the directory out of SMB/NFS shares and public backups. The template mounts
it read-only. It does not ask for passwords or private keys in the template form,
and it does not support PUID/PGID environment variables.

## Install

Until the Community Applications listing has been approved, use the XML in
`templates/brainnet-viewer-lan.xml` as a local Docker template. Copy it to
`/boot/config/plugins/dockerMan/templates-user/my-brainnet-viewer-lan.xml` on
your Unraid server, then select it from **Docker → Add Container → Template**.
After approval, search for **brainnet-viewer-lan** in **Apps** instead.

Configure:

| Field | Setting |
| --- | --- |
| Network | `bridge` |
| HTTPS port | `1031`, or another free host port |
| Public HTTPS origin | For example `https://192.168.1.10:1031`, with no path |
| Private server files | The directory containing the three prepared files |
| Rendering backend | `auto`; CPU is used when a GPU is unavailable |

The origin must match the browser address, certificate and host port. Container
port 1031 stays unchanged. If you use a DNS name instead of the server IP, open
that exact HTTPS origin directly and adjust the WebUI shortcut to match it.
Do not enable Tailscale integration or insert a TLS-terminating reverse proxy
without a separate origin/session deployment assessment.

Select **Apply**, then open **WebUI** and sign in with the team password.
Graphics initialization for a new session may take up to 90 seconds. The
container health check reports gateway readiness, not a completed rendering test.

## Capacity and optional GPU

Defaults are four sessions, a 30-minute idle timeout and two scientific-library
threads per worker. The container has a shared limit of one CPU, 16 GiB RAM,
512 processes and 2 GiB of temporary storage. Memory limits are ceilings, not
reservations. Adjust `--cpus`, `--memory` and the `/tmp` size in **Extra Parameters**
for your hardware and datasets; do not exceed the server's available resources.
Reduce maximum sessions on smaller servers. Thread count is not a CPU quota.

No GPU device is requested by default, so the template can start on CPU-only
servers. For NVIDIA acceleration, first configure the compatible Unraid NVIDIA
driver/container runtime. Then add `--gpus all` to **Extra Parameters** and add
the environment variable `NVIDIA_DRIVER_CAPABILITIES=graphics,utility,compute`.
Keep the rendering backend `auto`. The worker checks actual rendering capability;
visible hardware alone does not establish hardware rendering. If Docker cannot
allocate a GPU, remove `--gpus all` to restore CPU startup. Application fallback
cannot recover a container rejected by Docker before it starts.

Never enable privileged mode or mount the Docker socket for this application.

## Save work and update

**Add Files** uploads client files into the current session. **Save Scene**
downloads a portable ZIP; **Export Figure** downloads the image and its JSON
sidecar separately. Save work before signing out, updating or stopping the
container. Temporary uploads are not a persistent data archive.

Open `https://SERVER_IP:1031/_bnv/session` and choose **Sign out and end session**
for immediate session cleanup. Idle sessions also expire automatically.

The template pins 2.0.0 rather than following `latest`. Before changing the image
tag, save active Scenes and read the new release notes. A restart ends unsaved
sessions. Retain the previous image tag for rollback.

## Validation and support

This template is being prepared for Community Applications submission. Native
Unraid installation, its form rendering and NVIDIA passthrough have not yet been
validated. A Docker check on 2026-10-31 passed HTTPS readiness, login-page access,
two distinct secure session cookies, the HTTPS health-check override, non-root
execution, read-only secret mounts and resource limits. It did not exercise
worker rendering or Unraid's template installer and is not a claim of
Unraid certification. Native ARM64 container graphics validation also remains
incomplete.

- [Application source and issue tracker](https://github.com/mingruixia/BrainNet-Viewer)
- [LAN server documentation](https://github.com/mingruixia/BrainNet-Viewer/blob/master/docs/web/en/SERVER_DEPLOYMENT.md)
- [Published images](https://hub.docker.com/r/mingruixia/brainnet-viewer)
- [Community Applications submission requirements](https://ca.unraid.net/submit/help)

Template license: GPL-3.0-or-later. Application resources retain their own notices.
