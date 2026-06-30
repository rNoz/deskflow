# deskflow - local Arch package (with FD_CLOEXEC socket fix)

This directory rebuilds the official Arch `deskflow` package with one extra
local patch (`fd-cloexec.patch`) that the upstream package does not yet carry.
It is local-only: not an AUR package and not a PR to the Arch repo.

## Why this exists

On Wayland, deskflow-core spawns `wl-copy` / `wl-paste` for clipboard support.
The core's listen socket (TCP port 24800) was not marked close-on-exec, so
those clipboard child processes inherited the open listen socket. After the
server restarted, the new instance could not re-bind the port because a
lingering `wl-copy` / `wl-paste` child still held it:

    cannot bind address: Address already in use

`fd-cloexec.patch` sets `FD_CLOEXEC` on every socket right after creation in
`src/lib/arch/unix/ArchNetworkBSD.cpp`, so spawned helpers no longer inherit
it. The same change lives on the `rnoz/net-socket-cloexec` branch of the
source fork; this patch file is the packaged copy.

## Build and install

From this directory:

    cd ~/projects/deskflow/packaging/arch
    makepkg -si            # build + install, prompts for sudo at install

Or with yay (builds a local PKGBUILD dir, no AUR involved):

    yay -B ~/projects/deskflow/packaging/arch

What `makepkg` does:

1. clones upstream deskflow at tag `v1.26.0` into `./deskflow/`
2. `prepare()` applies `fd-cloexec.patch`
3. `build()` runs the full cmake / ninja build
4. `check()` runs `legacytests`
5. `package()` stages the install tree, producing
   `deskflow-1.26.0-2-x86_64.pkg.tar.zst`

Build artifacts (`deskflow/`, `src/`, `pkg/`, `*.pkg.tar.zst`) are gitignored.

## Verify the installed build

    pacman -Q deskflow                 # -> deskflow 1.26.0-2
    ss -ltnp | grep 24800              # running core should hold the socket

## Bumping the version later

When upstream releases a new tag:

1. bump `pkgver` in `PKGBUILD`
2. refresh `sha256sums` / `b2sums` for the git source
   (`makepkg -g`, or copy from the upstream Arch PKGBUILD)
3. re-apply / refresh `fd-cloexec.patch` against the new tag. If upstream
   merges the FD_CLOEXEC change, drop the patch and `prepare()` entirely and
   use the stock Arch PKGBUILD again.

Upstream Arch PKGBUILD to diff against:
https://gitlab.archlinux.org/archlinux/packaging/packages/deskflow
