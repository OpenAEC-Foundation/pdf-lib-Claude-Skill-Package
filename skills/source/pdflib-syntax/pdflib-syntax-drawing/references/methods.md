# Drawing Methods — Complete Signatures

## drawRectangle

```typescript
page.drawRectangle(options?: PDFPageDrawRectangleOptions): void
```

### PDFPageDrawRectangleOptions

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `x` | `number` | `0` | X coordinate (bottom-left corner of rectangle) |
| `y` | `number` | `0` | Y coordinate (bottom-left corner of rectangle) |
| `width` | `number` | `150` | Width in PDF points |
| `height` | `number` | `100` | Height in PDF points |
| `color` | `Color` | — | Fill color (`rgb()`, `cmyk()`, or `grayscale()`) |
| `borderColor` | `Color` | — | Stroke color |
| `borderWidth` | `number` | `0` | Stroke width in points |
| `opacity` | `number` | `1.0` | Fill opacity (0.0 transparent to 1.0 opaque) |
| `borderOpacity` | `number` | `1.0` | Stroke opacity (0.0-1.0) |
| `rotate` | `Rotation` | — | Rotation via `degrees()` or `radians()` |
| `blendMode` | `BlendMode` | — | Color blending mode |
| `borderDashArray` | `number[]` | — | Dash pattern, e.g. `[6, 3]` for 6pt dash, 3pt gap |
| `borderDashPhase` | `number` | `0` | Offset into dash pattern |
| `borderLineCap` | `LineCapStyle` | — | Line cap at dash ends |

---

## drawSquare

```typescript
page.drawSquare(options?: PDFPageDrawSquareOptions): void
```

### PDFPageDrawSquareOptions

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `x` | `number` | `0` | X coordinate (bottom-left corner) |
| `y` | `number` | `0` | Y coordinate (bottom-left corner) |
| `size` | `number` | `100` | Side length in PDF points |
| `color` | `Color` | — | Fill color |
| `borderColor` | `Color` | — | Stroke color |
| `borderWidth` | `number` | `0` | Stroke width |
| `opacity` | `number` | `1.0` | Fill opacity (0.0-1.0) |
| `borderOpacity` | `number` | `1.0` | Stroke opacity (0.0-1.0) |
| `rotate` | `Rotation` | — | Rotation |
| `blendMode` | `BlendMode` | — | Blending mode |
| `borderDashArray` | `number[]` | — | Dash pattern |
| `borderDashPhase` | `number` | `0` | Dash offset |
| `borderLineCap` | `LineCapStyle` | — | Line cap style |

---

## drawCircle

```typescript
page.drawCircle(options?: PDFPageDrawCircleOptions): void
```

### PDFPageDrawCircleOptions

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `x` | `number` | `0` | X coordinate of **center** |
| `y` | `number` | `0` | Y coordinate of **center** |
| `size` | `number` | `100` | **Diameter** of the circle (NOT radius) |
| `color` | `Color` | — | Fill color |
| `borderColor` | `Color` | — | Stroke color |
| `borderWidth` | `number` | `0` | Stroke width |
| `opacity` | `number` | `1.0` | Fill opacity (0.0-1.0) |
| `borderOpacity` | `number` | `1.0` | Stroke opacity (0.0-1.0) |
| `blendMode` | `BlendMode` | — | Blending mode |
| `borderDashArray` | `number[]` | — | Dash pattern |
| `borderDashPhase` | `number` | `0` | Dash offset |
| `borderLineCap` | `LineCapStyle` | — | Line cap style |

**Note:** drawCircle does NOT support `rotate` (a circle looks the same at any rotation).

---

## drawEllipse

```typescript
page.drawEllipse(options?: PDFPageDrawEllipseOptions): void
```

### PDFPageDrawEllipseOptions

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `x` | `number` | `0` | X coordinate of **center** |
| `y` | `number` | `0` | Y coordinate of **center** |
| `xScale` | `number` | `100` | Horizontal **radius** (NOT width/diameter) |
| `yScale` | `number` | `50` | Vertical **radius** (NOT height/diameter) |
| `color` | `Color` | — | Fill color |
| `borderColor` | `Color` | — | Stroke color |
| `borderWidth` | `number` | `0` | Stroke width |
| `opacity` | `number` | `1.0` | Fill opacity (0.0-1.0) |
| `borderOpacity` | `number` | `1.0` | Stroke opacity (0.0-1.0) |
| `rotate` | `Rotation` | — | Rotation |
| `blendMode` | `BlendMode` | — | Blending mode |

**CRITICAL:** `xScale` and `yScale` are **radii**, not diameters or full width/height. An ellipse with `xScale: 100` spans 200 points horizontally (100 points in each direction from center).

---

## drawLine

```typescript
page.drawLine(options: PDFPageDrawLineOptions): void
```

**Note:** The options parameter is REQUIRED (not optional like other draw methods).

### PDFPageDrawLineOptions

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `start` | `{ x: number, y: number }` | REQUIRED | Start point |
| `end` | `{ x: number, y: number }` | REQUIRED | End point |
| `thickness` | `number` | `1` | Line thickness (**NOT** `borderWidth`) |
| `color` | `Color` | — | Line color (**NOT** `borderColor`) |
| `opacity` | `number` | `1.0` | Opacity (0.0-1.0) |
| `dashArray` | `number[]` | — | Dash pattern (**NOT** `borderDashArray`) |
| `dashPhase` | `number` | `0` | Dash offset (**NOT** `borderDashPhase`) |
| `lineCap` | `LineCapStyle` | — | Line cap (**NOT** `borderLineCap`) |
| `blendMode` | `BlendMode` | — | Blending mode |

**CRITICAL:** drawLine uses completely different property names than the other shape methods. It does NOT use `x`, `y`, `borderColor`, `borderWidth`, `borderDashArray`, `borderDashPhase`, or `borderLineCap`.

---

## drawSvgPath

```typescript
page.drawSvgPath(path: string, options?: PDFPageDrawSVGOptions): void
```

### Parameters

- `path` — SVG path data string (e.g., `'M 0,0 L 100,100'`)

### PDFPageDrawSVGOptions

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `x` | `number` | `0` | X offset for the path |
| `y` | `number` | `0` | Y offset for the path |
| `color` | `Color` | — | Fill color |
| `borderColor` | `Color` | — | Stroke color |
| `borderWidth` | `number` | `0` | Stroke width |
| `opacity` | `number` | `1.0` | Fill opacity (0.0-1.0) |
| `borderOpacity` | `number` | `1.0` | Stroke opacity (0.0-1.0) |
| `scale` | `number` | — | Uniform scale factor |
| `rotate` | `Rotation` | — | Rotation |
| `blendMode` | `BlendMode` | — | Blending mode |

**Note:** `drawSvgPath` supports standard SVG path commands: M, L, H, V, C, S, Q, T, A, Z (and lowercase relative variants). It does NOT support full SVG markup — only the `d` attribute path data.

---

## Color Constructors

```typescript
function rgb(red: number, green: number, blue: number): RGB
function cmyk(cyan: number, magenta: number, yellow: number, key: number): CMYK
function grayscale(gray: number): Grayscale
```

ALL parameters MUST be in the range 0.0 to 1.0. Values outside this range produce undefined behavior.

**Type:** `Color = RGB | CMYK | Grayscale`

---

## Rotation Constructors

```typescript
function degrees(degreeAngle: number): Rotation
function radians(radianAngle: number): Rotation
```

---

## Enum Values

### BlendMode

```typescript
enum BlendMode {
  Normal,
  Multiply,
  Screen,
  Overlay,
  Darken,
  Lighten,
  ColorDodge,
  ColorBurn,
  HardLight,
  SoftLight,
  Difference,
  Exclusion,
}
```

### LineCapStyle

```typescript
enum LineCapStyle {
  Butt,    // Flat edge at endpoint
  Round,   // Semi-circle extending past endpoint
  Square,  // Half-square extending past endpoint
}
```

### LineJoinStyle

```typescript
enum LineJoinStyle {
  Miter,   // Sharp corner
  Round,   // Rounded corner
  Bevel,   // Flattened corner
}
```
