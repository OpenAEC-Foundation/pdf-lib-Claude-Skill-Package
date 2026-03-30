---
name: pdflib-syntax-drawing
description: >
  Use when drawing shapes, lines, or SVG paths on PDF pages with pdf-lib.
  Prevents the common color value mistake: pdf-lib uses 0-1 range for rgb/cmyk,
  not 0-255. Covers drawRectangle, drawCircle, drawLine, drawSvgPath,
  color constructors (rgb, cmyk, grayscale), rotation helpers.
  Keywords: drawRectangle, drawCircle, drawLine, drawSvgPath, rgb, cmyk,
  grayscale, BlendMode, draw shapes, add line, draw on PDF, colors,
  rectangles circles, SVG path.
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# pdf-lib Drawing Operations

> Draw shapes, lines, and SVG paths on PDF pages with full control over colors, borders, opacity, and rotation.

## Critical Rules

- **Color values are 0.0-1.0, NOT 0-255.** `rgb(1, 0, 0)` is red. `rgb(255, 0, 0)` silently produces wrong output.
- **drawEllipse uses `xScale`/`yScale`, NOT `width`/`height`.** These define radii, not diameters.
- **drawLine uses `thickness`, NOT `borderWidth`.** It also uses `dashArray`/`dashPhase` (not `borderDashArray`/`borderDashPhase`).
- **drawLine uses `start`/`end` point objects, NOT `x`/`y`.**
- **drawCircle and drawSquare use `size`, NOT `width`/`height` or `radius`.**
- **Coordinate origin is bottom-left.** Y increases upward. To draw near the top, use `y: height - offset`.
- **All draw methods are synchronous.** No `await` needed on `page.drawRectangle()`, etc.
- **ALWAYS import color constructors explicitly:** `import { rgb, cmyk, grayscale } from 'pdf-lib'`

## Required Imports

```typescript
import {
  PDFDocument,
  rgb,           // rgb(r, g, b) — all 0.0-1.0
  cmyk,          // cmyk(c, m, y, k) — all 0.0-1.0
  grayscale,     // grayscale(g) — 0.0 black, 1.0 white
  degrees,       // degrees(45) → Rotation
  radians,       // radians(Math.PI / 4) → Rotation
  BlendMode,     // BlendMode.Normal, .Multiply, etc.
  LineCapStyle,  // LineCapStyle.Butt, .Round, .Square
} from 'pdf-lib'
```

## Shape Decision Tree

```
What shape do you need?
├── Filled/stroked box?
│   ├── Square (equal sides) → page.drawSquare({ size })
│   └── Rectangle → page.drawRectangle({ width, height })
├── Rounded/circular?
│   ├── Circle → page.drawCircle({ size })  // size = diameter
│   └── Ellipse → page.drawEllipse({ xScale, yScale })  // radii
├── Line between two points?
│   └── page.drawLine({ start: {x,y}, end: {x,y}, thickness })
└── Complex/custom shape?
    └── page.drawSvgPath('M 0,0 L 100,100 ...', { scale, color })
```

## Quick Reference: Color Constructors

| Constructor | Parameters | Range | Example |
|------------|-----------|-------|---------|
| `rgb(r, g, b)` | Red, Green, Blue | 0.0-1.0 each | `rgb(0, 0.53, 0.71)` — teal |
| `cmyk(c, m, y, k)` | Cyan, Magenta, Yellow, Key | 0.0-1.0 each | `cmyk(1, 0, 0, 0)` — cyan |
| `grayscale(g)` | Gray level | 0.0 (black) to 1.0 (white) | `grayscale(0.5)` — mid-gray |

**Color type:** `Color = Grayscale | RGB | CMYK`

### Common Colors

```typescript
const black   = rgb(0, 0, 0)
const white   = rgb(1, 1, 1)
const red     = rgb(1, 0, 0)
const green   = rgb(0, 1, 0)
const blue    = rgb(0, 0, 1)
const yellow  = rgb(1, 1, 0)
const gray50  = grayscale(0.5)
```

## Quick Reference: All Shape Methods

### drawRectangle

```typescript
page.drawRectangle({
  x: 50, y: 50,
  width: 200, height: 100,
  color: rgb(0.2, 0.4, 0.8),        // fill color
  borderColor: rgb(0, 0, 0),        // stroke color
  borderWidth: 2,                    // stroke thickness
  opacity: 0.9,                     // fill opacity 0.0-1.0
  borderOpacity: 1,                 // stroke opacity 0.0-1.0
  rotate: degrees(15),              // rotation
  borderDashArray: [6, 3],          // dashed border: 6pt dash, 3pt gap
  borderDashPhase: 0,               // dash offset
  borderLineCap: LineCapStyle.Round, // cap style
  blendMode: BlendMode.Normal,      // blending
})
```

### drawSquare

```typescript
page.drawSquare({
  x: 50, y: 50,
  size: 100,                        // side length (NOT width/height)
  color: rgb(1, 0.8, 0),
  borderColor: rgb(0, 0, 0),
  borderWidth: 1.5,
  rotate: degrees(45),
  opacity: 1,
  borderOpacity: 1,
  borderDashArray: [4, 2],
  borderDashPhase: 0,
  borderLineCap: LineCapStyle.Butt,
  blendMode: BlendMode.Normal,
})
```

### drawCircle

```typescript
page.drawCircle({
  x: 200, y: 300,                   // center point
  size: 80,                         // diameter (NOT radius)
  color: rgb(0.9, 0.1, 0.1),
  borderColor: rgb(0, 0, 0),
  borderWidth: 2,
  opacity: 0.7,
  borderOpacity: 1,
  borderDashArray: [5, 5],
  borderDashPhase: 0,
  borderLineCap: LineCapStyle.Round,
  blendMode: BlendMode.Normal,
})
```

### drawEllipse

```typescript
page.drawEllipse({
  x: 250, y: 400,                   // center point
  xScale: 120,                      // horizontal RADIUS (NOT width)
  yScale: 60,                       // vertical RADIUS (NOT height)
  color: rgb(0.3, 0.7, 0.3),
  borderColor: rgb(0, 0, 0),
  borderWidth: 1,
  rotate: degrees(30),
  opacity: 0.8,
  borderOpacity: 1,
  blendMode: BlendMode.Normal,
})
```

### drawLine

**CRITICAL:** drawLine uses DIFFERENT option names than other shapes.

```typescript
page.drawLine({
  start: { x: 50, y: 500 },         // start point object
  end: { x: 400, y: 500 },          // end point object
  thickness: 2,                      // NOT borderWidth
  color: rgb(0, 0, 0),              // line color (NOT borderColor)
  opacity: 1,
  dashArray: [10, 5],               // NOT borderDashArray
  dashPhase: 0,                     // NOT borderDashPhase
  lineCap: LineCapStyle.Butt,        // NOT borderLineCap
  blendMode: BlendMode.Normal,
})
```

### drawSvgPath

```typescript
page.drawSvgPath(
  'M 0,20 L 100,160 Q 130,200 150,120 C 190,-40 200,200 300,150 L 400,90',
  {
    x: 50, y: 700,                  // offset position
    color: rgb(0.8, 0.2, 0.2),      // fill color
    borderColor: rgb(0, 0, 0),      // stroke color
    borderWidth: 2,                  // stroke width
    opacity: 1,
    borderOpacity: 1,
    scale: 0.5,                     // scale factor
    rotate: degrees(0),
    blendMode: BlendMode.Normal,
  }
)
```

## Drawing Options Cross-Reference Table

| Option | Rectangle | Square | Circle | Ellipse | Line | SVG Path |
|--------|:---------:|:------:|:------:|:-------:|:----:|:--------:|
| `x`, `y` | YES | YES | YES (center) | YES (center) | NO | YES (offset) |
| `width`, `height` | YES | NO | NO | NO | NO | NO |
| `size` | NO | YES | YES (diameter) | NO | NO | NO |
| `xScale`, `yScale` | NO | NO | NO | YES (radii) | NO | NO |
| `start`, `end` | NO | NO | NO | NO | YES | NO |
| `color` | YES | YES | YES | YES | YES | YES |
| `opacity` | YES | YES | YES | YES | YES | YES |
| `borderColor` | YES | YES | YES | YES | NO | YES |
| `borderWidth` | YES | YES | YES | YES | NO | YES |
| `borderOpacity` | YES | YES | YES | YES | NO | YES |
| `thickness` | NO | NO | NO | NO | YES | NO |
| `rotate` | YES | YES | NO | YES | NO | YES |
| `scale` | NO | NO | NO | NO | NO | YES |
| `borderDashArray` | YES | YES | YES | NO | NO | NO |
| `borderDashPhase` | YES | YES | YES | NO | NO | NO |
| `borderLineCap` | YES | YES | YES | NO | NO | NO |
| `dashArray` | NO | NO | NO | NO | YES | NO |
| `dashPhase` | NO | NO | NO | NO | YES | NO |
| `lineCap` | NO | NO | NO | NO | YES | NO |
| `blendMode` | YES | YES | YES | YES | YES | YES |

## Enums Reference

### BlendMode

`BlendMode.Normal` | `Multiply` | `Screen` | `Overlay` | `Darken` | `Lighten` | `ColorDodge` | `ColorBurn` | `HardLight` | `SoftLight` | `Difference` | `Exclusion`

### LineCapStyle

| Value | Description |
|-------|------------|
| `LineCapStyle.Butt` | Flat edge at endpoint (default) |
| `LineCapStyle.Round` | Rounded cap extending past endpoint |
| `LineCapStyle.Square` | Square cap extending past endpoint |

### LineJoinStyle

| Value | Description |
|-------|------------|
| `LineJoinStyle.Miter` | Sharp corner (default) |
| `LineJoinStyle.Round` | Rounded corner |
| `LineJoinStyle.Bevel` | Flattened corner |

## Rotation

```typescript
import { degrees, radians } from 'pdf-lib'

// Use degrees() or radians() to create Rotation values
page.drawRectangle({ x: 100, y: 100, width: 80, height: 40, rotate: degrees(45) })
page.drawRectangle({ x: 300, y: 100, width: 80, height: 40, rotate: radians(Math.PI / 6) })
```

## SVG Path Commands Reference

| Command | Parameters | Description |
|---------|-----------|-------------|
| `M x,y` | Move to | Start a new sub-path |
| `L x,y` | Line to | Draw straight line |
| `H x` | Horizontal line | Draw horizontal line |
| `V y` | Vertical line | Draw vertical line |
| `C x1,y1 x2,y2 x,y` | Cubic bezier | Curve with two control points |
| `Q x1,y1 x,y` | Quadratic bezier | Curve with one control point |
| `A rx,ry rot large-arc sweep x,y` | Arc | Elliptical arc |
| `Z` | Close path | Close current sub-path |

Lowercase variants (m, l, h, v, c, q, a, z) use relative coordinates.

## Reference Links

- [Complete method signatures](references/methods.md)
- [Code examples](references/examples.md)
- [Anti-patterns and common mistakes](references/anti-patterns.md)
