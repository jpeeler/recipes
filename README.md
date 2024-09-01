## recipes

This repository contains all of the recipes required to build Serpent OS
from source.

[![Repository status](https://repology.org/badge/repository-big/serpentos.svg)](https://repology.org/repository/serpentos)

## Quick start for boulder

`boulder` and `moss` rely on so-called `subuid` and `subgid` support.
If you do not already have this set up for your user in `/etc/subuid` and `/etc/subuid`, run this:

```bash
$ sudo touch /etc/sub{uid,gid}
$ sudo usermod --add-subuids 1000000-1065535 --add-subgids 1000000-1065535 root
$ sudo usermod --add-subuids 1065536-1131071 --add-subgids 1065536-1131071 "$USER"
```

**Non-Serpent OS hosts**

If you are not building on Serpent OS, you're going to have to install `boulder` first.
See [its readme][moss-boulder-readme] for instructions.

[moss-boulder-readme]: https://github.com/serpent-os/moss?tab=readme-ov-file#onboarding

### Local repository

Our `justfile` defaults to `local-x86_64` profile with boulder. While we traditionally shipped this pre-enabled configuration, we figured that mandating
`root`-user and world-accessible directories was generally a Bad Move.

**Create an empty local repository**

The path you use for this doesn't matter much, you can choose anything other than `~/.local_repo`
as long as the user account you want to use to run `boulder` has read/write access to it.

```bash
$ mkdir ~/.local_repo
$ moss index ~/.local_repo
```

If you're on Serpent OS and want to make the local repository available for package installation,
run the following command:

```bash
$ sudo moss repo add local file://${HOME}/.local_repo/stone.index -p 10
```

**Create a profile**

We'll add the (unversioned) volatile repository¹ at the bottom layer, and elevate
our local repository priority to take precedence.

```bash
$ boulder profile add local-x86_64 --repo name=volatile,uri=https://dev.serpentos.com/volatile/x86_64/stone.index,priority=0 --repo name=local,uri=file:///${HOME}/.local_repo/stone.index,priority=10
```

¹ the current one and only official online repository that you usually get all your packages from

**Create a `.env` file**

If you are not building on Serpent OS using the os-supplied boulder package, or if you want to specify custom arguments to the boulder invocation when using the `just` targets,
you might benefit from creating a `.env` file in the root of the `recipes/` directory, next to the supplied `justfile`.

_Example `.env` file:_

    BOULDER="${HOME}/.local/bin/boulder"
    BOULDER_ARGS="--data-dir=${HOME}/.local/share/boulder --config-dir=${HOME}/.config/boulder --moss-root=${HOME}/.cache/boulder"

The `justfile` is set up so you can also choose to specify either of the above environment variables on a command-line invocation of `just`:

_Example:_

    BOULDER_ARGS="--data-dir=${HOME}/.local/share/boulder" just build

## Go go go

Well, actually Rust.. Anyway, quickly try to `pushd m/m4/ && just build` or `pushd n/nano && just build` for a quick and easy confirmation that everything works OK.

## Git summary requirements

To keep git summaries readable, serpent-os requires the following git summary format

- `name: Add at v<version>`
- `name: Update to v<version>`
- `name: Fix <...>`
- `name: [NFC] <description of no functional change>`

The use of the `Initial inclusion` verbiage is _strongly discouraged_.

## Using `jq`

We provide `.jsonc` (JSON with comments) manifest files, the popular `jq` tool doesn't currently support `.jsonc` files; however, you can use the C preprocessor to strip any comments before passing to `jq` e.g.

`cpp -P -E manifest.x86_64.jsonc | jq .packages`

## Current focus

Packaging focus should be on bringing up the GNOME Desktop + associated stack

Other areas of focus:

 - Stateless enabling (+ hermetic usr)
 - Kernel enabling
 - Metrics-based performance improvements
 - Package updates and bug fixes

The aim for our desktop right now is to ship the following:

 - GNOME Shell
 - Flatpak w/ preconfigured flathub
 - GNOME Software (pending moss integration)
 - GNOME Console or Terminal
 - Nautilus
 - GNOME Control Center
 - Firefox
 - Thunderbird

### License

Unless otherwise specified, all packaging recipes are available under
the terms of the [MPL-2.0](https://spdx.org/licenses/MPL-2.0.html) license.

The Serpent OS Developers reserve the right to reject recipe contributions
which are not licensed to the Serpent OS Developer collective under the MPL-2.0 license

Individual software releases are available under the terms specified
upstream, collected in each `stone.yaml` recipe. Any patches against
a software package is under the relevant license for each upstream.

Copyright © 2020-2024 Serpent OS Developers
