# Coordinate Mistakes and Layout Anti-Patterns

## Anti-Pattern 1: Assuming Top-Left Origin

**THE most common layout mistake.** PDF coordinates use bottom-left origin. Y=0 is the BOTTOM of the page, not the top.

```typescript
// WRONG — text appears at the bottom of the page, not the top
page.drawText('Title', { x: 50, y: 50 });

// CORRECT — subtract from height to position near the top
const { height } = page.getSize();
page.drawText('Title', { x: 50, y: height - 50 });
```

ALWAYS get page dimensions with `page.getSize()` and use `height - offset` for top-down positioning.

---

## Anti-Pattern 2: Incrementing cursorY to Move Down

In bottom-left origin, moving "down" the page means DECREASING Y, not increasing it.

```typescript
// WRONG — this moves UP the page
let cursorY = 700;
cursorY += 20; // Now at 720, which is HIGHER on the page

// CORRECT — subtract to move down
let cursorY = height - 50;
cursorY -= 20; // Now 20 points lower on the page
```

NEVER add to cursorY when intending to move down the page. ALWAYS subtract.

---

## Anti-Pattern 3: Hardcoding Y Positions

```typescript
// WRONG — breaks on different page sizes
page.drawText('Header', { x: 50, y: 750 });
page.drawText('Footer', { x: 50, y: 30 });

// CORRECT — calculate from page dimensions
const { width, height } = page.getSize();
page.drawText('Header', { x: 50, y: height - 42 });
page.drawText('Footer', { x: 50, y: 30 }); // 30 from bottom is OK
```

ALWAYS calculate positions relative to `page.getSize()` for anything positioned from the top. Positions from the bottom (footers) can use absolute values.

---

## Anti-Pattern 4: Forgetting to Advance cursorY After Wrapped Text

When using `maxWidth`, text wraps to multiple lines. You MUST estimate the wrapped height and advance cursorY accordingly.

```typescript
// WRONG — cursorY only moves by one line, but text may wrap to 3+ lines
page.drawText(longText, { x: 50, y: cursorY, maxWidth: 400, lineHeight: 16 });
cursorY -= 16; // Only accounts for 1 line!

// CORRECT — estimate wrapped height
const textWidth = font.widthOfTextAtSize(longText, fontSize);
const maxWidth = 400;
const lineCount = Math.ceil(textWidth / maxWidth);
page.drawText(longText, { x: 50, y: cursorY, maxWidth, lineHeight: 16 });
cursorY -= lineCount * 16; // Accounts for all wrapped lines
```

---

## Anti-Pattern 5: Drawing Below the Bottom Margin

```typescript
// WRONG — no overflow check, content disappears below page
for (const line of lines) {
  page.drawText(line, { x: 50, y: cursorY });
  cursorY -= 16;
  // cursorY could go negative — text is lost!
}

// CORRECT — check for overflow before drawing
for (const line of lines) {
  if (cursorY - 16 < bottomMargin) {
    page = pdfDoc.addPage(PageSizes.A4);
    cursorY = page.getHeight() - topMargin;
  }
  page.drawText(line, { x: 50, y: cursorY });
  cursorY -= 16;
}
```

NEVER draw content without checking if it fits above the bottom margin. ALWAYS add a new page when content would overflow.

---

## Anti-Pattern 6: Estimating Text Width Instead of Measuring

```typescript
// WRONG — rough estimate, inaccurate for centering
const estimatedWidth = text.length * 7; // Unreliable!
const x = (pageWidth - estimatedWidth) / 2;

// CORRECT — use font measurement method
const textWidth = font.widthOfTextAtSize(text, fontSize);
const x = (pageWidth - textWidth) / 2;
```

ALWAYS use `font.widthOfTextAtSize()` for accurate text width measurement. Character widths vary between fonts and even between characters in the same font.

---

## Anti-Pattern 7: Ignoring Margins

```typescript
// WRONG — content touches the edge of the page
page.drawText('Edge text', { x: 0, y: 0 });

// CORRECT — use margins
const margin = 50;
page.drawText('Margin text', { x: margin, y: margin });
```

ALWAYS define and apply margins. Content at the page edge may be cut off during printing.

---

## Anti-Pattern 8: Using page.moveTo() for Document Layout

```typescript
// WRONG — page.moveTo() is for SVG path drawing, not general layout
page.moveTo(50, 700);
page.drawText('Hello'); // drawText ignores moveTo position!

// CORRECT — pass x/y directly to drawText
page.drawText('Hello', { x: 50, y: 700 });
```

The `page.moveTo()`, `page.moveUp()`, `page.moveDown()` methods control an internal cursor primarily used by `drawSvgPath()`. NEVER use them for positioning text, images, or shapes. ALWAYS pass explicit `x` and `y` to drawing methods.

---

## Anti-Pattern 9: Not Accounting for lineHeight in Content Area

```typescript
// WRONG — only checks if cursorY is below margin, not cursorY MINUS lineHeight
if (cursorY < bottomMargin) { /* new page */ }

// CORRECT — check if the NEXT line would go below the margin
if (cursorY - lineHeight < bottomMargin) { /* new page */ }
```

ALWAYS check `cursorY - lineHeight` (not just `cursorY`) against the bottom margin. The text baseline is at cursorY, but the text extends below it.

---

## Anti-Pattern 10: Mixing Up drawRectangle Origin

`drawRectangle()` draws from the bottom-left corner of the rectangle, consistent with PDF's bottom-left origin.

```typescript
// The rectangle starts at (x, y) = its BOTTOM-LEFT corner
page.drawRectangle({
  x: 50,      // Left edge
  y: 100,     // BOTTOM edge (not top!)
  width: 200,
  height: 50, // Extends UPWARD from y=100 to y=150
});
```

NEVER assume the y position is the top edge of a rectangle. It is ALWAYS the bottom edge.

---

## Anti-Pattern 11: Forgetting Page Dimensions Change Per Page

```typescript
// WRONG — assumes all pages have the same size
const { width, height } = pdfDoc.getPage(0).getSize();
// ... later, working with page at index 3
page3.drawText('Text', { y: height - 50 }); // Wrong if page3 has different size!

// CORRECT — get dimensions for each page
const { width, height } = page3.getSize();
page3.drawText('Text', { y: height - 50 });
```

ALWAYS call `page.getSize()` on the specific page you are drawing on. Different pages in the same document can have different dimensions.

---

## Summary Table

| Anti-Pattern | Fix |
|-------------|-----|
| Assuming top-left origin | Use `height - offset` for top positions |
| Incrementing cursorY to move down | Subtract from cursorY |
| Hardcoding Y positions | Calculate from `page.getSize()` |
| Not advancing cursorY for wrapped text | Estimate line count and multiply by lineHeight |
| Drawing below bottom margin | Check overflow before drawing, add new page |
| Estimating text width | Use `font.widthOfTextAtSize()` |
| Ignoring margins | ALWAYS define and enforce margins |
| Using moveTo() for layout | Pass x/y directly to draw methods |
| Not checking cursorY - lineHeight | Check `cursorY - lineHeight < bottomMargin` |
| Wrong rectangle origin assumption | `y` is the BOTTOM edge, not top |
| Assuming same page dimensions | Call `getSize()` per page |
