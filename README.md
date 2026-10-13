# Capture the Evidence

A first-person 3D mystery game written in Rust on top of a custom Vulkan engine. You play a photographer in the home of Cecil Cedric Coulson IV, a celebrity bird who has gone missing. You walk through the house and photograph evidence on a limited roll of film. Once the film runs out, comic panels review what you found and ask you to name the culprit.

Built as a final project for CS 181G.

## Gameplay

1. **Intro:** comic panels set up the story and introduce the suspects.
2. **Investigation:** explore the living room in first person. Press **Space** to take a photo. A clue counts as found if enough of it is visible in the frame. You have **10 shots**.
3. **Review and accusation:** when the film is used up, the game shows the clues and asks who did it. Press a number key to choose a suspect.
4. **Ending:** the panels reveal whether you were right and what really happened.

Clue descriptions and the remaining film are printed to the terminal after each photo.

### Controls

| Key | Action |
| --- | --- |
| W A S D / arrow keys | Move |
| Mouse | Look around |
| Space | Take a photo |
| Enter | Advance story panels |
| 3–8 | Pick a suspect on the accusation panel |
| 1 / 0 | Pause / resume mouse look |
| R | Reset position |
| E | Reset view direction |
| Q | Reset player |

## Building and running

Requirements:

- A recent stable [Rust toolchain](https://rustup.rs/)
- A Vulkan-capable GPU and driver. On macOS, install the [Vulkan SDK](https://vulkan.lunarg.com/) (MoltenVK).

Run the game from the repository root so the `content/` paths resolve:

```sh
cargo run --release
```

The first build downloads a prebuilt Assimp library for `russimp`, so it needs network access.

## How it works

- **Rendering:** [vulkano](https://github.com/vulkano-rs/vulkano) 0.29 and [winit](https://github.com/rust-windowing/winit) are used for the window and GPU. The renderer has four pipelines in [src/renderer/](src/renderer/): `flat`, `textured`, `skinned` (animated meshes) and `sprites` (comic panels).
- **Clue detection:** each clue is drawn inside a Vulkan **occlusion query** (see [src/renderer/flat.rs](src/renderer/flat.rs)). When you take a photo, the game reads back how many pixels of each clue passed the depth test. A clue with at least 4000 visible pixels counts as found. This means a clue that is hidden behind furniture, or only just inside the frame, does not count.
- **Game flow:** [src/main.rs](src/main.rs) drives a small state machine (`StartScene → GamePlay → ClueDisplay → EndScene`). It switches to an orthographic camera to show the comic panels.
- **Assets:** models are loaded with [russimp](https://github.com/JoeOsborn/russimp) (Assimp bindings). The scene is `content/livingroom-base.glb` and the clue models are in `content/clues/`.

### Project layout

```
src/
  main.rs           game state, clue definitions, story panels
  engine.rs         main loop, window setup, asset loading entry points
  object.rs         player, clues, rooms, and movement/camera input
  camera.rs         perspective and orthographic cameras
  renderer.rs       render state shared by all pipelines
  renderer/         flat, textured, skinned, and sprite pipelines
  vulkan.rs         Vulkan device, swapchain, and render pass setup
  animation.rs      skeletal animation
  anim_2d.rs        2D sprite animation
  assets.rs         asset storage and handles
  input.rs          keyboard and mouse state
  types.rs          geometry types (RPrism, Rect, etc.)
content/            models, textures, and comic panels
```

### Changing clues

Clues are defined in `get_clues()` in [src/main.rs](src/main.rs). Each entry has a model path, the text shown when it is found, a position, a rotation and a scale. The renderer has room for `NUM_CLUES = 10` occlusion queries ([src/renderer/flat.rs](src/renderer/flat.rs)). If you change the number of clues, update that constant to match.

## Credits

- Katiana Wieser ([@kittyk4t](https://github.com/kittyk4t))
- Evan Von Oehsen

The engine is based on course code from CS 181G. The sample models in `content/khronos/` come from the [Khronos glTF sample models](https://github.com/KhronosGroup/glTF-Sample-Models).
