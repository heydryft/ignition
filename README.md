# Ignition

A game launcher for Linux and macOS, for developers who stopped gaming years ago
and would like to play on the machine they already use every day.

One library across Steam, Epic, GOG and anything installed by hand. Windows
games run through translation on both platforms. You are never asked to pick a
translation layer, and the words "prefix", "runner" and "wine" never appear.

A game's environment is called a **cartridge**. You slot it in and it runs.

> Early design stage. See [SPEC.md](SPEC.md) for the specification and
> [AGENTS.md](AGENTS.md) for how work on this project is done.

## Status

| Platform                   | Status  |
| -------------------------- | ------- |
| macOS 14+ on Apple Silicon | planned, primary |
| Linux x86_64               | planned, primary |

Nothing is implemented yet. The runtime stack it depends on has been validated
by hand: Wine ARM64EC with DXMT reached 120–130 fps in a D3D11 title at
3440×1440 on an M5 Pro, with no proprietary launcher in the render path.

## Why

Playing PC games on a Mac or a Linux laptop is possible today and has been for
years. It is also an afternoon of reading about prefixes, translation layers and
environment variables before anything starts.

The people most able to work through that are often the least willing to. They
have a machine that could run these games, a library they stopped opening, and
no appetite for another yak to shave.

Ignition is the attempt to make that afternoon unnecessary.

## Licence

Not yet chosen. Components it will build on — Wine, DXMT, DXVK, FEX — are LGPL,
zlib and MIT, and must remain dynamically linked and replaceable.
