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

## Privacy

This application runs entirely on your NAS. **The developer does not collect, receive or
have access to any data processed by the application.** There is no telemetry, no analytics
and no account.

| Topic | Detail |
|---|---|
| Data processed | The application's configuration, database, caches and logs, plus any files you create with it. |
| Where it is stored | The application data folder (`/Volume1/DockerAppData/syncthingdocker/`) on your NAS. Nothing is uploaded to the developer. |
| Retention period | Until **you** delete it. Logs are rotated or size-limited by the application; uninstalling keeps the data unless you tick “delete data”. |
| Your rights | **Access / export**: copy the data folder. **Correct**: change the values in the web UI. **Delete**: uninstall with “delete data”, or remove the folder after stopping the container. |
| Deletion channel | Delete the folder yourself, or email **moechz@163.com** (answered within 30 days). |
| Security measures | Container runs as uid 1000 (never root), no `privileged`, no host network, no `cap_add`, no Docker socket, one published port; image tag pinned and refreshed on upstream fixes. |
| Children | Not directed at children. |

**Full privacy policy:** [PRIVACY.md](PRIVACY.md) —
<https://github.com/Moechz/syncthing/blob/main/PRIVACY.md> — and the website version at
<https://www.moechz.com/privacy.html>.

## Third-party services (API disclosure)

The developer operates no server in this path. The application contacts only the services
**you configure**, plus (optionally) an update check and the image registry.

| Service | Endpoint | Data sent | Destination / region |
|---|---|---|---|
| Services you configure in the app | the address you enter | only the data that feature needs | normally your own LAN or the provider you chose |
| Update check (if enabled) | the upstream project's release feed | current version string | GitHub, US |
| Image pull | `registry-1.docker.io` or your registry mirror | anonymous pull | Docker Hub, US / your mirror |

See `PRIVACY.md` §6 for the full table. Disable any integration in the app's settings if you
do not want it.

## Licence and attributions

| Component | Licence | Notes |
|---|---|---|
| Syncthing | MPL-2.0 | upstream application, used unmodified inside the image |
| Container image `syncthing/syncthing:2.1.7` | see the image's own metadata | pulled from Docker Hub at install time |
| This packaging | GPL-3.0 (see `LICENSE`) | compose file, metadata, translations, icon |

“Syncthing”, “TerraMaster” and “TOS” are trademarks of their respective owners. This is an
independent community package by Moechz and is not affiliated with the upstream project or
TerraMaster. Full details: [NOTICE.md](NOTICE.md).

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


## Limitations

Only the web UI port is published. Direct LAN sync needs 22000/tcp+udp and 21027/udp; without them Syncthing falls back to discovery and relays, which is slower.
The web UI has no default password. Set a GUI username/password immediately; otherwise anyone who can reach port 8384 can manage the instance.
Plain HTTP on the LAN; do not expose the published port to the internet without a TLS reverse proxy.

---

Packaged for TOS 7 by Moechz. Upstream image used unmodified from Docker Hub.

## Support

- Issues and questions: <https://github.com/Moechz/syncthing/issues>
- Email: moechz@163.com
- Privacy: [PRIVACY.md](PRIVACY.md) · Licence: [NOTICE.md](NOTICE.md)
