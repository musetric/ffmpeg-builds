# ffmpeg-builds

LGPL static **ffmpeg** and **ffprobe** binaries for Windows, Linux and macOS,
built in CI and published as GitHub Releases. This is the single source of
ffmpeg binaries for the Musetric monorepo — its `@musetric/ffmpeg` package
pins a release asset URL + SHA-256 per platform/arch.

## Why this repo exists

The Musetric desktop app bundles ffmpeg/ffprobe and needs **LGPL** builds — GPL
would block Mac App Store distribution and impose source-offer obligations on
the whole product. Prebuilt LGPL binaries are only partially available
([BtbN](https://github.com/BtbN/FFmpeg-Builds) covers Windows/Linux but not
macOS; macOS prebuilts all ship `--enable-gpl`). Building everything here gives
one uniform LGPL source with no third-party dependency.

The app only decodes/encodes audio (mp3/aac/vorbis/opus/flac/pcm), all handled
by ffmpeg's built-in codecs — **no external (GPL) libraries are needed**.
`./configure` is LGPL by default, and every job fails if `CONFIG_GPL` or
`CONFIG_NONFREE` is ever enabled.

## How each platform is built

| Platform | Arch | Runner / method |
|---|---|---|
| macOS | arm64, x64 | On `macos-14`: arm64 native, x64 cross-compiled (`clang -arch x86_64`) |
| Linux | x64, arm64 | Native on `ubuntu-24.04` / `ubuntu-24.04-arm`, **fully static** (`-static`) so it runs on any distro |
| Windows | x64, arm64 | **Cross-compiled** from Linux with [llvm-mingw](https://github.com/mstorsjo/llvm-mingw), statically linked |

## Licensing

- The workflows in this repo: **MIT** (see `LICENSE`).
- The produced ffmpeg/ffprobe binaries: **LGPL-2.1** (built with
  `--disable-gpl`, no external libraries; `COPYING.LGPLv2.1` is shipped inside
  each archive as `LICENSE.txt`).

## How to build a release

**All platforms at once:** Actions → **ffmpeg-release** → Run workflow → set
`ffmpeg_ref` (e.g. `n8.1.2`). It fans out to the three platform workflows.

**One platform:** run `ffmpeg-macos-lgpl`, `ffmpeg-linux-lgpl` or
`ffmpeg-windows-lgpl` directly.

Output: a release tagged `ffmpeg-<ref>` with these assets (each `+ .sha256`):

```
ffmpeg-lgpl-windows-x64.tar.gz    ffmpeg-lgpl-windows-arm64.tar.gz
ffmpeg-lgpl-linux-x64.tar.gz      ffmpeg-lgpl-linux-arm64.tar.gz
ffmpeg-lgpl-macos-x64.tar.gz      ffmpeg-lgpl-macos-arm64.tar.gz
```

Each archive is a gzip-compressed tarball (extractable with `tar` on every
platform, including GNU tar on Linux) containing `ffmpeg[.exe]`,
`ffprobe[.exe]` and `LICENSE.txt` at its root.

## How the monorepo consumes it

`@musetric/ffmpeg`'s `scripts/fetchFfmpeg.ts` pins, per platform/arch:

```
https://github.com/musetric/ffmpeg-builds/releases/download/ffmpeg-<ref>/<asset>.tar.gz
```

together with the SHA-256 from the matching `.sha256` sidecar.

## Notes

- Binaries are **unsigned**. The Musetric app's `electron-builder` signs and
  notarizes the nested macOS binaries when packaging the app.
- Both macOS arches build on the Apple Silicon `macos-14` runner (x64 is
  cross-compiled), so the deprecated Intel `macos-13` runner is not used.
- Actions needs write permission to publish releases (the workflows declare
  `permissions: contents: write`; for a private repo also check
  Settings → Actions → Workflow permissions).
