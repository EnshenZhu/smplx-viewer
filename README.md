(This project is still under construction!)

# SMPL-X Viewer

An interactive, browser-based viewer for the **SMPL-X** parametric human body model.
Load your own SMPL-X model file and explore how the body changes as you adjust
**shape**, **pose**, and **facial expression** parameters in real time — all computed
client-side, nothing is uploaded anywhere.

![tech: React + TypeScript + three.js] · dark "lab instrument" UI · runs fully offline once the model is loaded

---

## What it does

- **Shape (β)** — 16 shape coefficients that morph height, weight, and proportions.
- **Pose (θ)** — global orientation plus per-joint X/Y/Z rotations for the spine, head,
  arms, and legs, driven through the full linear-blend-skinning kinematic chain.
- **Hands** — aggregate finger-curl controls per hand, plus a "relaxed hands" toggle
  that applies the model's mean hand pose.
- **Expression (ψ)** — 10 facial-expression coefficients (when the loaded model
  includes an expression space).
- **Presets** — Neutral, Relaxed, Wave, Sit, T-pose.
- **Display** — wireframe, skeleton overlay, mesh opacity, camera recenter, and a
  "pose blend shapes" toggle to trade anatomical accuracy for faster dragging.

The forward pass (shape + expression blend shapes → joint regression → Rodrigues
rotations → pose blend shapes → kinematic chain → linear blend skinning) mirrors the
semantics of the reference `smplx` Python library.

---

## Getting the model file

The SMPL-X model weights are **licensed by Max Planck and cannot be redistributed**,
so they are not included here. To use the viewer:

1. Register and accept the license at **https://smpl-x.is.tue.mpg.de**.
2. Download the SMPL-X models and locate **`SMPLX_NEUTRAL.npz`**
   (the `.npz` format, not the `.pkl`).
3. Open the viewer and **drag the `.npz` file onto the loader** (or click to browse).
   Parsing happens entirely in your browser — the file never leaves your machine.

> Only SMPL-X model files (10,475 vertices, 55 joints) are supported — not SMPL or SMPL+H.

---

## Run locally

Requires Node 18+ and [pnpm](https://pnpm.io).

```bash
cd main/
pnpm install
pnpm dev        # start the dev server (Vite)
pnpm build      # type-check + production build into dist/
```

Then open the printed local URL and load your `.npz`.

### Single-file build

`bundle.html` in this folder is a fully self-contained build — open it directly in a
browser, no server needed. (Fonts load from Google Fonts when online; everything else
is inlined.)

---

## Project layout

```
main/
  src/
    smplx/
      npy.ts          # .npy / .npz parsers (NumPy dtypes, zip via fflate)
      skeleton.ts     # joint names, hand joint ranges, curated pose joint groups
      smplxModel.ts   # SmplxModel: load + forward pass (shape/pose/LBS) + caching
      params.ts       # ParamState, buildPose (deg→rad, hand mean, finger curl), presets
    components/
      threeViewer.ts  # imperative three.js scene (renderer, lights, mesh, skeleton)
      Viewer.tsx      # React wrapper around ThreeViewer
      ControlPanel.tsx# Shape / Pose / Face / View tabbed controls
      ModelLoader.tsx # drag-drop .npz loader
      ParamSlider.tsx # labeled slider with mono readout
    hooks/
      useSmplxGeometry.ts  # recompute vertices on param change (rAF-coalesced)
    App.tsx           # layout + state wiring
```

---

## Notes & limitations

- **Finger curl is a simplified aggregate control.** Real SMPL-X has 15 articulated
  joints per hand; this UI exposes a single curl per hand (about each finger's primary
  axis) plus the mean-hand toggle, rather than 90 individual finger sliders.
- **Pose blend shapes** are the most expensive part of the forward pass. If dragging
  feels heavy on a low-power device, turn them off in the **View** tab — the body will
  still pose correctly, just without the corrective deformations.
- Computation runs on the main thread, coalesced to one recompute per animation frame.
- The coordinate convention is Y-up with the camera looking from +Z.

## License

The application code here is yours to use freely. The **SMPL-X model itself is governed
by its own license** (https://smpl-x.is.tue.mpg.de) — review and comply with those terms.
