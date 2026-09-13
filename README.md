# Ligeia — public release distribution

This repository is **public on purpose**: it hosts the built, signed Ligeia
installers and the per-platform updater manifests that the in-app
auto-updater reads. The application source lives in the private `ligeia`
repository, which includes this repo as a git submodule at `releases/`.

## How the auto-updater uses this repo

Each shipped platform polls its own manifest via a templated endpoint:

```
https://raw.githubusercontent.com/LudoBermejoES/ligeia-public/main/latest-{{target}}-{{arch}}.json
```

e.g. `latest-windows-x86_64.json` for Windows. Each manifest independently
lists that platform's version, notes, download URL, and minisign `signature`.
The app downloads the installer and verifies the signature against the public
key baked into `tauri.conf.json` before applying the update. Only artifacts
signed with the project key are accepted.

Publishing one platform's manifest never changes what another platform's
manifest advertises — each OS is versioned independently.

## Contents

- `latest-<target>-<arch>.json` — one updater manifest per platform (version,
  notes, pub_date, that platform's url + signature). Only
  `latest-windows-x86_64.json` is published today; `darwin-aarch64`,
  `darwin-x86_64`, and `linux-x86_64` are reserved for when those builds exist.
- `Ligeia_<version>_x64-setup.exe` (+ `.sig`) — Windows NSIS installer, the
  updater payload.
- `Ligeia_<version>_x64_en-US.msi` — Windows MSI, a manual-download
  convenience (not consumed by the updater).

## Publishing a new version

See `doc/RELEASING.md` in the private `ligeia` repo. In short: build a signed
installer locally, copy the artifact (+ `.sig`) here, regenerate that one
platform's manifest with `scripts/publish-release.sh`, and push `main`.
`raw.githubusercontent.com/.../main/latest-<target>-<arch>.json` always
resolves to the newest committed manifest for that platform.
