# Minimal NixOS `configuration.nix` with pinning using `npins`

This repository implements a minimal NixOS configuration that replaces
`nix-channel` functionality with [`npins`](https://github.com/andir/npins) to
manage your [`nixpkgs`](https://github.com/nixos/nixpkgs) version.

`minimal` is a feature of this repo, so that it is easy to understand and
incorporate into your own existing NixOS configuration. Honestly reading the
code is faster than reading the rationale in this README.

The specific pinned NixOS version in this repo is based on `nixos-25.05`, but
can be changed using the `npins` command.

In addition to the basic `npins` setup, this configuration also provides a
helper script `./rebuild` to setup `NIX_PATH` so that `./rebuild` always calls
`nixos-rebuild` using the pinned `nixpkgs`. Further, it sets up symlinks and
configuration options so that `nix-shell` and friends will also use the
`nixpkgs` which was used to build the currently running NixOS configuration,
instead of whatever is currently downloaded by `nix-channel`.

## Why pin `nixpkgs` within a NixOS configuration?

NixOS options change over time and a given configuration may not evaluate
correctly or at all against a different `nixpkgs` commit. Pinning your
`nixpkgs` version within your configuration is useful if you want to share your
NixOS configuration or you want to go back to an old configuration in a "known
good" state.

## What does this repo do?

This repo shows how `npins` can be used to pin a `nixpkgs` version within a
NixOS configuration. `npins` acts similarly to `nix-channel`, downloading a
specific commit of the NixOS/nixpkgs repository to a location on your computer,
in this case within `/nix/store/<hash>-source`. Where `npins` differs is that
it also creates a project-specific `npins/` folder containing a `sources.json`
which acts as a lockfile, tracking the specific commit of NixOS/nixpkgs against
which your NixOS configuration is being developed/evaluated.

Because of how `nixos-rebuild` works, it uses the environment variable
`NIX_PATH` (among other methods) to configure the active `nixpkgs`. The
`rebuild` script in this repo is a light wrapper around `nixos-rebuild` that
sets `NIX_PATH` so that `nixos-rebuild` uses the pinning `nixpkgs` version.

Additionally, in the typical `nix-channel` workflow, `nix-shell` uses the
`nixpkgs` version listed as current in `sudo nix-channel --list-generations`,
regardless of whatever version of `nixpkgs` was used to build the currently
activated system. The `configuration.nix` in this repo configures a NixOS
system that sets the `NIX_PATH` so that `nix-shell` and friends will use the
version of `nixpkgs` which was used to build the currently running system.

## Basic usage

1. Download and unpack the repository from
   https://github.com/NixOS/nixpkgs/archive/refs/heads/master.tar.gz
2. Add an appropriate `hardware-configuration.nix`. 
3. Edit `configuration.nix` to incoporate your packages and options.

   **Because this is a minimal configuration, in does not set
   `system.stateVersion`. You should set it to match the output of
   `nixos-option system.stateVersion` to prevent potential data loss if you use
   this configuration as the basis for updating an existing configuration. Read
   the comments in `configuration.nix` for more details. If you run this
   without adding a user to `configuration.nix`, you may not be able to log in.
   Similarly, if you are using `GRUB` instead of `systemd-boot`, you may need
   to change `boot.loader` options in `configuration.nix` appropriately.**

You will now use `./rebuild` anywhere you normally would use `nixos-rebuild`.
Running `./rebuild` without arguments will call `nixos-rebuild dry-build`,
showing you what new packages would be built.

In order to build the configuration and switch to it, without adding the new
configuration to your boot menu, use:

`./rebuild --sudo test`

Similarly, to build the configuration, add it to your boot menu and switch to
it, use:

`./rebuild --sudo switch`

Additional arguments passed to `./rebuild` are passed through to the underlying
`nixos-rebuild` call. For example, `./rebuild --help` will show you the man
page for `nixos-rebuild`. If you just want to experiment with this minimal
config, consider something like `./rebuild build-vm`.

## Quick `npins` primer

To update the tracked commit of `nixpkgs-25.05`, run:

`nix-shell -p npins --run 'npins update'`

To change the tracked channel to `nixpkgs-unstable`, use:

`nix-shell -p npins --run 'npins add --name nixpkgs channel nixos-unstable'`

Refer to the [npins repo](https://github.com/andir/npins) for more detailed
information on `npins` usage.

## Why not just use `flakes`?

`npins` is a simple tool that is conceptually very similar to `nix-channel`
while addressing the biggest piece of mutable state present in the default
`nix-channel` workflow.

NixOS configurations are personal, and if you want to use `flakes` go right
ahead. This repo shows just one option among many to pin your `nixpkgs`
version. 

## Acknowledgements

This repo basically implements ideas presented at

  - https://jade.fyi/blog/pinning-packages-in-nix/
  - https://piegames.de/dumps/pinning-nixos-with-npins-revisited/

If you also use overlays, https://piegames.de/dumps/nixpkgs-global-overlays/
offers some options on how to make those overlays available to `nix-shell` and
friends.
