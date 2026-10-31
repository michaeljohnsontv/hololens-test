(written by Claude Code)

# Holo Test

A single-page WebXR viewer for `.glb` models exported from Blender. Open it in a headset or phone browser, tap the AR button, and the model appears in your room. You can grab it, move it and spin it.

**Live site:** https://michaeljohnsontv.github.io/holo-test/

Built with [Babylon.js](https://www.babylonjs.com/). Nothing to install: it's one `index.html` plus your model files.

## Where it runs

| Device | Browser | Experience |
|---|---|---|
| Meta Quest 2 / 3 | Meta Quest Browser | Passthrough AR (Quest 2 passthrough is black-and-white) |
| HoloLens 2 | Microsoft Edge | See-through holographic AR |
| Android phone (ARCore) | Chrome | Handheld "magic window" AR |
| Desktop | Any modern browser | 3D preview only (drag to orbit) |
| iPhone / iPad | — | Not supported (iOS Safari has no WebXR) |

## Using it

1. Open the live site in the headset's browser.
2. Pick a model from the **Model** dropdown in the top-left, or use **Load .glb from this device…** to open a local file (it is never uploaded).
3. Tap the **AR goggles button** in the bottom-right corner and allow the permission prompt.
4. After about half a second, the model is placed 1 m in front of you, just below eye level.

### Controls in AR

| Action | Input |
|---|---|
| Grab and move | Trigger or grip on the controller, or pinch with hand tracking |
| Spin in place | Thumbstick left/right (controllers only) |
| Switch model | Exit AR, choose from the dropdown, re-enter AR |

## Adding a model

1. Export from Blender as **glTF Binary (`.glb`)**, with transforms applied. 1 Blender unit = 1 metre.
2. Upload the `.glb` to this repository: **Add file → Upload files → Commit changes**. Web uploads are limited to 25 MB per file.
3. Edit `index.html` and add a line to the `MODELS` list:

   ```js
   const MODELS = [
     { name: "A35 - 20deg", file: "A35 gas turbine - sector 20deg 20261028.glb", scale: 1 },
     { name: "Engine part", file: "engine.glb", scale: 0.1 },
   ];
   ```

   - `name` is the label shown in the dropdown.
   - `file` must match the file name in the repository exactly, including capitals and spaces.
   - `scale` resizes the model (`0.1` = one tenth, `10` = ten times).
4. Commit. GitHub Pages redeploys automatically. Check the **Actions** tab for a green check, then reload the page.

File names without spaces (e.g. `engine-part.glb`) avoid surprises in web addresses.

## How `index.html` is organised

### Page setup

- **Meta tags and title:** text encoding, mobile-friendly sizing, and the tab name.
- **Styles:** a full-screen canvas, a blue-grey page background, and the top-left control panel.
- **Babylon.js scripts:** the 3D engine and its glTF/`.glb` loader, loaded from Babylon's CDN.
- **`<canvas>`:** the surface the 3D scene is drawn on.
- **`#ui` panel:** the model dropdown, the local-file button with its hidden file picker, and a status line. This is ordinary web content, visible on the page but not inside AR.

### Script, top to bottom

1. **`MODELS`:** the list of models for the dropdown. This is the main thing to edit.
2. **Scene setup:** the engine and the scene. The background is transparent so passthrough shows through in AR. Also adds a desktop orbit camera, an overhead light, and reflection lighting so metallic materials don't render black.
3. **`load()`:** removes the previous model, loads a new one from a file name or a local file, applies the scale and position, attaches the grab behaviour, and updates the status line.
4. **Dropdown and local-file wiring:** fills the dropdown from `MODELS`, loads the first model on start, and connects the local-file button.
5. **`placeInFront()`:** places the model 1 m ahead of the headset, 0.2 m below eye level, in whichever direction you're facing.
6. **AR setup:** adds the AR button with hand tracking enabled, and turns off teleport and turning so the thumbsticks are free. After entering AR, it waits about 30 frames for tracking to settle, then calls `placeInFront()`.
7. **Thumbstick spin:** registers each controller's thumbstick as it connects. Every frame, it rotates the model around its own bounding-box centre by the stick's left/right amount. Includes a dead zone so the model doesn't drift. Speed is set by `SPIN_SPEED` (radians per second; negative reverses the direction).
8. **Render loop and resize:** redraws every frame and adjusts the canvas when the window changes size.

## Troubleshooting

| What you see | Likely cause |
|---|---|
| GitHub 404 page | Pages not enabled (**Settings → Pages**) or still deploying (**Actions** tab) |
| Status says "Couldn't load…" | The `file` in `MODELS` doesn't exactly match the uploaded file name |
| Model loads but looks black | Metallic materials need reflection lighting, which is already included; make sure the latest `index.html` is uploaded |
| No AR button | The browser or device doesn't support WebXR AR (e.g. iPhone), or the page isn't served over HTTPS |
| Old version still showing | The browser cached it; fully reload the page |
| Can't find the model in AR | Look around; very large models can surround you. Adjust `scale` |

## Notes

- This repository is **public** (required for GitHub Pages on a free account), so anyone can view the page and download the models. Don't upload confidential or employer-owned models; use **Load .glb from this device…** for those instead.
- The `<select>` dropdown isn't visible inside AR. A floating in-AR menu is a possible future addition.

## Ideas for later

- Thumbstick up/down to scale
- Two-handed grab to rotate and resize
- A/X button to reset the model in front of you
- Highlight parts and show their Blender names
- Exploded view, cross-section, X-ray toggle
- Play animations exported from Blender
- Floating 3D menu or wrist menu inside AR
