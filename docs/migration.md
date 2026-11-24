# Moving applications out of redox-rpi

What moves from [redox-rpi](https://github.com/redox-rpi/redox-rpi) to this
repository, and what stays. Maintainer decision 2026-11-24, see the
[README](../README.md#the-split). The moves happen by pull request after
the v0.7.0 cut-off, one application per PR, so the release is not
disturbed. Paths are those in redox-rpi.

## Moves to apps

| What | Where in redox-rpi today | Notes |
|---|---|---|
| Tux Typing, t4kcommon, the SDL 1.2 libraries | branch `tuxtype` (PR #534): `patches/experiments/tuxtype/` with `recipes/games/tuxtype`, `recipes/wip/games/math/t4kcommon`, `qemu-tuxtype-check.sh` | Already package-only. The relibc scanf fix (relibc patch 0020) in the same PR stays in redox-rpi |
| Ri-li | branch `rili` (PR #532): `patches/experiments/orbital-apps/recipes/games/ri-li`, `qemu-rili-check.sh` | |
| SDL_mixer with MOD/XM (libmodplug) | branch `rili-mod` (PR #535) | A library only games use |
| Sopwith | `patches/experiments/orbital-apps/` (`sopwith`, `sdl2`, launcher entry `70_sopwith`); checks `launcher-sopwith` and `sopwith` in `scripts/regression-set.py`; PR #460 | In the v0.6.0 desktop image: see decision 1 |
| COSMIC Files and Edit | `patches/experiments/cosmic-apps-source/` (recipes, freedesktop-icons patch) | In the v0.6.0 desktop image: see decision 1 |
| TinyCC | `patches/experiments/tcc/` (recipe and 5 patches), `tools/tcc-bench`, `docs/topics/tcc.md` | In the release images; the compiler source is in redox-rpi/tinycc. See decision 1 |
| Assistant (llama.cpp, rinfer, Qwen3 model) | `patches/experiments/assistant*` (3 experiments), `tools/assistant`, `docs/topics/assistant.md` | |
| mdnsd | `tools/mdnsd`, `patches/experiments/mdns/` (PR #558) | Already package-only. It is a network service with an init script and will carry the HomeNodeServer; owner stays Network. See decision 4 |
| Binary packages of upstream programs | `patches/experiments/prebuilt-programs/` | An investigation, not a product; moves only if someone keeps it |

## Stays in redox-rpi

- **Patches** of kernel, relibc, base, bootloader, redoxfs, netutils,
  fdt and U-Boot (`patches/*`, not `experiments/`).
- **Desktop shell and boot experience:** `rpi-fb`, `orbital`, `desktop`,
  `orbital-apps` without the games (the launcher and icons),
  `keyboard-de`, `welcome` (`tools/welcome`, base patch 0040), `quiet-boot`,
  `boot-no-errors` and the other boot experiments.
- **The package manager side:** experiments `pkg-repo`, `pkg-hold`,
  `pkg-persist`, `tools/pkg-persist`, `scripts/publish-pkg-repo.sh`. They
  change for the per-release path (below).
- **System tools:** `timesync`, `timerd`, `reboot`, `redox-fatfs2`,
  `redoxfs-file`, `zoneinfo`, `rpi-boot-stub`, `devshd`, `tone`, `volume`.
- **Tools that test drivers and the system:** `gamepad-test`, `audiotop`,
  `cputop`, `netdiag`, `net6`, `mcast`, `i2c`, `spi`, `socdump`,
  `sigregs`, `polltest`, `icachetest`, `ttyinfo`, `chaos`, `log-triage`,
  and the `*-test`, `*-debug` and `*-stats` experiments.
- **Driver work in open PRs,** for example `usb-gamepad` (PR #378).

## Build and publishing

1. **Pin:** a file `os-release` with the redox-rpi tag (`v0.6.0` first).
2. **Build** (`scripts/build.sh`, on the build host, in its own clone):
   check out redox-rpi at the pinned tag; read the experiment list from
   that release's manifest (`redox-rpi-<tag>-pi3bplus.img.manifest.toml`, a release
   asset); copy this repository's recipes into a generated experiment in
   the clone that lists the packages; run redox-rpi's `build-image.sh`
   with the release options plus that experiment. This is what the
   `tuxtype` experiment does today, without touching redox-rpi's tree.
3. **Publish:** redox-rpi's `scripts/publish-pkg-repo.sh --out
   <pkg dir>/<release> <packages>`, with the same signing key as today.
   `repo.toml` then names the release's build ID.
4. **CI:** GitHub Actions checks that every `recipe.toml` parses and that
   `os-release` is a tag of redox-rpi. The full build runs on the build
   host, by hand or from a timer after a new OS release; setting up the
   timer is the admin session's work.

Changes in redox-rpi this needs (small PRs, after the cut-off):

- `pkg-repo`: `/etc/pkg.d/40_redox-rpi` names `<pkg dir>/<release>`,
  where the release is the image's own tag.
- `publish-pkg-repo.sh`: the OS packages also go to the per-release path;
  the old top-level path stays until no Pi uses it.

## Decisions needed

1. **Applications in release images.** The v0.6.0 desktop image has
   Sopwith, COSMIC Files/Edit and TinyCC. Do they leave the images and
   become packages only, from v0.8.0 on? The desktop then shows an empty
   launcher until someone installs a game.
2. **Development builds.** Nightly and `-dev` images have no release tag.
   Proposal: a second path, `<pkg dir>/main/`, built from redox-rpi main
   by the nightly run, for development images only.
3. **Owner.** Release & CI owns the repository, build and publishing;
   each application's topic owner owns its recipe (Graphics for the games
   and COSMIC, TCC for TinyCC, Assistant for the assistant). A topic page
   for "apps" in redox-rpi, or not?
4. **mdnsd** moves (package-only, a service people add) or stays (network
   service owned by Network)? Proposal: it moves.
