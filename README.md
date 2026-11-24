# redox-rpi/apps

Applications and ported software for
[Redox OS on the Raspberry Pi](https://github.com/redox-rpi/redox-rpi),
published as packages. People install them on a running Pi with `pkg`.
They are not part of the release images.

## The split

Maintainer decision, 2026-11-24: the operating system and the
applications live in separate repositories.

| Repository | What is in it |
|---|---|
| [redox-rpi](https://github.com/redox-rpi/redox-rpi) | The OS: kernel, relibc and driver patches, init, images, firmware glue, the package manager side (`pkg-repo`, `pkg-hold`, `pkg-persist`), and the tools that test drivers (for example `gamepad-test`) |
| **apps** (this one) | Ported software and our own applications: games such as Tux Typing, Sopwith and Ri-li, other applications, and the libraries only they need |

The rule of thumb: if the Pi needs it to boot, to run its drivers or to
test them, it belongs to redox-rpi. If a person installs it to use the
Pi, it belongs here.

## Building against an OS release

Redox has no stable system call ABI. relibc and the kernel change
together, so a program built for one redox-rpi release may not run on
another. Every package here is therefore built against one pinned
redox-rpi release:

- `os-release` in this repository names the redox-rpi tag (for example
  `v0.6.0`).
- The build checks out redox-rpi at that tag, builds the recipes of this
  repository with that release's options (from its build manifest) and
  publishes the packages for that release.
- A new OS release means a new build of every package here, published
  next to the old ones.

Recipes are in the Redox cookbook format, under `recipes/<group>/<name>/`
(`recipe.toml` and its patches). A full build needs the redox-rpi build
environment and takes about an hour, so it runs on the build host, not in
GitHub Actions; GitHub Actions checks the recipes and the pin.

## Installing on a Pi

The package repository has one path per OS release:

```text
<package server>/pkg/<release>/aarch64-unknown-redox/
```

An image built with redox-rpi's `pkg-repo` experiment lists the path of
its own release in `/etc/pkg.d/40_redox-rpi`, so `pkg` finds only
packages built for it:

```sh
pkg install tuxtype            # SD card image
pkg-persist install tuxtype    # network or direct boot (experiment pkg-persist)
```

## Contributing

The rules of the whole organisation apply:
[CONTRIBUTING.md](https://github.com/redox-rpi/redox-rpi/blob/main/CONTRIBUTING.md)
and the [project management plan](https://github.com/redox-rpi/redox-rpi/blob/main/docs/process/pmp.md).
Pull requests use the organisation's shared template from
[redox-rpi/.github](https://github.com/redox-rpi/.github); this repository
has no template of its own.

## Licence

Each recipe keeps the licence of the software it builds. Our own code and
recipes here are under the MIT licence, as in redox-rpi.
