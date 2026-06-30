# Local multi-platform workflow (one repo, Mac + Linux)

This is a single fork (`github.com/rNoz/deskflow`) cloned on two machines.
There is no separate "Mac project" and "Linux project": same repo, same
structure, same branches. Pick the branch and run the platform-specific steps
below.

## Branches

| Branch | Purpose | Where it runs |
| --- | --- | --- |
| `master` | tracks upstream | both |
| `rnoz/macos-right-left-modifiers` | clean PR #9916 (one commit) | upstream PR only |
| `rnoz/macos-altgr-typing` | clean PR #9917 (stacked on modifiers) | upstream PR only |
| `rnoz/net-socket-cloexec` | clean cloexec branch | upstream/source ref |
| `rnoz/integration` | everything below, the branch you actually run locally | both |

`rnoz/integration` = the 2 macOS key fixes + the FD_CLOEXEC socket fix +
`packaging/arch/`. Check it out on both machines for an identical tree:

    git checkout rnoz/integration

Keep the three single-topic branches clean for upstream PRs; do day-to-day
local work and builds from `rnoz/integration`.

## On Linux (the server)

Relevant local change here: the FD_CLOEXEC socket fix (Wayland clipboard /
wl-copy restart bug). Build and install the patched package:

    cd packaging/arch
    makepkg -si            # or: yay -B ~/projects/deskflow/packaging/arch

Details + verify steps: `packaging/arch/README.md`.
Personal monitoring notes are kept locally in `_local/` (git-excluded, not
committed).

## On Mac (the client)

Relevant local changes here: the two macOS key fixes (right/left modifiers,
AltGr via right Option). These are client-side and take effect once you run a
build that includes them. Build from source to test:

    cmake -B build -S . -G Ninja
    cmake --build build

Then run the built client. (Once PRs #9916/#9917 merge upstream, a stock
release build will include them and you can drop the local build.)

## Configuration (important)

None of the local patches require special configuration:

- The two macOS fixes are pure key-mapping code (`OSXKeyState.cpp/.h`); there
  is no option, no GUI setting, nothing to enable. They work automatically.
- The FD_CLOEXEC fix needs no config either. The only relevant runtime
  condition is that clipboard sharing is on (`clipboardSharing = true`, the
  default GUI option) - that is what makes the server spawn `wl-copy` /
  `wl-paste`, i.e. the scenario the fix addresses.
- The optional server-side custom config file is just a normal screen layout
  (your screens, links, standard options). It is byte-identical to the
  GUI-generated config. Using an external config file is a preference (GUI
  toggle), not a requirement of any patch.

So there is nothing config-related to document in the upstream PRs as a
prerequisite.
