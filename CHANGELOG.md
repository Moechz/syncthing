# Changelog

All notable user-visible changes to the Syncthing TOS application package.

## 2.1.7-1 — 2026-11-09

### Added
- First release. Syncthing (`syncthing/syncthing:2.1.7`, Docker Hub official image, unmodified)
  packaged as a TOS 7 Docker application, published on port 8025 and running
  as a non-root user (1000:1000).
- Persistent data is kept under `/Volume1/DockerAppData/syncthingdocker/`.
