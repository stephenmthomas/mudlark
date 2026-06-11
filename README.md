# Mudlark

**SVG to Interactive UI Framework**

Mudlark turns static SVG graphics into interactive, data-driven UI components — knobs, meters, sliders, toggles, and anything else you might imagine. Drop in any SVG, bind its attributes to controls, add event reactions, and export a self-contained JS factory with getters, setters, and drag input support.

> **mud·lark** /ˈmədˌlärk/ *noun* — a person who scavenges vector galleries for objects of value.

---

#### NO STRINGS ATTACHED

*Mudlark is a single HTML file with zero dependencies. No build step, no framework, no npm install. You simply open the file, turn an SVG into a functional UI control, generate the supporting code, and then use the UI control in your own projects.*

---

## What It Does

You find (or draw) an SVG of a knob, a meter, a button, a gauge — anything. Mudlark lets you:

1. **Inspect** every element and attribute in the SVG
2. **Bind** link SVG attributes like height, width to events or domains (sliders that drive rotation, position, opacity, color, stroke-dasharray, etc.)
3. **Add event reactions** (hover glow, click color change, mouseenter opacity shift — including SVG filter effects)
4. **Parameterize** your controls - so you can customize color, size, or other attributes before and during runtime.
5. **Export** a clean, standalone JS factory that creates instances of your component with a full getter/setter API

The output is code - HTML, CSS, and JS - based on your manipulations of the SVG file. Each exported component is a factory function: call it with a mount point, get back an object with named setters, getters, state, and drag input — ready to drop into any page.

---

## Quick Guide

1. **Open Mudlark** in a browser (it's a single HTML file)
2. **Load an SVG** — drag and drop onto the stage, or use File → Open SVG, or paste XML in the XML tab
3. **Select an element** — click it on the stage
4. **Create a control** — for moving UI elements like sliders or knobs, create a controlin INTERACT - CONTROLS section that links the domains of UI position to the size of the SVG.
5. **Add a reaction** — for non-moving and moving objects a like, add events like hover, active, drag, etc. and apply or mutate SVG attributes.
6. **Export** — go to the Export tab → there are a few modes but Multi Instance + Mapped is typically the best bet. If you have parameterized controls, check parameters.

---

## Interface Overview

### Left Panel — Four Tabs

#### ⚡ Interact
The main workspace. Two sub-tabs:

**Controls** — Primarily for UI elements that MOVE when interacted with, or due to input. Sliders, progress bars, knobs. Controls have a continuous range, and map that range to the size and attributes of the SVG.
- A slider with min/max/step range
- One or more *targets* (element + attribute bindings)
- Optional drag input (lets users drag an SVG element to control the value)
- Interpolation modes (linear, ease, snap, etc.)
- Template support for complex attributes (e.g. `rotate({v} 25 25)`)

**Events** — Event reactions tied to SVG elements - primarily used for static or toggled UI controls, although its styling uses can be for anything.:
- Pick a listener element and event type (click, hover, mouseenter, etc.)
- Add one or more reactions: set an attribute, toggle between values, apply a filter effect
- CSS pseudo-events (`:hover`, `:active`) generate real CSS rules inside the SVG
- JS events support set, toggle, temp (auto-restore), and class reactions
- Filter FX effects (glow, shadow, blur, grayscale, sepia, invert) with adjustable parameters

#### ◈ Element
Two sub-tabs:

**Element Inspector** — Shows the selected element's tag, ID, CSS selector, and all attributes. Edit any attribute inline. The ID field is highlighted in red as a reminder that renaming it cascades to all bindings and events.

**Path Inspector [BETA]** — For `<path>` elements:
- Analyze the `d=` attribute into structured commands
- View stats (command count, vertices, control points, subpaths)
- Render color-coded markers on the stage (purple squares = moveto, green circles = vertices, amber = arc endpoints, pink hollow = control points)
- Select individual commands and bind them to controls
- Shape detection identifies paths that are really circles, rectangles, or lines

#### ⟨/⟩ XML
Two sub-tabs:

**Source** — Live XML editor. Edit the SVG markup directly, then click Render to apply. Includes Format, Copy, and Minify buttons.

**Analysis** — Scans the SVG for issues and offers one-click refactors:
- Auto-name unnamed elements
- Round float values to 2 decimal places
- Remove empty groups and unused defs
- Convert ellipses-as-circles to `<circle>`
- Detect paths that are primitive shapes and convert them back
- Convert inline `style=""` to SVG presentation attributes
- Sort elements, add viewBox, add accessibility attributes

#### ⬡ Export
Four code output panes (HTML, CSS, JS, Full Page) with options:

**SVG Delivery:**
- *Single instance* — inline SVG paste, simple setter functions
- *Multi-instance* — JS factory pattern, create as many instances as needed

**Setter Type:**
- *Unmapped* — raw attribute values (set `rotation` to `45`)
- *Mapped* — normalized input domain → output range (set `rotation` to `0.5` and it maps to `0°–360°`)

**Parameterized Options** — If you've marked attributes as parameters, the factory accepts an `options` object so each instance can have different colors, sizes, etc.

### Stage (Center)
The SVG canvas with:
- Zoom in/out/fit/reset
- Checkerboard or grid background
- Click to select elements (shift+click for multi-select)
- Right-click context menu for quick actions
- Drag-and-drop SVG file loading

### Right Panels (Toggle with buttons in the toolbar)

**Edit Panel** — Visual editors for the selected element:
- Fill & Stroke (color pickers, opacity, palette)
- Gradient Fill (linear/radial, stop editor)
- Filters & Effects (glow, shadow, blur, grayscale, etc. with live parameter sliders)
- Transform & Scale (position, rotation, scale, flip)
- Vector & Path Tools (shape-to-path, reverse, close, join, split)
- Align, Distribute & Merge (alignment grid, boolean ops)
- Shapes & Structure (add elements, duplicate, wrap/unwrap groups, z-order)

Note: Many aspect of the edit panel are under development.

**Data Panel** — Live export preview sidebar showing the generated HTML and bindings JSON.

---

## Core Concepts

### Controls & Targets

A **control** is a named slider (e.g. "volume", "rotation", "fill-level"). It has a numeric range and drives one or more **targets**.

A **target** is a binding: element + attribute + output range + interpolation. When the control's slider moves, each target computes its output value and sets the attribute.

Example: A "level" control (0–1) with two targets:
- `#fillBar` → `stroke-dasharray` → 0 to 81.5
- `#needle` → `transform` → `rotate(-135 25 25)` to `rotate(135 25 25)`

Move one slider, both the fill bar and the needle respond.

### Templates

Some attributes need more than a bare number. The `{v}` template placeholder lets you embed the computed value inside a larger string:

- `rotate({v} 25 25)` — rotation around a center point
- `{v} 100` — stroke-dasharray with a fixed total length
- `translate({v}, 0)` — horizontal translation

### Interpolation

Controls support different interpolation curves between min and max:
- **linear** — straight mapping
- **ease** — ease-in-out curve
- **snap** — jumps between discrete values
- **color** — lerps between two hex colors

### Event Reactions

Events are separate from controls. Instead of continuous slider-driven values, they're triggered by user interaction:

- **set** — permanently change an attribute
- **toggle** — alternate between two values on each trigger
- **temp** — change on event, restore on the paired event (e.g. mouseenter/mouseleave)
- **class** — toggle a CSS class
- **Filter FX** — apply SVG filter effects (glow, shadow, blur) with customizable parameters

CSS pseudo-events (`:hover`, `:active`, `:focus`) generate real CSS rules injected into the SVG's `<style>` block — these work without JavaScript.

### Parameters

Parameters mark specific attributes as configurable per-instance. When exported with the "Parameterized" option, the factory function accepts an `options` object:

```js
const knob1 = createKnob(mount1, { color: '#ff0000', size: '48' });
const knob2 = createKnob(mount2, { color: '#00ff00', size: '64' });
```

### Drag Input

Any control can have a drag input: pick an SVG element, and dragging it up/down (or left/right) drives the control's value. The exported component includes pointer event handlers with capture, so drag interactions work reliably.

---

## Export Output

### Multi-instance / Mapped (Recommended)

The most capable export mode. Generates:

```js
// Factory function
function createMyComponent(container, options) {
  // Stamps the SVG
  // Resolves target elements
  // Returns { setRotation, getValue, getState, setAll, reset, onChange }
}

// Create instances
const inst1 = createMyComponent(document.getElementById('mount-1'));
const inst2 = createMyComponent(document.getElementById('mount-2'));

// Drive them independently
inst1.setRotation(0.75);  // input domain 0–1, maps to output range
inst2.setRotation(0.25);

// Read state
inst1.getValue();  // 0.75
inst1.getState();  // { rotation: { val: 0.75, min: 0, max: 1, step: 0.01 } }

// onChange callback for UI sync
inst1.onChange = (key, value) => {
  console.log(`${key} changed to ${value}`);
};
```

The Full Page output includes a complete demo page with:
- Per-instance preview columns with value readouts
- Individual sliders for each instance × control
- Per-instance parameter controls (color pickers, text inputs)
- Bidirectional sync: drag a knob → slider moves; move a slider → knob responds

### Single-instance / Unmapped

The simplest mode. Generates bare setter functions that directly set attribute values:

```js
function setRotation(value) {
  document.querySelector('#needle').setAttribute('transform', value);
}
```

---

## Workflow Examples

### Rotary Knob

1. Load an SVG with a circle (body) and a triangle/line (indicator)
2. Wrap the indicator in a `<g>` or nested `<svg>` if needed
3. Create a control named "rotation"
4. Add a reaction: target the indicator group → `transform` attribute → template `rotate({v} 25 25)` → range -135 to 135
5. Set up drag input: pick the circle body as the drag element
6. Export as Multi-instance / Mapped

### Progress Meter

1. Load an SVG with a background track circle and a foreground arc
2. Set `pathLength="100"` on the foreground circle and `stroke-dasharray="0 100"`
3. Create a control named "level"
4. Bind `stroke-dasharray` with template `{v} 100` and output range 0–100
5. Optionally bind a rotation on the indicator needle
6. Add a color parameter on the stroke color so each instance can be themed

### Toggle Button

1. Load an SVG with two states (e.g. a sliding toggle track + thumb)
2. Add an event reaction: `click` on the track → toggle the thumb's `cx` between two positions
3. Add another reaction on the same click: toggle the track's `fill` between active/inactive colors
4. Export — the component toggles on click with no additional JS needed

---

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| Click | Select element |
| Shift+Click | Multi-select |
| Right-click | Context menu |
| Drag & Drop | Load SVG file |

---

## File Menu

| Action | Description |
|--------|-------------|
| New SVG | Create a blank SVG canvas |
| Open SVG | Load an SVG file from disk |
| Save SVG | Download the current SVG |
| Open Project | Load a saved `.json` project (SVG + all bindings, events, params) |
| Save Project | Save everything as a `.json` project file |
| Export CSV | Export binding data as a CSV table |
| Demos | Load example SVGs with pre-configured bindings |

---

## Project Files

Mudlark saves and loads `.json` project files that contain:
- The SVG markup
- All control bindings (slider ranges, targets, templates, interpolation)
- All event reactions
- All parameter definitions
- Drag input configurations

This lets you save your work, share projects, and iterate across sessions.

---

## Tips

- **Name your elements.** Use Analysis → Auto-name to give every element an ID. Bindings and events reference elements by CSS selector — IDs are the most stable selectors.
- **Use the path inspector** for complex shapes. You can bind individual path commands to controls for morphing effects.
- **Start simple.** One control, one target. Get it working, then add complexity.
- **Use mapped setters** for reusable components. A 0–1 normalized input is easier to work with than raw pixel values.
- **Test with the Full Page export.** It generates a complete demo with sliders — if something doesn't work there, the binding is wrong.
- **Right-click is your friend.** The context menu has quick access to binding, cloning, visibility, and structure operations.
- **Filter FX on hover** are the quickest way to make SVGs feel interactive. Select an element → Events → `:hover` → add a Glow reaction → done.

---

## Tech Stack

Mudlark is a single HTML file with zero dependencies. No build step, no framework, no npm install. It runs entirely in the browser using:
- Vanilla JS (ES2020+)
- SVG DOM manipulation
- CSS custom properties for theming
- `localStorage` for preferences

The exported components are equally dependency-free — plain JS factories that work anywhere.

---

## License

Open source. See [LICENSE](LICENSE) for details.
