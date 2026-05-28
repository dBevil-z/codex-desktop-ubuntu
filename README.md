# codex-desktop-ubuntu

Unofficial Ubuntu and Debian packaging repository for Codex Desktop on Linux.

中文说明见 [README.zh-CN.md](README.zh-CN.md).

This project does not build Codex Desktop from original source code. It repackages the official upstream macOS `Codex.dmg` into a Linux-compatible Electron app and then publishes a native `.deb` package for Ubuntu-class systems.

## Status

- Target platform: Ubuntu / Debian / Pop!_OS / Linux Mint
- Package format: `.deb`
- Distribution model: GitHub Releases asset
- Repository type: metadata, documentation, and release hosting

The `.deb` file itself should be uploaded as a GitHub Release asset rather than committed into git history. A real package is typically hundreds of megabytes, which exceeds GitHub's normal repository file limits.

## Upstream Dependencies

This packaging flow depends on two upstream sources:

1. OpenAI Codex Desktop macOS distribution
   - Upstream app format: `Codex.dmg`
   - Purpose: provides the original Electron application bundle and app resources
2. Linux packaging and patching project
   - Repository: [ilysenko/codex-desktop-linux](https://github.com/ilysenko/codex-desktop-linux)
   - Purpose: extracts the DMG, patches the Electron bundle for Linux, rebuilds native modules, downloads a Linux Electron runtime, and packages the result

This repository is intentionally thin. It is meant to document, distribute, and track Ubuntu-ready builds that come from the upstream Linux adaptation project above.

## How It Works

The packaging process is not a generic "convert any DMG into a DEB" workflow. It works here because Codex Desktop is an Electron app and most of the application logic is packaged as cross-platform web assets.

The high-level flow is:

1. Download or reuse the latest upstream `Codex.dmg`
2. Extract `Codex.app` from the DMG
3. Unpack `app.asar`
4. Patch Linux-specific behavior in the bundled JavaScript
5. Rebuild native Node modules such as `better-sqlite3` and `node-pty` for Linux
6. Download a matching Linux Electron runtime
7. Reassemble the Linux app directory
8. Build a native Debian package

So this is better described as "rebuild the Electron runtime around the upstream app bundle" rather than "convert a DMG directly into a DEB".

## Why A Separate Release Repository

This repository exists so Ubuntu and Debian users can find:

- a prebuilt `.deb`
- a clear explanation of where it came from
- the exact upstream project that made the build possible
- release metadata such as version, build date, and checksums

The actual build logic should stay upstream in `ilysenko/codex-desktop-linux`. If the DMG structure changes and Linux patches need maintenance, that upstream repository is the source of truth.

## Build Source Of Truth

- Upstream Linux adapter: [ilysenko/codex-desktop-linux](https://github.com/ilysenko/codex-desktop-linux)
- Build entrypoint: `install.sh`
- Debian packaging script: `scripts/build-deb.sh`

If you want to reproduce a release locally:

```bash
git clone https://github.com/ilysenko/codex-desktop-linux.git
cd codex-desktop-linux
./install.sh --fresh
make deb
```

If you already have a local `Codex.dmg`, reusing it is often more stable for debugging:

```bash
./install.sh /absolute/path/to/Codex.dmg
make deb
```

## Current Release

This section is updated per package release.

- Source repository commit: `39da63072705e5f9330e6a5509626b320e7b8a1c` (`39da630`)
- Upstream DMG last-modified: `2026-05-16 02:26:28 UTC`
- Electron runtime version: `42.0.1`
- Package filename: `codex-desktop_2026.xxx_amd64.deb`
- Package size: `242M` (`253,224,686` bytes)
- SHA256: `a643958136e74f769003912aabfb7ccf5d8693cfb32b958ed091dfd620127328`

The built `.deb` should be attached to the corresponding GitHub Release for this repository.

## Recommended Release Layout

For each GitHub release, upload:

1. `codex-desktop_2026.xxx_amd64.deb`
2. `SHA256SUMS`

Users can then install with:

```bash
sudo apt install ./codex-desktop_2026.xxx_amd64.deb
```

## Notes And Caveats

- This is unofficial packaging. It is not published by OpenAI.
- Automatic updates inside the Linux package are local rebuilds, not direct downloads of a Linux package from OpenAI.
- A newer `Codex.dmg` can still require patch updates if the upstream Electron bundle changes shape.
- When a new DMG appears, it is usually best to update the upstream Linux packaging repository first, then rebuild.

## License And Attribution

Please keep upstream attribution intact:

- OpenAI owns the Codex Desktop application and branding.
- Linux adaptation and packaging logic comes from [ilysenko/codex-desktop-linux](https://github.com/ilysenko/codex-desktop-linux).

This repository should clearly state that it redistributes an unofficial Linux package derived from those upstream components.
