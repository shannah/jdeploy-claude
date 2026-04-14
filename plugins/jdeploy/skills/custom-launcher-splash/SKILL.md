---
description: Add a custom HTML launcher splash screen to a jDeploy project. Creates a branded launcher-splash.html that displays during app installation and updates.
---

# jdeploy:custom-launcher-splash

Add a custom HTML launcher splash screen to a jDeploy project.

## Overview

jDeploy 5.2+ supports a custom HTML splash screen that displays during app installation and updates. By placing a `launcher-splash.html` file in the project root (same directory as `package.json`), the jDeploy launcher automatically uses it instead of the default splash screen.

**Key points:**
- File must be named exactly `launcher-splash.html` in the project root
- No `package.json` configuration needed — auto-detected by the launcher
- Uses inline HTML, CSS, and optionally JavaScript
- Supports Lottie animations via the `dotlottie-player` web component
- Displays during initial installation and updates
- Replaces the default launcher splash screen

## Instructions

### Step 1: Verify jDeploy Project

Check that the project has jDeploy configured:

```bash
ls package.json 2>/dev/null || echo "No package.json found"
```

If no `package.json` exists, suggest running `/jdeploy:setup` first.

### Step 2: Read Project Context

Gather branding information from the existing configuration:

```bash
node -p "const p = require('./package.json'); JSON.stringify({name: p.name, title: p.jdeploy?.title, description: p.description, version: p.version}, null, 2)" 2>/dev/null
```

Check for existing splash/icon files:

```bash
ls -la launcher-splash.html splash.png icon.png 2>/dev/null || echo "No existing splash files"
```

If `launcher-splash.html` already exists, read it and ask the user if they want to replace or modify it.

### Step 3: Ask User About Design Preferences

Use AskUserQuestion to understand what the user wants. Offer these style options:

1. **Gradient** — Modern gradient background with centered text and animated spinner (default)
2. **Minimal** — Clean white/light background with subtle loading bar
3. **Dark** — Dark theme with glowing accent colors
4. **Lottie** — Lottie animation as the loading indicator. The user can either:
   - Describe the animation they want and have it generated automatically
   - Provide an existing `.lottie` or `.json` file, or a URL to one
5. **Brand-colored** — Ask user for primary/secondary colors
6. **Custom** — User describes their desired look

Also ask:
- App name / title to display (default: `jdeploy.title` from package.json)
- Tagline or subtitle (optional)
- Loading message (default: "Installing and updating...")
- For **Lottie** style: ask the user whether they want to **describe** an animation to generate, or **provide** an existing `.lottie`/`.json` file or URL

### Step 4: Generate launcher-splash.html

Create `launcher-splash.html` in the project root. The file must be self-contained with all CSS inline in a `<style>` tag. No external resources.

#### Template: Gradient Style (Default)

```html
<!DOCTYPE html>
<html>
<head>
    <style>
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
        }
        .content {
            text-align: center;
        }
        h1 {
            font-size: 2.5em;
            margin-bottom: 10px;
            font-weight: 600;
        }
        .subtitle {
            font-size: 1.1em;
            opacity: 0.85;
            margin-bottom: 30px;
        }
        .spinner {
            border: 4px solid rgba(255,255,255,0.3);
            border-top: 4px solid white;
            border-radius: 50%;
            width: 50px;
            height: 50px;
            animation: spin 1s linear infinite;
            margin: 20px auto;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        .status {
            font-size: 0.95em;
            opacity: 0.8;
        }
    </style>
</head>
<body>
    <div class="content">
        <h1>APP_TITLE</h1>
        <p class="subtitle">SUBTITLE</p>
        <div class="spinner"></div>
        <p class="status">STATUS_MESSAGE</p>
    </div>
</body>
</html>
```

#### Template: Minimal Style

```html
<!DOCTYPE html>
<html>
<head>
    <style>
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
            background: #fafafa;
            color: #333;
        }
        .content {
            text-align: center;
        }
        h1 {
            font-size: 2em;
            font-weight: 600;
            margin-bottom: 8px;
            color: #222;
        }
        .subtitle {
            font-size: 1em;
            color: #666;
            margin-bottom: 30px;
        }
        .progress-bar {
            width: 200px;
            height: 4px;
            background: #e0e0e0;
            border-radius: 2px;
            margin: 20px auto;
            overflow: hidden;
        }
        .progress-bar::after {
            content: '';
            display: block;
            width: 40%;
            height: 100%;
            background: #667eea;
            border-radius: 2px;
            animation: loading 1.5s ease-in-out infinite;
        }
        @keyframes loading {
            0% { transform: translateX(-100%); }
            100% { transform: translateX(350%); }
        }
        .status {
            font-size: 0.85em;
            color: #999;
            margin-top: 15px;
        }
    </style>
</head>
<body>
    <div class="content">
        <h1>APP_TITLE</h1>
        <p class="subtitle">SUBTITLE</p>
        <div class="progress-bar"></div>
        <p class="status">STATUS_MESSAGE</p>
    </div>
</body>
</html>
```

#### Template: Dark Style

```html
<!DOCTYPE html>
<html>
<head>
    <style>
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
            background: #1a1a2e;
            color: #eee;
        }
        .content {
            text-align: center;
        }
        h1 {
            font-size: 2.5em;
            font-weight: 600;
            margin-bottom: 10px;
            background: linear-gradient(90deg, #e94560, #0f3460);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
        }
        .subtitle {
            font-size: 1em;
            color: #888;
            margin-bottom: 30px;
        }
        .spinner {
            width: 50px;
            height: 50px;
            border: 3px solid #333;
            border-top: 3px solid #e94560;
            border-radius: 50%;
            animation: spin 1s linear infinite;
            margin: 20px auto;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        .status {
            font-size: 0.9em;
            color: #666;
        }
    </style>
</head>
<body>
    <div class="content">
        <h1>APP_TITLE</h1>
        <p class="subtitle">SUBTITLE</p>
        <div class="spinner"></div>
        <p class="status">STATUS_MESSAGE</p>
    </div>
</body>
</html>
```

#### Template: Lottie Animation

Use this template when the user wants a Lottie animation as the loading indicator. There are two embedding strategies depending on the source:

**Strategy A: CDN URL (simplest, recommended)**

If the user has a URL to a `.lottie` or `.json` file hosted on LottieFiles or another CDN, use the `dotlottie-player` web component:

```html
<!DOCTYPE html>
<html>
<head>
    <script src="https://unpkg.com/@dotlottie/player-component@2.7.12/dist/dotlottie-player.mjs" type="module"></script>
    <style>
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
        }
        .content {
            text-align: center;
        }
        h1 {
            font-size: 2.5em;
            margin-bottom: 10px;
            font-weight: 600;
        }
        .subtitle {
            font-size: 1.1em;
            opacity: 0.85;
            margin-bottom: 20px;
        }
        dotlottie-player {
            width: 200px;
            height: 200px;
            margin: 0 auto;
        }
        .status {
            font-size: 0.95em;
            opacity: 0.8;
            margin-top: 10px;
        }
    </style>
</head>
<body>
    <div class="content">
        <h1>APP_TITLE</h1>
        <p class="subtitle">SUBTITLE</p>
        <dotlottie-player src="LOTTIE_URL" background="transparent" speed="1" loop autoplay></dotlottie-player>
        <p class="status">STATUS_MESSAGE</p>
    </div>
</body>
</html>
```

Replace `LOTTIE_URL` with the direct URL to the `.lottie` or `.json` file (e.g. `https://lottie.host/xxxxx/animation.lottie`).

**Strategy B: Inline Lottie JSON (fully self-contained)**

If the user has a local Lottie `.json` file and wants the splash to work fully offline, embed the animation data inline using the `lottie-web` library:

```html
<!DOCTYPE html>
<html>
<head>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/lottie-web/5.12.2/lottie.min.js"></script>
    <style>
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
        }
        .content {
            text-align: center;
        }
        h1 {
            font-size: 2.5em;
            margin-bottom: 10px;
            font-weight: 600;
        }
        .subtitle {
            font-size: 1.1em;
            opacity: 0.85;
            margin-bottom: 20px;
        }
        #lottie-container {
            width: 200px;
            height: 200px;
            margin: 0 auto;
        }
        .status {
            font-size: 0.95em;
            opacity: 0.8;
            margin-top: 10px;
        }
    </style>
</head>
<body>
    <div class="content">
        <h1>APP_TITLE</h1>
        <p class="subtitle">SUBTITLE</p>
        <div id="lottie-container"></div>
        <p class="status">STATUS_MESSAGE</p>
    </div>
    <script>
        lottie.loadAnimation({
            container: document.getElementById('lottie-container'),
            renderer: 'svg',
            loop: true,
            autoplay: true,
            animationData: LOTTIE_JSON_DATA
        });
    </script>
</body>
</html>
```

To use Strategy B:
1. Read the user's Lottie `.json` file
2. Replace `LOTTIE_JSON_DATA` with the raw JSON content (no quotes — it's a JS object literal)
3. This makes the animation fully embedded, though the `lottie-web` library is still loaded from CDN

**Strategy C: Generate from description (AI-generated)**

If the user describes the animation they want rather than providing a file, generate the Lottie JSON directly. This works well for loading-style animations: spinners, pulsing shapes, bouncing dots, rotating elements, progress indicators, etc.

Use Strategy B's HTML template (inline `lottie-web` + `animationData`), and generate the Lottie JSON to embed.

**Lottie JSON Format Reference:**

A Lottie animation is a JSON object with this top-level structure:

```json
{
  "v": "5.7.4",
  "fr": 30,
  "ip": 0,
  "op": 60,
  "w": 200,
  "h": 200,
  "nm": "animation",
  "ddd": 0,
  "assets": [],
  "layers": []
}
```

| Field | Description |
|-------|-------------|
| `v` | Lottie format version (use `"5.7.4"`) |
| `fr` | Frame rate (30 is standard) |
| `ip` | In-point frame (start, usually 0) |
| `op` | Out-point frame (end — at 30fps, `op: 60` = 2 second loop) |
| `w`, `h` | Canvas width/height in pixels |
| `nm` | Animation name |
| `ddd` | 3D flag (always 0 for 2D) |
| `assets` | External assets (usually empty for generated animations) |
| `layers` | Array of layer objects (the actual animation content) |

**Layer structure (shape layer):**

```json
{
  "ty": 4,
  "nm": "Layer Name",
  "sr": 1,
  "ks": {
    "o": { "a": 0, "k": 100 },
    "r": { "a": 0, "k": 0 },
    "p": { "a": 0, "k": [100, 100, 0] },
    "a": { "a": 0, "k": [0, 0, 0] },
    "s": { "a": 0, "k": [100, 100, 100] }
  },
  "ao": 0,
  "shapes": [],
  "ip": 0,
  "op": 60,
  "st": 0
}
```

| Field | Description |
|-------|-------------|
| `ty` | Layer type: `4` = shape layer |
| `ks` | Transform: `o` = opacity, `r` = rotation, `p` = position, `a` = anchor, `s` = scale |
| `shapes` | Array of shape objects (ellipses, rects, paths, fills, strokes, groups) |
| `ip`/`op`/`st` | In-point, out-point, start time for this layer |

**Animated property:**

A static property: `{ "a": 0, "k": 100 }` (a=0 means not animated, k=value)

An animated property: `{ "a": 1, "k": [ ...keyframes ] }` where each keyframe is:
```json
{
  "i": { "x": [0.33], "y": [1] },
  "o": { "x": [0.67], "y": [0] },
  "t": 0,
  "s": [0]
}
```

| Field | Description |
|-------|-------------|
| `t` | Keyframe time (frame number) |
| `s` | Start value at this keyframe |
| `i` | Bezier in-tangent (easing). `{"x":[0.33],"y":[1]}` is ease-out |
| `o` | Bezier out-tangent (easing). `{"x":[0.67],"y":[0]}` is ease-in |

For the final keyframe, only `t` and `s` are needed (no `i`/`o`).

**Common easing presets:**

| Easing | `i` | `o` |
|--------|-----|-----|
| Linear | `{"x":[1],"y":[1]}` | `{"x":[0],"y":[0]}` |
| Ease-in-out | `{"x":[0.42],"y":[1]}` | `{"x":[0.58],"y":[0]}` |
| Ease-out | `{"x":[0.33],"y":[1]}` | `{"x":[0.67],"y":[0]}` |

**Common shape types:**

Ellipse:
```json
{
  "ty": "el",
  "p": { "a": 0, "k": [0, 0] },
  "s": { "a": 0, "k": [80, 80] }
}
```

Rectangle:
```json
{
  "ty": "rc",
  "p": { "a": 0, "k": [0, 0] },
  "s": { "a": 0, "k": [80, 80] },
  "r": { "a": 0, "k": 10 }
}
```

Fill:
```json
{
  "ty": "fl",
  "c": { "a": 0, "k": [1, 1, 1, 1] },
  "o": { "a": 0, "k": 100 }
}
```

Colors in `c.k` are `[r, g, b, a]` with values 0–1 (not 0–255).

Stroke:
```json
{
  "ty": "st",
  "c": { "a": 0, "k": [1, 1, 1, 1] },
  "o": { "a": 0, "k": 100 },
  "w": { "a": 0, "k": 4 },
  "lc": 2,
  "lj": 2
}
```

`lc` = line cap (1=butt, 2=round, 3=square), `lj` = line join (1=miter, 2=round, 3=bevel).

Group (contains shapes + transforms):
```json
{
  "ty": "gr",
  "it": [ ...shapes, fill/stroke, {
    "ty": "tr",
    "p": { "a": 0, "k": [0, 0] },
    "a": { "a": 0, "k": [0, 0] },
    "s": { "a": 0, "k": [100, 100] },
    "r": { "a": 0, "k": 0 },
    "o": { "a": 0, "k": 100 }
  }]
}
```

The last item in a group's `it` array must be a transform (`ty: "tr"`).

**Trim paths (for stroke animations like spinning arcs):**

```json
{
  "ty": "tm",
  "s": { "a": 0, "k": 0 },
  "e": { "a": 0, "k": 75 },
  "o": { "a": 1, "k": [
    { "t": 0, "s": [0], "i": {"x":[1],"y":[1]}, "o": {"x":[0],"y":[0]} },
    { "t": 60, "s": [360] }
  ]}
}
```

`s` = start (0–100%), `e` = end (0–100%), `o` = offset (0–360°). Animate `o` for a spinning arc effect.

---

**Example: Spinning Ring Loader**

A circular arc that rotates continuously. Good default for most loading screens.

```json
{
  "v": "5.7.4", "fr": 30, "ip": 0, "op": 60, "w": 200, "h": 200,
  "nm": "spinner", "ddd": 0, "assets": [],
  "layers": [{
    "ty": 4, "nm": "ring", "sr": 1,
    "ks": {
      "o": {"a":0,"k":100}, "r": {"a":0,"k":0},
      "p": {"a":0,"k":[100,100,0]}, "a": {"a":0,"k":[0,0,0]},
      "s": {"a":0,"k":[100,100,100]}
    },
    "ao": 0,
    "shapes": [{
      "ty": "gr",
      "it": [
        {"ty":"el","p":{"a":0,"k":[0,0]},"s":{"a":0,"k":[80,80]}},
        {"ty":"st","c":{"a":0,"k":[1,1,1,1]},"o":{"a":0,"k":100},"w":{"a":0,"k":6},"lc":2,"lj":2},
        {"ty":"tm",
          "s":{"a":0,"k":0},
          "e":{"a":0,"k":25},
          "o":{"a":1,"k":[
            {"t":0,"s":[0],"i":{"x":[1],"y":[1]},"o":{"x":[0],"y":[0]}},
            {"t":60,"s":[360]}
          ]}
        },
        {"ty":"tr","p":{"a":0,"k":[0,0]},"a":{"a":0,"k":[0,0]},"s":{"a":0,"k":[100,100]},"r":{"a":0,"k":0},"o":{"a":0,"k":100}}
      ]
    }],
    "ip": 0, "op": 60, "st": 0
  }]
}
```

**Example: Three Bouncing Dots**

Three circles that bounce up and down in sequence. Friendly and playful feel.

```json
{
  "v": "5.7.4", "fr": 30, "ip": 0, "op": 40, "w": 200, "h": 200,
  "nm": "dots", "ddd": 0, "assets": [],
  "layers": [
    {
      "ty": 4, "nm": "dot1", "sr": 1,
      "ks": {
        "o": {"a":0,"k":100}, "r": {"a":0,"k":0},
        "p": {"a":1,"k":[
          {"t":0,"s":[60,120,0],"i":{"x":[0.42],"y":[1]},"o":{"x":[0.58],"y":[0]}},
          {"t":10,"s":[60,80,0],"i":{"x":[0.42],"y":[1]},"o":{"x":[0.58],"y":[0]}},
          {"t":20,"s":[60,120,0]}
        ]},
        "a": {"a":0,"k":[0,0,0]}, "s": {"a":0,"k":[100,100,100]}
      },
      "ao": 0,
      "shapes": [{"ty":"gr","it":[
        {"ty":"el","p":{"a":0,"k":[0,0]},"s":{"a":0,"k":[24,24]}},
        {"ty":"fl","c":{"a":0,"k":[1,1,1,1]},"o":{"a":0,"k":100}},
        {"ty":"tr","p":{"a":0,"k":[0,0]},"a":{"a":0,"k":[0,0]},"s":{"a":0,"k":[100,100]},"r":{"a":0,"k":0},"o":{"a":0,"k":100}}
      ]}],
      "ip": 0, "op": 40, "st": 0
    },
    {
      "ty": 4, "nm": "dot2", "sr": 1,
      "ks": {
        "o": {"a":0,"k":100}, "r": {"a":0,"k":0},
        "p": {"a":1,"k":[
          {"t":5,"s":[100,120,0],"i":{"x":[0.42],"y":[1]},"o":{"x":[0.58],"y":[0]}},
          {"t":15,"s":[100,80,0],"i":{"x":[0.42],"y":[1]},"o":{"x":[0.58],"y":[0]}},
          {"t":25,"s":[100,120,0]}
        ]},
        "a": {"a":0,"k":[0,0,0]}, "s": {"a":0,"k":[100,100,100]}
      },
      "ao": 0,
      "shapes": [{"ty":"gr","it":[
        {"ty":"el","p":{"a":0,"k":[0,0]},"s":{"a":0,"k":[24,24]}},
        {"ty":"fl","c":{"a":0,"k":[1,1,1,1]},"o":{"a":0,"k":100}},
        {"ty":"tr","p":{"a":0,"k":[0,0]},"a":{"a":0,"k":[0,0]},"s":{"a":0,"k":[100,100]},"r":{"a":0,"k":0},"o":{"a":0,"k":100}}
      ]}],
      "ip": 0, "op": 40, "st": 0
    },
    {
      "ty": 4, "nm": "dot3", "sr": 1,
      "ks": {
        "o": {"a":0,"k":100}, "r": {"a":0,"k":0},
        "p": {"a":1,"k":[
          {"t":10,"s":[140,120,0],"i":{"x":[0.42],"y":[1]},"o":{"x":[0.58],"y":[0]}},
          {"t":20,"s":[140,80,0],"i":{"x":[0.42],"y":[1]},"o":{"x":[0.58],"y":[0]}},
          {"t":30,"s":[140,120,0]}
        ]},
        "a": {"a":0,"k":[0,0,0]}, "s": {"a":0,"k":[100,100,100]}
      },
      "ao": 0,
      "shapes": [{"ty":"gr","it":[
        {"ty":"el","p":{"a":0,"k":[0,0]},"s":{"a":0,"k":[24,24]}},
        {"ty":"fl","c":{"a":0,"k":[1,1,1,1]},"o":{"a":0,"k":100}},
        {"ty":"tr","p":{"a":0,"k":[0,0]},"a":{"a":0,"k":[0,0]},"s":{"a":0,"k":[100,100]},"r":{"a":0,"k":0},"o":{"a":0,"k":100}}
      ]}],
      "ip": 0, "op": 40, "st": 0
    }
  ]
}
```

**Example: Pulsing Circle**

A circle that scales up and fades out repeatedly. Calm, minimal feel.

```json
{
  "v": "5.7.4", "fr": 30, "ip": 0, "op": 50, "w": 200, "h": 200,
  "nm": "pulse", "ddd": 0, "assets": [],
  "layers": [{
    "ty": 4, "nm": "circle", "sr": 1,
    "ks": {
      "o": {"a":1,"k":[
        {"t":0,"s":[80],"i":{"x":[0.42],"y":[1]},"o":{"x":[0.58],"y":[0]}},
        {"t":50,"s":[0]}
      ]},
      "r": {"a":0,"k":0},
      "p": {"a":0,"k":[100,100,0]}, "a": {"a":0,"k":[0,0,0]},
      "s": {"a":1,"k":[
        {"t":0,"s":[40,40,100],"i":{"x":[0.42,0.42,0.42],"y":[1,1,1]},"o":{"x":[0.58,0.58,0.58],"y":[0,0,0]}},
        {"t":50,"s":[120,120,100]}
      ]}
    },
    "ao": 0,
    "shapes": [{"ty":"gr","it":[
      {"ty":"el","p":{"a":0,"k":[0,0]},"s":{"a":0,"k":[80,80]}},
      {"ty":"fl","c":{"a":0,"k":[1,1,1,1]},"o":{"a":0,"k":100}},
      {"ty":"tr","p":{"a":0,"k":[0,0]},"a":{"a":0,"k":[0,0]},"s":{"a":0,"k":[100,100]},"r":{"a":0,"k":0},"o":{"a":0,"k":100}}
    ]}],
    "ip": 0, "op": 50, "st": 0
  }]
}
```

---

**Generating from a user description:**

When the user describes an animation instead of providing a file:

1. Ask the user to describe what they want. Examples:
   - "A spinning ring loader"
   - "Three dots bouncing in sequence"
   - "A pulsing circle that fades in and out"
   - "A gear icon rotating"
   - "A progress bar that fills and resets"
   - "Bouncing app logo letters"

2. Generate valid Lottie JSON based on the description. Use the examples above as starting points and adapt:
   - **Colors**: Match the splash screen background (use contrasting colors). Convert hex to 0–1 range: `#667eea` → `[0.4, 0.494, 0.918, 1]`.
   - **Timing**: Keep loops short (1–3 seconds). At 30fps, that's 30–90 frames for `op`.
   - **Size**: Design for a 200x200 canvas. The HTML container controls display size.
   - **Complexity**: Stick to shape layers with ellipses, rectangles, and strokes. Avoid paths unless the user specifically asks for a custom shape.

3. Embed the generated JSON inline using Strategy B's HTML template.

4. Tell the user to preview by opening `launcher-splash.html` in a browser. If the animation isn't right, iterate — adjust timing, colors, sizes, or easing as needed.

**Adapting colors for the background:**

When generating animations for different background styles:
- **Gradient/Dark backgrounds**: Use white or light-colored shapes (`[1,1,1,1]`)
- **Light/Minimal backgrounds**: Use the accent color from the gradient or brand colors
- **Brand-colored**: Match the user's brand palette

**Sourcing Lottie Animations (alternative to generation):**

If the user prefers a pre-made animation or the description is too complex to generate, suggest they browse:
- [LottieFiles](https://lottiefiles.com/) — free and premium animations (search for "loading", "spinner", "progress")
- They can download a `.lottie` or `.json` file and provide the path or URL

**Sizing:**

Adjust the `width` and `height` of the `dotlottie-player` element or `#lottie-container` to fit the animation. Typical sizes: 150px–300px for a loading indicator.

### Step 5: Write the File

Choose the appropriate template based on user preference and substitute:
- `APP_TITLE` — the app title from `jdeploy.title` or user input
- `SUBTITLE` — the user's tagline, or remove the element if none
- `STATUS_MESSAGE` — loading message (default: "Installing and updating...")

If the user chose **Brand-colored**, modify the gradient/accent colors in any template to match their brand colors.

If the user chose **Lottie**, ask for the animation source:
- **Describe it** → generate Lottie JSON from the description using Strategy C, embed with Strategy B's HTML template
- A URL to a `.lottie` or `.json` file → use Strategy A (dotlottie-player CDN)
- A local `.json` file path → read it and use Strategy B (inline animation data)
- No animation yet and doesn't want to describe one → suggest browsing LottieFiles

If the user chose **Custom**, design the HTML/CSS/JS to match their description while following these constraints:
- All CSS must be inline in a `<style>` tag
- External resources are limited to well-known CDNs for JS libraries (e.g. lottie-web, dotlottie-player) when needed
- Keep the design centered and responsive using `height: 100vh`
- Include some form of loading indicator (spinner, progress bar, Lottie animation, etc.)

Write the file to the project root:

```bash
# File: launcher-splash.html (project root, same directory as package.json)
```

### Step 6: Verify

Confirm the file was created:

```bash
ls -la launcher-splash.html
```

Tell the user:
1. The file `launcher-splash.html` has been created in the project root
2. It will automatically be used by the jDeploy launcher during installation and updates
3. No `package.json` changes are needed — the launcher detects the file automatically
4. They can preview the splash screen by opening `launcher-splash.html` in a web browser
5. To test the full experience, they can run `/jdeploy:install` to install the app locally

## Modifying an Existing Splash

If `launcher-splash.html` already exists, read the current file and ask the user what they'd like to change:
- Colors/gradient
- Text content
- Animation style (CSS spinner/progress bar, or switch to Lottie animation)
- Lottie animation (replace URL, swap to a different animation, or embed inline)
- Layout
- Complete redesign

Then edit the file accordingly, preserving any customizations the user doesn't want to change.

## Constraints

- **File name**: Must be exactly `launcher-splash.html`
- **Location**: Project root (same directory as `package.json`)
- **Self-contained CSS**: All CSS must be inline in a `<style>` tag. No external stylesheets.
- **JavaScript**: Allowed. Keep scripts minimal — only for animation libraries (Lottie) or lightweight effects. CDN-hosted libraries are acceptable (the splash displays during installation when network is available).
- **Responsive**: Use `height: 100vh` and flexbox centering so it looks good at any window size.
- **Lottie CDN versions**: Pin to specific versions to avoid breakage (e.g. `@dotlottie/player-component@2.7.12`, `lottie-web/5.12.2`).

## Quick Reference

```bash
# Check for existing splash
ls launcher-splash.html 2>/dev/null

# Preview in browser (macOS)
open launcher-splash.html

# Preview in browser (Linux)
xdg-open launcher-splash.html

# Preview in browser (Windows)
start launcher-splash.html
```
