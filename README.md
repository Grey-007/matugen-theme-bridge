# Matugen Theme Bridge

> Dynamic Material You / Matugen color palettes in VS Code & VSCodium — **zero reloads, zero restarts.**

Matugen Theme Bridge watches a single JSON file on disk. When that file changes (because [matugen](https://github.com/InioX/matugen) re-generated it after a wallpaper change), the extension instantly merges the new colors into VS Code's `workbench.colorCustomizations` — updating every visible UI element in real time.

```
wallpaper changes
      │
      ▼
 matugen generates
 vscode-palette.json
      │
      ▼
 FileSystemWatcher fires
      │  (debounced 300 ms)
      ▼
 palette is read & validated
      │
      ▼
 colors merged into
 workbench.colorCustomizations
      │
      ▼
 VS Code repaints — no reload ✅
```

---

## Features

| Feature | Details |
|---|---|
| 🎨 Live color updates | Colors apply the moment the palette file changes |
| 🔒 Safe merge | Only overwrites its own previous colors — never touches user settings |
| 🛡 Malformed file guard | Gracefully handles missing or broken JSON without crashing |
| ♻️ Zero reload | Uses `workbench.colorCustomizations` — VS Code repaints without restarting |
| ⚡ No polling | Pure event-driven via VS Code's `FileSystemWatcher` |
| 🎛 Configurable | Custom palette path, debounce delay, log verbosity |
| 🖥 Cross-editor | Works on VS Code and VSCodium identically |

---

## Folder Structure

```
matugen-theme-bridge/
├── src/
│   ├── extension.ts              ← Entry point: wires everything together
│   ├── types.ts                  ← Shared type definitions
│   ├── engine/
│   │   ├── paletteReader.ts      ← Reads & validates vscode-palette.json
│   │   └── colorApplier.ts       ← Merges & writes workbench.colorCustomizations
│   ├── watcher/
│   │   └── paletteWatcher.ts     ← FileSystemWatcher + debounce wrapper
│   └── utils/
│       ├── logger.ts             ← Output channel logger (singleton)
│       └── debounce.ts           ← Generic debounce utility
├── resources/
│   ├── themes/
│   │   ├── matugen-dark-base.json   ← Static base dark theme (contributed)
│   │   └── matugen-light-base.json  ← Static base light theme (contributed)
│   └── vscode-palette.example.json  ← Example palette file for matugen
├── package.json
├── tsconfig.json
├── .vscodeignore
└── README.md
```

---

## Prerequisites

- **Node.js** ≥ 18
- **npm** ≥ 8
- **VS Code** ≥ 1.75 **or VSCodium** ≥ 1.75
- [matugen](https://github.com/InioX/matugen) (optional — you can also write the palette JSON manually)

---

## Installation

Option A - Install from VSIX (recommended)

Download ` matugen-theme-bridge-1.0.0.vsix` from releases
then open VSCodium ` ctrl+shift+P ` -> Extension:Install Extension -> install form vsix
select `matugen-theme-bridge-1.0.0.vsix` and Done 

                    or 

```
# 5a. Install in VS Code
code --install-extension matugen-theme-bridge-1.0.0.vsix

# 5b. Install in VSCodium
codium --install-extension matugen-theme-bridge-1.0.0.vsix
```
### Option B — Install from VSIX (recommended)

```bash
# 1. Clone the repository
git clone https://github.com/yourusername/matugen-theme-bridge
cd matugen-theme-bridge

# 2. Install dependencies
npm install

# 3. Compile TypeScript
npm run compile

# 4. Package as .vsix
npm run package
# → creates matugen-theme-bridge-1.0.0.vsix

# 5a. Install in VS Code
code --install-extension matugen-theme-bridge-1.0.0.vsix

# 5b. Install in VSCodium
codium --install-extension matugen-theme-bridge-1.0.0.vsix
```

### Option C — Open in development mode (no packaging needed)

```bash
git clone https://github.com/yourusername/matugen-theme-bridge
cd matugen-theme-bridge
npm install
npm run compile

# Open in VS Code / VSCodium
code .
# Press F5 → "Run Extension"
```

---

## Setup

### Step 1 — Select the theme

Open the Command Palette (`Ctrl+Shift+P`) and run:

```
Preferences: Color Theme
```

Choose **Matugen Dark** or **Matugen Light**.

This activates the static base theme. The extension will layer dynamic colors on top.

---

### Step 2 — Create the palette directory

```bash
mkdir -p ~/.config/VSCodium/User/Theme
```

> For **VS Code** (not VSCodium) the default path is the same structure but you can override it — see [Configuration](#configuration).

---

### Step 3 — Create your palette file

Copy the example and edit it, **or** let matugen generate it automatically.

```bash
# Manual: copy the example
cp resources/vscode-palette.example.json \
   ~/.config/VSCodium/User/Theme/vscode-palette.json
```

The palette file is a flat JSON object where every key is a VS Code color token:

```json
{
  "_meta": { "generated": "2024-01-15", "variant": "mocha" },

  "editor.background":       "#1e1e2e",
  "editor.foreground":       "#cdd6f4",
  "activityBar.background":  "#181825",
  "statusBar.background":    "#181825",
  "tab.activeBorderTop":     "#cba6f7",
  "focusBorder":             "#cba6f7"
}
```

Rules:
- Keys must contain a `.` (VS Code token format)
- Values must be valid hex colors (`#rgb`, `#rgba`, `#rrggbb`, `#rrggbbaa`)
- Keys starting with `_` are treated as metadata and ignored
- Unknown or invalid entries are silently skipped — the rest still apply

---

### Step 4 — Integrate with matugen (optional)

If you use matugen for automatic Material You theming from your wallpaper, add a template that writes to the palette path.

Create `~/.config/matugen/templates/vscode.json`:

```json
{{colors.dark.surface}}
```

Actually, use matugen's full template system. Here is a working template example:

**`~/.config/matugen/templates/vscode-palette.json`**:
```
{
  "_meta": {
    "source": "matugen",
    "variant": "{{mode}}"
  },

  "editor.background": "{{colors.surface.default.hex}}",
  "editor.foreground": "{{colors.on_surface.default.hex}}",
  "editorLineNumber.foreground": "{{colors.on_surface_variant.default.hex}}",
  "editorLineNumber.activeForeground": "{{colors.primary.default.hex}}",
  "editorCursor.foreground": "{{colors.primary.default.hex}}",
  "editor.selectionBackground": "{{colors.surface_container_highest.default.hex}}",
  "editor.inactiveSelectionBackground": "{{colors.surface_container_high.default.hex}}",
  "editor.lineHighlightBackground": "{{colors.surface_container_low.default.hex}}",
  "editor.findMatchBackground": "{{colors.tertiary_container.default.hex}}80",
  "editor.findMatchHighlightBackground": "{{colors.tertiary_container.default.hex}}40",
  "editor.wordHighlightBackground": "{{colors.secondary_container.default.hex}}40",
  "editor.wordHighlightStrongBackground": "{{colors.secondary_container.default.hex}}60",
  "editor.hoverHighlightBackground": "{{colors.surface_container_high.default.hex}}",

}
```

Add the output path to matugen's config (`~/.config/matugen/config.toml`):

```toml
[config]
reload_apps = true

[[templates]]
input_path  = "~/.config/matugen/templates/vscode-palette.json"
output_path = "~/.config/VSCodium/User/Theme/vscode-palette.json"
```

Now whenever you run `matugen image ~/wallpapers/new.jpg`, VS Code updates its colors automatically within ~300 ms.

---

## Commands

Open the Command Palette (`Ctrl+Shift+P`) and type **Matugen**:

| Command | Description |
|---|---|
| `Matugen: Apply Palette Now` | Force re-read and re-apply the palette file |
| `Matugen: Clear Color Overrides` | Remove all managed colors, restore base theme |
| `Matugen: Show Bridge Status` | Show how many colors are active and when last applied |

---

## Configuration

All settings are in `settings.json` under the `matugenBridge` namespace.

```jsonc
{
  // Override the default palette file path.
  // Default: ~/.config/VSCodium/User/Theme/vscode-palette.json
  // Supports ~ expansion.
  "matugenBridge.palettePath": "",

  // Milliseconds to wait after a file change before re-applying.
  // Prevents thrashing if the file is written in multiple small chunks.
  "matugenBridge.debounceMs": 300,

  // Disable the extension without uninstalling it.
  "matugenBridge.enabled": true,

  // Log verbosity in the "Matugen Bridge" output channel.
  // "silent" | "info" | "debug"
  "matugenBridge.logLevel": "info"
}
```

**Example — use VS Code instead of VSCodium:**
```json
"matugenBridge.palettePath": "~/.config/Code/User/Theme/vscode-palette.json"
```

**Example — custom location:**
```json
"matugenBridge.palettePath": "~/.cache/matugen/vscode-palette.json"
```

---

## How It Works Internally

```
┌─────────────────────────────────────────────────────────────┐
│                      extension.ts                           │
│                                                             │
│  activate()                                                 │
│    │                                                        │
│    ├─ resolvePalettePath()  ─────────────────────────────┐  │
│    │                                                     │  │
│    ├─ PaletteWatcher.start(path)                         │  │
│    │    └─ vscode.FileSystemWatcher(dir glob)            │  │
│    │         onChange → debounce(300ms) → applyPalette() │  │
│    │                                                     │  │
│    └─ applyPalette() ◄───────────────────────────────────┘  │
│         │                                                   │
│         ├─ paletteReader.readPalette(path)                  │
│         │    └─ fs.readFileSync → JSON.parse → validate     │
│         │         returns { ok: true, colors: ColorMap }    │
│         │                                                   │
│         └─ colorApplier.applyColors(colors)                 │
│              │                                              │
│              ├─ read  workbench.colorCustomizations         │
│              ├─ strip  previously-managed keys only         │
│              ├─ merge  new palette colors on top            │
│              ├─ write  __matugenBridge metadata marker      │
│              └─ config.update() → VS Code repaints ✅       │
└─────────────────────────────────────────────────────────────┘
```

**Why no reload?**
`vscode.workspace.getConfiguration().update()` writes to `settings.json` and triggers VS Code's internal settings reconciler, which repaints the workbench UI without reloading the window. This is the same mechanism VS Code itself uses when you change a theme color from the settings editor.

**Why is it safe to merge?**
Every time colors are applied, the extension writes a `__matugenBridge` marker into `colorCustomizations` that lists every key it owns. On the next apply, only those exact keys are removed before the new palette is written in. The user's own custom overrides are untouched because they are not in that list.

---

## Troubleshooting

**Colors are not applying**
1. Run `Matugen: Show Bridge Status` — check the output channel for errors
2. Confirm the palette file exists: `ls ~/.config/VSCodium/User/Theme/vscode-palette.json`
3. Verify the JSON is valid: `python3 -m json.tool < ~/.config/VSCodium/User/Theme/vscode-palette.json`
4. Make sure you have selected the **Matugen Dark** or **Matugen Light** theme

**Colors apply but look wrong**
- Check that your color values are valid hex (`#rrggbb` or `#rrggbbaa`)
- Non-hex values (e.g. `rgb(...)`, named colors) are skipped — the log will show which ones

**The watcher is not picking up changes**
- On some Linux setups, `inotify` limits can be hit. Increase them:
  ```bash
  echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf
  sudo sysctl -p
  ```
- You can always run `Matugen: Apply Palette Now` as a fallback

**I want to use VS Code, not VSCodium**
Set `matugenBridge.palettePath` to:
```
~/.config/Code/User/Theme/vscode-palette.json
```

---

## License

Dual License © 2026
