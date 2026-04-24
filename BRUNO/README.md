# Web-based AR Image Tracker

A minimal image-tracking AR experience for the browser. Uses MindAR for tracking, Three.js for rendering. Tracks a printed/displayed image and overlays an animated glTF model on top, with proper lighting and a contact shadow so it looks rooted on the page.

No build step. Drop the files into a GitHub Pages repo and it works.

## Repo layout

```
/
├── index.html      # the whole app (provided)
├── targets.mind    # compiled image target (you create this, see below)
└── model.glb       # your animated glTF (you provide)
```

## Setup

### 1. Compile your target image

MindAR needs your tracking image pre-processed into a `.mind` file.

1. Open the online compiler: https://hiukim.github.io/mind-ar-js-playground/tools/compile.html
2. Upload your image (JPG/PNG).
3. Click **Start**, wait for it to finish, then **Download** the `.mind` file.
4. Rename it to `targets.mind` and drop it next to `index.html`.

**Tracking quality tips** — the image matters a lot:

- High feature density (lots of corners, edges, contrast). Photos and detailed illustrations work well.
- **Avoid**: solid colors, simple logos, repeating patterns, anything rotationally symmetric.
- Print it cleanly and keep it flat. Matte paper > glossy (less glare).
- Bigger printed size = easier tracking from further away.

### 2. Add your animated model

- Export as `.glb` (binary glTF). Embed textures inside the file.
- Keep it reasonably lightweight (ideally under 5 MB for mobile).
- Name it `model.glb` and drop it next to `index.html`.

If your model loads in the wrong orientation or at the wrong size, see **Tuning** below.

### 3. Host on GitHub Pages

1. Push `index.html`, `targets.mind`, and `model.glb` to your repo.
2. In the repo: **Settings → Pages → Branch: main → Save**.
3. Wait a minute, then open `https://<you>.github.io/<repo>/` on your phone.

GitHub Pages serves over HTTPS, which is required for camera access on mobile.

## Tuning

All of these live in `index.html` inside the `loader.load('./model.glb', (gltf) => { ... })` block.

### Orientation

```js
model.rotation.x = Math.PI / 2;
```

MindAR's anchor space uses +Z as up (out of the image). Most glTFs are authored with +Y as up, so the model is rotated 90° around X to stand upright on the target. If yours starts out lying down, facing the wrong way, or exported +Z up:

- Lying down the wrong way → flip the sign: `-Math.PI / 2`
- Facing wrong direction → add a rotation around Z: `model.rotation.z = Math.PI;`
- Already +Z up → delete the rotation line entirely

### Scale

```js
model.scale.setScalar(0.5);
```

In MindAR anchor space, a unit of 1 roughly equals the width of your target image. A scale of `0.5` means the model is half the width of the image. Bump up or down to taste.

### Position

```js
// model.position.set(0, 0, 0);
```

By default the model's origin sits at the center of the image, at the image plane. Uncomment and tweak if you want to offset it, lift it off the surface, etc.

### Shadow softness / darkness

In the shadow-catcher block:

```js
new THREE.ShadowMaterial({ opacity: 0.45 })
```

Higher `opacity` = darker shadow. `0.3`–`0.6` usually reads best.

```js
keyLight.shadow.radius = 3;
```

Higher = softer shadow edges. Try `1` for sharper, `5`+ for diffuse.

### Lighting

Two lights in the scene:

- `hemi` — soft ambient fill (sky color on top, ground bounce on bottom). Intensity `1.0`.
- `keyLight` — directional light that casts the shadow. Intensity `1.8`, positioned up-right-front of the target.

Bump `keyLight.intensity` up for more dramatic shading, down for a flatter look. Move `keyLight.position` to change where the shadow falls.

## Troubleshooting

**Camera permission denied or nothing happens on Start**  
You need HTTPS. GitHub Pages provides it. If testing locally, use `npx serve` with HTTPS or open via your phone's mobile hotspot IP with a self-signed cert.

**Model loads but doesn't appear over the image**  
Open browser dev tools (desktop Chrome connected to your phone via USB works). Check console for glTF load errors. Double-check the file is at `./model.glb` relative to `index.html`.

**Target never tracks**  
Most common cause: low-feature-density image. Print a more detailed image and recompile the `.mind` file. Also check your lighting — tracking struggles in very dim light or with heavy glare.

**Model is huge / tiny / floating / buried in the image**  
See **Tuning** above. Start with scale, then rotation, then position.

**Animation doesn't play**  
Confirm the glTF actually contains animation clips — open it in https://gltf-viewer.donmccurdy.com/ and look for the animation dropdown. If there's nothing there, re-export from your 3D tool with animations included.

**Tracking is laggy**  
Model file too heavy, or too many polygons. Aim for under 50k triangles and compressed textures (Draco compression for geometry, KTX2 for textures is ideal). You can also reduce the shadow map size from `1024` to `512`.

## Credits

- [MindAR](https://github.com/hiukim/mind-ar-js) — MIT-licensed image tracking in the browser.
- [Three.js](https://threejs.org/) — 3D rendering.
