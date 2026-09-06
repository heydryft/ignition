# Ignition

A game launcher for Linux and macOS, for developers who stopped gaming years ago
and would like to play on the machine they already use every day.

## Who this is for

Someone who has a capable Mac or Linux laptop, a Steam library they abandoned,
and no interest in learning what a Wine prefix is. They will give this software
one evening. If it does not work in that evening, they will go back to not
gaming.

That constraint drives every decision below.

## What it does

- One library, several stores. Steam, Epic, GOG and anything installed manually
  appear as a single grid, not as tabs per store.
- A game is launched by selecting it. No configuration step precedes the first
  launch.
- Windows games run through translation on both platforms. The user is never
  asked to choose a translation layer.
- Custom and non-store installations are first-class, not an "advanced" mode.

## What it is not

- Not a Wine front-end. Wine is an implementation detail and must never appear
  in the interface.
- Not a tweaking tool. Per-game knobs exist, but the default path exposes none.
- Not a store. Ignition never sells anything and takes no cut.

## Vocabulary

The interface uses these words. Nothing else.

| Term          | Meaning                                                        |
| ------------- | -------------------------------------------------------------- |
| **Cartridge** | one game's self-contained environment (prefix, runtime, config) |
| **Library**   | the merged view across every connected source                   |
| **Source**    | a store account or a folder on disk                             |
| **Misfire**   | a cartridge that failed to launch, with a recoverable reason     |

"Bottle", "prefix", "runner", "proton" and "wine" are forbidden in user-facing
strings. They may appear in logs and developer documentation.

## Platforms

| Platform | Status  | Notes                                                     |
| -------- | ------- | --------------------------------------------------------- |
| macOS 14+ on Apple Silicon | primary | the harder target, so it leads |
| Linux x86_64               | primary | the mature target; most prior art exists here |
| macOS on Intel             | no      | Apple is removing the substrate; not worth the surface |
| Linux ARM64                | later   | the same stack as Apple Silicon minus Metal |

## Architecture

### Layers

```
Ignition UI
    |
Library service ......... merges sources into one set of game records
    |
Cartridge manager ....... creates, launches, repairs environments
    |
Runtime layer ........... Wine + graphics translation + CPU translation
```

The runtime layer is swappable per platform and per game. Nothing above it may
assume a particular implementation.

### Runtime matrix

| Concern      | macOS (Apple Silicon)                | Linux x86_64            |
| ------------ | ------------------------------------ | ----------------------- |
| Win32        | Wine ARM64EC                         | Wine / Proton           |
| D3D11/10     | DXMT → Metal                         | DXVK → Vulkan           |
| D3D12        | D3DMetal → Metal                     | VKD3D-Proton → Vulkan   |
| CPU          | Rosetta 2, then FEX                  | native                  |

### The macOS CPU problem

Rosetta 2 is fully supported through macOS 27 and largely removed in macOS 28.
Apple has said it will retain a subset "aimed at supporting older unmaintained
gaming titles". That may or may not cover Wine-hosted Windows games.

Ignition must not depend on that outcome. FEX-Emu provides `libarm64ecfex.dll`,
a drop-in ARM64EC emulator backend that Wine selects through a registry key —
the same slot Rosetta occupies today. The CPU translator is therefore a
configuration value, not an assumption.

FEX targets ARM64 Linux; the macOS port is not production-ready. In the ARM64EC
configuration FEX does not handle syscalls (Wine does), which keeps the porting
surface small. This is the single largest technical risk in the project.

### Licensing

| Component | Licence | Redistributable |
| --------- | ------- | --------------- |
| Wine      | LGPL-2.1 | yes, with source and dynamic linking |
| DXMT      | LGPL-2.1 | yes, same conditions |
| DXVK / VKD3D-Proton | zlib / LGPL | yes |
| FEX       | MIT      | yes |
| D3DMetal  | Apple    | bundled by CrossOver and Heroic; verify terms before shipping |
| Rosetta 2 | Apple    | never redistribute; it is part of the OS |

Ignition's own code is MIT. LGPL components must stay dynamically linked and
replaceable.

## Library model

The central design decision. Get this wrong and Ignition becomes a tabbed list
of stores.

A game is one record. A source is a way to obtain or launch it.

```
Game
  id            stable, internal
  title
  artwork
  installs[]    zero or more, each with a source and a cartridge
  sources[]     where it can be obtained

Install
  source        steam | epic | gog | manual
  path
  cartridge     nullable; native games have none
  last_played
```

The same title owned on two stores collapses to one tile with a launch-source
choice. Identity matching should use store IDs where available and fall back to
normalised titles; it must be correctable by the user, because it will be wrong
sometimes.

## Cartridge lifecycle

```
create      allocate a prefix, pick a runtime, record provenance
detect      inspect the executable: PE architecture, D3D version, launcher type
configure   choose graphics and CPU backends from detection, not from the user
launch      run with a bounded, supervised process tree
repair      reset a cartridge without touching game data
```

Detection drives configuration. If a title ships `rendersystemdx11.dll` and no
`d3d12`, the cartridge chooses the D3D11 path without asking.

Game data and cartridge state must be separable, so repairing an environment
never risks a re-download.

## Performance defaults

These were measured on an M5 Pro during development and belong in the default
configuration, not in a tuning screen.

| Setting | Effect |
| ------- | ------ |
| `WINEMSYNC_QLIMIT=16` | Mach port queue depth for msync defaults to 5 and fills under load, blocking senders. Removed a ~29 ms/frame stall in two unrelated games. |
| ARM64 bottle over win64 | Wine and DXMT run native; Metal encode fell from 55–100 µs to 20.9 µs per encoder. Roughly doubled frame rate in a D3D11 title. |
| `WINEDEBUG=-all` | Wine defaults to `err+all,fixme-all` and formats every message at runtime. |

Environment variables must be exported into the launch environment. Setting them
in a bottle configuration file is unreliable — msync silently falls back.

## Interface principles

1. The first screen is the library. Not a wizard, not a store, not settings.
2. Launching is one action from a cold start.
3. Failure states name a cause and offer one action. "Misfire: needs a graphics
   backend that is not installed. [Install]"
4. Nothing in the default path mentions prefixes, runners or translation layers.
5. Controller navigable end to end, because the reference is a console menu.

## Open questions

- Does Apple's gaming carve-out for Rosetta cover Wine-hosted Windows games?
- What is the real cost of porting FEX's ARM64EC backend to macOS?
- Can D3DMetal be redistributed, or must users supply it?
- How should anti-cheat titles be represented? Most kernel anti-cheat will never
  work; the library should say so up front rather than let someone install for
  an hour and fail.

## Non-goals

- Windows support
- Selling games
- Emulating consoles
- Being configurable enough to satisfy people who enjoy configuring things
