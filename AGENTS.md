# Working agreement for agents

Read `SPEC.md` first. It defines the product; this file defines how to work on
it.

## The one rule

**Measure before you claim, and measure before you change.**

This project exists because of a long debugging session where roughly eight
plausible theories were wrong and two measurements were right. The wins came
from `strings` on a binary and a microbenchmark. The losses came from confident
reasoning that was never checked.

If you cannot measure it, say that you cannot measure it.

## Rules that follow from that

1. **One variable per experiment.** Stacking five changes and reporting the
   total tells you nothing about which one worked, and hides the one that made
   it worse.
2. **Keep a known-good baseline and never edit it in place.** Changes go to a
   separate build or launcher. Promote only after it beats the baseline.
3. **Never benchmark a laptop on a flat battery.** Charging a depleted battery
   steals SoC power budget and halves frame rate. This looked exactly like a
   code regression for half an hour.
4. **State measured and inferred separately.** "GPU time is 8.95 ms" and "this
   is probably bandwidth-bound" are different kinds of sentence. Label the
   second one.
5. **Read the source before theorising about it.** Upstream comments frequently
   name the limitation you are guessing at. Several dead ends here were already
   documented as `FIXME` in the code.
6. **Do not touch the user's game settings.** Resolution, quality and upscaling
   are theirs. Optimise the stack, not their experience.

## Verification

A change is done when it has been run, not when it compiles.

- Performance claims need a before and an after, on the same scene, same power
  state, same resolution.
- "It launches" is not verification for a launcher. Reaching gameplay is.
- Prefer the platform's own instrumentation over inference. On macOS the Metal
  HUD (`MTL_HUD_ENABLED=1`) reports GPU time and frame interval per frame, and
  DXMT adds pass and encode counters to it.

## Things already established

Do not re-derive these. They cost real time.

- **Steam Cloud overwrites `cs2_video.txt` on launch.** Any file edit to a
  Steam game's settings will silently revert. Change settings in-game, or
  disable cloud sync for that title first.
- **Bottle-config environment variables are unreliable.** `WINEMSYNC` set in
  `cxbottle.conf` does not reach client processes; msync then falls back to the
  slow path without saying so. Export into the launch environment.
- **`wineserver -k` returns 1 when no server is running.** That is not a
  failure. Always follow with `-w` and wait on the specific prefix.
- **CrossOver creates a throwaway `wine.app` per launch for its Dock icon.**
  These accumulate as stale icons and temp directories.
- **D3DMetal is x86_64 only.** There is no native ARM64 D3D12 → Metal
  implementation, so D3D12 titles cannot benefit from an ARM64 prefix.
- **DXMT emits an empty tile-shader dispatch between two PSO switches to
  emulate an intra-pass barrier.** Expensive on TBDR. `raster_order_group` is
  declared in `airconv` but unused — the hardware feature that would replace it.

## Style

- Prose over bullet soup in documentation. Explain why, not just what.
- Comments explain reasoning, not syntax. If a constant came from a
  measurement, record the measurement.
- No emoji in code, commits or documentation.
- Commit messages describe the change and its evidence.

## Scope discipline

Do not add retries, telemetry, abstraction layers or configuration surfaces
that nobody asked for. This project's whole thesis is that the user should not
have to make choices. Every option added to the interface is a small failure of
detection.

When something cannot be done properly, say so and stop. A clear statement that
a path is blocked is worth more than a plausible-looking implementation that
does not work.
