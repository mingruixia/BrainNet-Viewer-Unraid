# BrainNet Viewer LAN for Unraid

[中文安装说明](INSTALL.zh-CN.md)

BrainNet Viewer is available in [Unraid Community Applications](https://ca.unraid.net/apps?q=brainnet).
It displays brain surfaces, networks, volumetric maps and atlas regions in a browser.

## Install and open

1. In Unraid **Apps**, search for **brainnet-viewer-lan** or **BrainNet Viewer**.
2. Choose **Install**. Keep port **1031**, or select another free host port.
3. Choose **Apply**, then open **WebUI** at `http://UNRAID_IP:1031`.

The default mode needs **no certificate, password file or server-origin setting**.
Each browser session has a separate process and workspace; tabs in the same
browser profile share a session. The first session can take up to 90 seconds to
initialize its graphics worker.

HTTP is unencrypted and has no password by default. Use this mode on a trusted
LAN, without a public port forward. HTTPS and password access are independent
advanced options described below.

The template uses `mingruixia/brainnet-viewer:2.0.0-unraid.1`, a LAN gateway variant
of the published 2.0.0 application. The original `2.0.0` and `latest` images are
unchanged. [Gateway source and build instructions](https://github.com/mingruixia/BrainNet-Viewer/blob/maintenance/unraid-easy-lan-20261102/docs/web/en/UNRAID_LAN.md).

For manual installation, copy `templates/brainnet-viewer-lan.xml` to
`/boot/config/plugins/dockerMan/templates-user/my-brainnet-viewer-lan.xml`,
then select it from **Docker → Add Container → Template**.

## Optional HTTPS and password

Open the advanced template settings only if needed:

| Field | Default | To enable |
| --- | --- | --- |
| HTTP or HTTPS | `http` | Select `https` |
| Password access | `off` | Select `on` |
| Public origin | empty | Exact `http(s)://host:port`, no path |
| Private server files | empty; no mount | Read-only directory for the files below |

For HTTPS, prepare `server.crt` and `server.key`, set the exact HTTPS origin,
select `https`, and mount their directory at `/run/bnv-secrets`. The certificate
must cover that IP/hostname, and client browsers must trust its issuer. Update
the WebUI shortcut to the same HTTPS address. Do not bypass certificate warnings.

For password access, prepare `access_password.txt` in that directory and select
`on`. It contains one line of 16–256 UTF-8 bytes; generate a random team password.
There is no default password. Enable HTTPS as well when encryption is required.

Only the chosen options require their corresponding files. Keep the directory
private and outside SMB/NFS shares. The files must be readable by UID **10001**.
For a new dedicated directory, after placing only the required files:

```sh
bnvSecrets=/mnt/user/appdata/brainnet-viewer-lan/secrets
chown 10001:10001 "$bnvSecrets"
chmod 700 "$bnvSecrets"
for bnvFile in server.crt server.key access_password.txt; do
  if [ -f "$bnvSecrets/$bnvFile" ]; then
    chown 10001:10001 "$bnvSecrets/$bnvFile"
    chmod 400 "$bnvSecrets/$bnvFile"
  fi
done
```

Use those commands only on that dedicated directory. The template mounts it
read-only, does not expose the password in its form, and does not use PUID/PGID.
For a DNS name in HTTP mode, set Public origin explicitly. Automatic address
discovery accepts literal private/loopback IP addresses, not arbitrary DNS names.

## Capacity and GPU

Defaults are four browser sessions, a 30-minute idle timeout and two scientific
threads per worker. All sessions share one CPU, 16 GiB RAM, 512 processes and
2 GiB temporary storage. Memory limits are ceilings, not reservations. Adjust
`--cpus`, `--memory` and `/tmp` in **Extra Parameters** for your hardware; reduce
sessions on smaller servers. Thread count is not a CPU quota.

The template requests no GPU device. Rendering backend `auto` uses an accessible
GPU after a rendering probe, otherwise CPU. For NVIDIA acceleration, configure
the compatible Unraid NVIDIA driver/container runtime, add `--gpus all` to
Extra Parameters and add `NVIDIA_DRIVER_CAPABILITIES=graphics,utility,compute`.
If Docker cannot allocate a GPU, remove `--gpus all`; application fallback cannot
repair a container rejected before it starts. Do not enable privileged mode or
mount the Docker socket.

## Save, end and update

**Add Files** uploads into the current session. **Save Scene** downloads a portable
ZIP. **Export Figure** downloads the image and JSON separately. Save before
ending a session, expiry, restart or update; uploads are temporary.

Use `http://UNRAID_IP:1031/_bnv/session` (HTTPS when enabled) to end your session.
A new session starts only when you return to the application or choose Start new
session. Other browser sessions continue independently.

Existing HTTPS installations keep their saved Unraid settings. Updating the
image alone does not remove old entry-point overrides, Post Arguments or required
secret mounts. To adopt the new defaults, save Scenes, stop the old container,
then use a fresh Apps template with an available name/port; retain the old
configuration until the new installation works. Keep private files for rollback.

## Validation and support

The gateway variant has targeted Docker/Linux HTTP-session, CPU surface/export,
browser entry and optional HTTPS/password checks. These do not constitute native
Unraid installation/form, NVIDIA passthrough or native ARM64 rendering validation.
The original 2.0.0 application and resources retain their published identity.

- [Application source and issues](https://github.com/mingruixia/BrainNet-Viewer/issues)
- [Published images](https://hub.docker.com/r/mingruixia/brainnet-viewer)

Template license: GPL-3.0-or-later. Application resources retain their own notices.
