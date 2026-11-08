# syncthing-for-tos

> TerraMaster TOS 7 App Center — **Docker application** package for
> [Syncthing](https://github.com/syncthing/syncthing).

## What this is

A four-file TOS Docker application archive (`syncthingdocker.tar.gz`) that installs
Syncthing on TOS 7 through the App Center:

- `config.ini` — TOS application metadata (`application_type: docker`, id `syncthingdocker`)
- `syncthingdocker.lang` — 23-language superset (TOS requires 14)
- `syncthingdocker.svg` — application icon (1236 bytes, 2 SVG elements)
- `docker-compose.yml` — one service, image `syncthing/syncthing:2.1.7`

## How it works

| | |
|---|---|
| Container image | `syncthing/syncthing:2.1.7` (Docker Hub, used unmodified) |
| Open | `http://<NAS-IP>:8025/` (container port 8384) |
| Runs as | non-root, `user: "1000:1000"` |
| Persistent data | `/var/syncthing` → `/Volume1/DockerAppData/syncthingdocker/data` |
| Logs | `docker logs syncthingdocker` (stdout/stderr) |

`config` holds the device ID, certificates, GUI credentials and folder database; `data` is the default synchronized folder. Copying both folders preserves the device identity and all synced files.

## Design highlights

- **Non-root.** `user: "1000:1000"`; verified on real hardware (the container reports uid 1000).
- **Fixed image tag, never `:latest`**, Docker Hub only, no `privileged`, no `network_mode: host`, and no Docker socket, no `cap_add`.
- **Health check** probing the service on container port 8384.
- **Deterministic build**: GNU tar with uid/gid 0 and zeroed timestamps, gzip without a timestamp, plus the review-standards self-check above.
- **Non-root.** Runs as `1000:1000`; verified on real hardware.
- **Two volumes, one web port.** `/var/syncthing/config` + `/var/syncthing/data`; only 8384 is published.
- **Health check** probes `/` on 8384.
- **Fixed image tag**, Docker Hub only, no privileged mode, no host network, no Docker socket.

## Runtime file manifest

Everything the application writes on the NAS lives inside its own data
directory; the container-internal `/tmp` is discarded with the container. The
table lists what is created and how it behaves. Nothing else is written.

| Path | Purpose | Created when | Growth bound | Lifecycle |
|---|---|---|---|---|
| `data/config` | device identity, certificates, GUI credentials and folder database | first start | small; grows with folders/devices | never overwritten; without it the device identity changes |
| `data/data` | default synchronized folder | by the user | grows with synced files | owned by the user; back it up |
| `/Volume1/DockerAppData/syncthingdocker/data` | Syncthing application data (everything the container writes outside /tmp) | on first start / on demand | bounded by your own data | persistent; kept on uninstall unless you ask to delete it |

## Build

```bash
scripts/build.sh                  # -> out/syncthingdocker.tar.gz + .sha256
scripts/build.sh 2.1.7-2   # packaging iteration bump
scripts/build.sh 2.1.7-1 aarch64
```

The build stages the four files, substitutes the version/platform placeholders,
removes CRLF/BOM/AppleDouble, then runs
`tools/verify.py`, a review-standards self-check (archive layout, JSON validity,
required fields, 14+ language coverage, version consistency, Docker-Hub-only
images, fixed tag, reserved/recommended host ports, non-root user, per-service
healthcheck/TZ/restart, data path below the application data root, never the
`/Volume*` wildcard, comment-free sequence blocks, `x-app-meta` position, no
literal secrets and the SVG size/element limits).

## Local verification

```bash
./TOSAppSelfTestingTool -c out/syncthingdocker.tar.gz   # official spec check
bash test/install-and-verify.sh                  # on-device E2E (needs root)
```

> Docker applications cannot be side-loaded through the App Center's manual
> install page (it accepts `.deb` only); the official self-testing tool's `-i`
> mode or `docker compose` are the only ways to install one for testing.

## Store submission checklist

1. Public GitHub repo (code + README only).
2. Release **tag `v2.1.7-1`**, assets `syncthingdocker.tar.gz` +
   `syncthingdocker.tar.gz.sha256` (Docker asset naming: `<app_id>.tar.gz`, **no
   platform suffix**).
3. Developer platform → Add Application: ID `syncthingdocker`, package type Docker,
   repository URL, architecture `x86_64`.
4. Version Management → Add Version `2.1.7-1` → automated validation → review.

## License / branding

Syncthing is distributed by its upstream authors under **MPL-2.0**. The container image is the official image from Docker Hub (`syncthing/syncthing:2.1.7`) and is used unmodified; no upstream source code is redistributed here. This packaging is an independent community submission by Moechz and is not affiliated with the upstream project or TerraMaster.

## Limitations

Only the web UI port is published. Direct LAN sync needs 22000/tcp+udp and 21027/udp; without them Syncthing falls back to discovery and relays, which is slower.
The web UI has no default password. Set a GUI username/password immediately; otherwise anyone who can reach port 8384 can manage the instance.
Plain HTTP on the LAN; do not expose the published port to the internet without a TLS reverse proxy.

---

Packaged for TOS 7 by Moechz. Upstream image used unmodified from Docker Hub.
