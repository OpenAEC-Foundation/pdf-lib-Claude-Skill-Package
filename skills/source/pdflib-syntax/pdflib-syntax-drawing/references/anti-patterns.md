# Drawing Anti-Patterns and Common Mistakes

## AP-1: Color Values 0-255 Instead of 0-1

**Severity:** CRITICAL — produces wrong colors silently (no error thrown)

```typescript
// WRONG — values MUST be 0.0-1.0
const red = rgb(255, 0, 0)          // NOT red — values overflow
const blue = rgb(0, 0, 255)         // NOT blue

// CORRECT
const red = rgb(1, 0, 0)
const blue = rgb(0, 0, 1)

// Converting from CSS 0-255 to pdf-lib 0-1:
const cssColor = rgb(51 / 255, 102 / 255, 204 / 255)  // CSS rgb(51,102,204)
```

**Why this happens:** Developers coming from HTML Canvas, CSS, or other libraries that use 0-255 ranges. pdf-lib uses the PDF specification's native 0.0-1.0 range for ALL color constructors (`rgb`, `cmyk`, `grayscale`).

---

## AP-2: Using width/height for drawEllipse

**Severity:** HIGH — produces ellipse with wrong dimensions

```typescript
// WRONG — drawEllipse uses xScale/yScale (radii), NOT width/height
page.drawEllipse({
  x: 200, y: 300,
  width: 100,    // IGNORED — not a valid option
  height: 50,    // IGNORED — not a valid option
})

// CORRECT — xScale and yScale define radii
page.drawEllipse({
  x: 200, y: 300,
  xScale: 100,   // horizontal radius: ellipse spans 200pt wide
  yScale: 50,    // vertical radius: ellipse spans 100pt tall
})
```

**NEVER** use `width`/`height` with `drawEllipse`. The options `xScale` and `yScale` define **radii** (half-width and half-height), measured from the center point (`x`, `y`).

---

## AP-3: Using borderWidth for drawLine

**Severity:** HIGH — line appears with default thickness instead of specified value

```typescript
// WRONG — drawLine does NOT use borderWidth
page.drawLine({
  start: { x: 50, y: 500 },
  end: { x: 400, y: 500 },
  borderWidth: 3,        // IGNORED
  borderColor: rgb(0,0,0), // IGNORED
})

// CORRECT — drawLine uses thickness, color, dashArray, lineCap
page.drawLine({
  start: { x: 50, y: 500 },
  end: { x: 400, y: 500 },
  thickness: 3,           // correct property
  color: rgb(0, 0, 0),    // correct property
})
```

**Full property name mapping for drawLine:**
| Other shapes | drawLine equivalent |
|-------------|-------------------|
| `borderWidth` | `thickness` |
| `borderColor` | `color` |
| `borderDashArray` | `dashArray` |
| `borderDashPhase` | `dashPhase` |
| `borderLineCap` | `lineCap` |

---

## AP-4: Using x/y for drawLine

**Severity:** HIGH — line draws at default position instead of intended position

```typescript
// WRONG — drawLine does NOT use x, y
page.drawLine({
  x: 50,           // IGNORED
  y: 500,          // IGNORED
  x2: 400,         // NOT a valid option
  y2: 500,         // NOT a valid option
  thickness: 2,
})

// CORRECT — use start and end point objects
page.drawLine({
  start: { x: 50, y: 500 },
  end: { x: 400, y: 500 },
  thickness: 2,
  color: rgb(0, 0, 0),
})
```

---

## AP-5: Confusing drawCircle `size` with Radius

**Severity:** MEDIUM — circle is twice as large or twice as small as intended

```typescript
// MISLEADING — `size` is DIAMETER, not radius
page.drawCircle({
  x: 200, y: 300,
  size: 50,        // This creates a circle with 50pt DIAMETER (25pt radius)
})

// If you want a circle with radius 50:
page.drawCircle({
  x: 200, y: 300,
  size: 100,       // 100pt diameter = 50pt radius
})
```

**Rule:** `drawCircle` `size` = diameter. Divide your desired radius by 2 if thinking in terms of diameter, or multiply by 2 if thinking in terms of radius.

---

## AP-6: Coordinate System Confusion (Y-axis Direction)

**Severity:** HIGH — shapes appear at bottom of page instead of top

```typescript
// WRONG — y=0 is BOTTOM of page, not top
page.drawRectangle({
  x: 50, y: 50,     // This draws near the BOTTOM
  width: 200, height: 100,
  color: rgb(1, 0, 0),
})

// CORRECT — to draw near the top, subtract from page height
const { height } = page.getSize()
page.drawRectangle({
  x: 50, y: height - 150,  // 150pt from top edge
  width: 200, height: 100,
  color: rgb(1, 0, 0),
})
```

**Rule:** PDF coordinates use bottom-left origin. Y increases UPWARD. ALWAYS use `page.getSize()` to calculate positions from the top.

---

## AP-7: Forgetting to Import Color/Rotation Constructors

**Severity:** HIGH — runtime error

```typescript
// WRONG — rgb is not imported
page.drawRectangle({
  color: rgb(1, 0, 0),    // ReferenceError: rgb is not defined
  rotate: degrees(45),     // ReferenceError: degrees is not defined
})

// CORRECT — ALWAYS import what you use
import { rgb, degrees, cmyk, grayscale, BlendMode, LineCapStyle } from 'pdf-lib'
```

---

## AP-8: Expecting drawSvgPath to Accept Full SVG Markup

**Severity:** HIGH — error or no output

```typescript
// WRONG — drawSvgPath only accepts path data strings
page.drawSvgPath('<svg><path d="M 0,0 L 100,100"/></svg>')
page.drawSvgPath('<path d="M 0,0 L 100,100"/>')

// CORRECT — pass only the d attribute content
page.drawSvgPath('M 0,0 L 100,100', {
  x: 50, y: 400,
  borderColor: rgb(0, 0, 0),
  borderWidth: 1,
})
```

**Rule:** `drawSvgPath` accepts ONLY the SVG path `d` attribute string, NOT full SVG or path element markup.

---

## AP-9: Omitting Fill or Stroke — Invisible Shape

**Severity:** MEDIUM — shape is drawn but invisible

```typescript
// PROBLEM — no color or borderColor specified
page.drawRectangle({
  x: 50, y: 400,
  width: 200, height: 100,
  // No color → no fill visible
  // No borderColor → no stroke visible
  // Shape exists but is invisible!
})

// CORRECT — ALWAYS specify at least one of color or borderColor
page.drawRectangle({
  x: 50, y: 400,
  width: 200, height: 100,
  color: rgb(0.9, 0.9, 0.9),       // visible fill
  borderColor: rgb(0, 0, 0),        // visible stroke
  borderWidth: 1,
})
```

**Rule:** ALWAYS provide at least `color` (fill) or `borderColor` + `borderWidth` (stroke) to make a shape visible.

---

## AP-10: Setting borderColor Without borderWidth

**Severity:** LOW — border color is set but border is invisible because width defaults to 0

```typescript
// PROBLEM — borderColor set but borderWidth is 0 (default)
page.drawRectangle({
  x: 50, y: 400,
  width: 200, height: 100,
  color: rgb(0.9, 0.9, 1),
  borderColor: rgb(0, 0, 0),       // specified but invisible!
  // borderWidth defaults to 0 → border not drawn
})

// CORRECT — ALWAYS pair borderColor with borderWidth
page.drawRectangle({
  x: 50, y: 400,
  width: 200, height: 100,
  color: rgb(0.9, 0.9, 1),
  borderColor: rgb(0, 0, 0),
  borderWidth: 1,                  // MUST be > 0 for border to appear
})
```

---

## AP-11: Using rotate with drawCircle

**Severity:** LOW — no error, but the option has no effect

```typescript
// UNNECESSARY — rotating a circle does nothing visible
page.drawCircle({
  x: 200, y: 300,
  size: 100,
  color: rgb(1, 0, 0),
  rotate: degrees(45),    // Has no visible effect on a circle
})
```

**Note:** `drawCircle` does not support the `rotate` option in its type definition. If passed, it is silently ignored since rotation has no visible effect on a circle.

---

## Summary: Property Name Differences by Shape

| Concept | Rectangle/Square/Circle/Ellipse/SVG | Line |
|---------|--------------------------------------|------|
| Position | `x`, `y` | `start: {x,y}`, `end: {x,y}` |
| Size | `width`/`height`, `size`, `xScale`/`yScale` | N/A |
| Stroke color | `borderColor` | `color` |
| Stroke width | `borderWidth` | `thickness` |
| Dash pattern | `borderDashArray` | `dashArray` |
| Dash offset | `borderDashPhase` | `dashPhase` |
| Line cap | `borderLineCap` | `lineCap` |
