# ShulkerPeek (WebAssembly)

Hover a shulker box and see what's inside: its contents as a 9 × 3 grid in the tooltip, under the lines Horizon
already shows, with item models, counts and durability bars in the box's own slot order.

This is a **`.hmod` (WebAssembly) port** of [vstxx's ShulkerPeek](https://github.com/vstxx/shulkerpeek), which is a
native `.hnmod` mod. The pure logic (slot normalisation, item offsets, dye tints, settings) and its tests come from
the original; the hooks are rewritten against the `.hmod` API 0.2. Both are MIT: see `LICENSE` (copyright vstxx and
the original's contributors).

> **Status:** it builds with `cargo horizon-mod build --wasm`, and its 23 unit tests and `clippy -D warnings`
> pass. It has **not been tested in game yet**, so the rendering details listed under "Known limits" are still
> unconfirmed. Bug reports are welcome.

## Features

- Works with the plain box and the 16 dyed boxes, whether empty, partly filled or full.
- Works wherever the game draws an item tooltip: inventory, chests, creative inventory and so on.
- Keeps every line of the tooltip (name, custom name, lore, advanced lines); the grid goes below them.
- Honours `minecraft:tooltip_display`: a hidden tooltip, or `minecraft:container` among its hidden components,
  is left to the game.
- A box that also holds a (non-empty) `minecraft:bundle_contents` is left to the game.
- Slots are placed as vanilla does (`ItemContainerContents.fromSlots`, then `copyInto`): a later entry for a slot
  replaces an earlier one, slots outside 0–26 and empty stacks show nothing, and at most 256 entries are read.
- Screen edges: the game positions the tooltip component, so it flips left near the right edge and moves up near
  the bottom edge. The mod only reports its size (162 × 54 GUI pixels).
- Resource packs: the background is the 9 × 3 slots of `gui/container/shulker_box`, mapped by fraction so a
  larger pack texture works. A pack without that texture gets plain slot squares.
- Never changes an item, an inventory or a world, and sends nothing to a server.

## Settings

Options → Mods → ShulkerPeek (default first):

| Setting | Choices |
|---|---|
| ShulkerPeek | On · Off |
| Show contents | Always · While holding Shift |
| Empty boxes | No grid · Show the empty grid |
| Grid colour | Plain · The box's dye |

Values are kept by the game, so they persist between sessions.

## Install

1. Put `shulkerpeek_wasm.hmod` in the `horizon-mods/` folder of your instance.
2. If you use the native `shulker_peek.hnmod`, **turn it off**: both draw the same grid, so each box would show two.
   The two have different ids (`shulker_peek` and `shulkerpeek_wasm`) and separate settings.

## Build from source

The wasm build needs Rust **1.99.1 through rustup** (a different system Rust is refused by the tool).

```sh
rustup toolchain install 1.99.1 --profile minimal --component rust-src --component clippy
cargo test
cargo clippy --all-targets -- -D warnings
cargo horizon-mod build --wasm      # writes target/horizon-mod/shulkerpeek_wasm.hmod
```

## Known limits

- **Contents sent by the server.** The original also patches Horizon's server so a shulker box's
  `minecraft:container` reaches the client in single player and on LAN. A WebAssembly client mod can't change what
  the server sends. That gap was in `lan::guest::network_stack` in the 2026-10-29 nightly; Horizon's `main`
  (`b2a84d2`, 2026-11-22) now writes both `minecraft:container` and `minecraft:tooltip_display` there. If the grid
  stays empty for a box that is full, your Horizon build still has the gap and this mod can't fix it.
- **Contents are read from the stack's component, not from `Host::item_container_slots`.** That call only answers
  for items a mod gave a container capability, which would also make other mods treat shulker items as containers.
- No grid outside a world: the item data and the settings reads need a loaded world.
- One `ItemStack` is built per occupied slot per frame while the tooltip shows (the API has no borrowed draw).
- The spacing under the name or lore is now the game's own tooltip-component spacing, so it may differ by a pixel
  or two from the native mod.
- Not yet confirmed in game: spacing, durability bars, dye tint, and resource-pack texture mapping.

## Test in game

1. Install the `.hmod`, turn the native `shulker_peek` off, start the game and open a creative world.
2. Run:
   ```
   give @s minecraft:red_shulker_box[minecraft:container=[{slot:0,item:{id:"minecraft:diamond",count:64}},{slot:13,item:{id:"minecraft:apple",count:5}},{slot:26,item:{id:"minecraft:stone"}}]]
   ```
3. Open the inventory and hover the box: diamonds (64), apples (5) and stone in slots 0, 13 and 26.
4. Check the Shift setting, an empty box with and without "Show the empty grid", "The box's dye" with a red box,
   a damaged tool inside (durability bar), the screen edges at GUI scales 1 to 4, and a resource pack that changes
   `gui/container/shulker_box`.
5. A box with `minecraft:tooltip_display={hide_tooltip:true}`, and one with
   `hidden_components:["minecraft:container"]`, must show no grid.

## Credits

ShulkerPeek © vstxx and contributors, MIT. This port keeps its logic and tests and replaces the native hooks with
`.hmod` ones. Not made or endorsed by the Horizon project or by Mojang.
