# pdflib-syntax-images — Anti-Patterns & Common Mistakes

## AP-1: Using Unsupported Image Formats

pdf-lib supports ONLY PNG and JPG. There is NO method for GIF, BMP, TIFF, WebP, SVG, AVIF, or HEIC.

```typescript
// BROKEN — these methods do NOT exist
const img = await pdfDoc.embedGif(gifBytes);    // NO such method
const img = await pdfDoc.embedBmp(bmpBytes);    // NO such method
const img = await pdfDoc.embedSvg(svgString);   // NO such method
const img = await pdfDoc.embedImage(anyBytes);   // NO such method
const img = await pdfDoc.embedWebp(webpBytes);   // NO such method

// CORRECT — convert to PNG or JPG first, then embed
// Use a canvas or image processing library (sharp, jimp, canvas) to convert
const pngBytes = await convertToPng(gifBytes); // your conversion logic
const image = await pdfDoc.embedPng(pngBytes);
```

## AP-2: Image Stretching / Aspect Ratio Distortion

Setting arbitrary width AND height values without calculating the correct
aspect ratio causes visual distortion.

```typescript
// BAD — image will be stretched/squished
page.drawImage(image, {
  x: 50, y: 50,
  width: 300,   // arbitrary
  height: 300,  // arbitrary — almost certainly wrong aspect ratio
});

// CORRECT — use scale() to preserve aspect ratio
const dims = image.scale(0.5);
page.drawImage(image, {
  x: 50, y: 50,
  width: dims.width,
  height: dims.height,
});

// CORRECT — use scaleToFit() to preserve aspect ratio
const dims = image.scaleToFit(300, 300);
page.drawImage(image, {
  x: 50, y: 50,
  width: dims.width,
  height: dims.height,
});

// CORRECT — manual calculation preserving ratio
const targetWidth = 300;
const scale = targetWidth / image.width;
page.drawImage(image, {
  x: 50, y: 50,
  width: targetWidth,
  height: image.height * scale,
});
```

## AP-3: Forgetting to Await Embed Methods

All image embedding methods are async. Forgetting `await` means you pass a
Promise instead of a PDFImage to `drawImage()`.

```typescript
// BROKEN — embedPng returns a Promise, not a PDFImage
const image = pdfDoc.embedPng(pngBytes);  // Missing await!
page.drawImage(image, { x: 50, y: 50 }); // TypeError: image is a Promise

// CORRECT
const image = await pdfDoc.embedPng(pngBytes);
page.drawImage(image, { x: 50, y: 50 });
```

## AP-4: Awaiting drawImage() or drawPage()

Drawing operations are synchronous. Adding `await` does not cause an error
but indicates a misunderstanding of the API.

```typescript
// UNNECESSARY — drawImage is synchronous, returns void
await page.drawImage(image, { x: 50, y: 50 });

// CORRECT
page.drawImage(image, { x: 50, y: 50 });

// SAME for drawPage
page.drawPage(embeddedPage, { x: 50, y: 50 }); // No await needed
```

## AP-5: Coordinate System Confusion (Y-axis)

PDF coordinates start at the bottom-left. Using top-left mental model causes
images to appear in unexpected positions.

```typescript
// WRONG mental model — image appears at bottom, not top
page.drawImage(image, { x: 50, y: 50 }); // This is 50pt from the BOTTOM

// CORRECT — position relative to page height for top placement
const { width: pageWidth, height: pageHeight } = page.getSize();
const dims = image.scale(0.5);
page.drawImage(image, {
  x: 50,
  y: pageHeight - dims.height - 50,  // 50pt from the TOP
  width: dims.width,
  height: dims.height,
});
```

## AP-6: Embedding the Same Image Multiple Times

Embedding the same image bytes multiple times wastes space. ALWAYS embed
once and reuse the PDFImage reference.

```typescript
// BAD — same image embedded 10 times, 10x the file size cost
for (let i = 0; i < 10; i++) {
  const image = await pdfDoc.embedPng(samePngBytes); // Redundant!
  const page = pdfDoc.addPage();
  page.drawImage(image, { x: 50, y: 50 });
}

// CORRECT — embed once, draw on every page
const image = await pdfDoc.embedPng(samePngBytes);
const dims = image.scale(0.5);
for (let i = 0; i < 10; i++) {
  const page = pdfDoc.addPage();
  page.drawImage(image, {
    x: 50, y: 50,
    width: dims.width,
    height: dims.height,
  });
}
```

## AP-7: Ignoring Original Dimensions

When no width/height options are given to `drawImage()`, the original pixel
dimensions are used as PDF points. A 3000x2000 pixel image will render as
3000x2000 points (41.7 x 27.8 inches), overflowing the page.

```typescript
// BAD — a high-resolution photo will be enormous
const image = await pdfDoc.embedJpg(highResPhotoBytes);
page.drawImage(image, { x: 0, y: 0 });
// A 4000x3000px image = 4000x3000pt = 55.6 x 41.7 inches!

// CORRECT — ALWAYS scale high-resolution images
const image = await pdfDoc.embedJpg(highResPhotoBytes);
const dims = image.scaleToFit(500, 700); // fit within reasonable bounds
page.drawImage(image, {
  x: 50, y: 50,
  width: dims.width,
  height: dims.height,
});
```

## AP-8: Confusing copyPages() with embedPdf()/embedPage()

`copyPages()` copies full pages for adding to a document. `embedPdf()`/`embedPage()`
creates a drawable snapshot for use with `drawPage()`. They serve different purposes.

```typescript
// WRONG — copyPages is for page-level operations, not drawing within a page
const [copiedPage] = await targetDoc.copyPages(sourceDoc, [0]);
page.drawPage(copiedPage); // copiedPage is a PDFPage, NOT a PDFEmbeddedPage

// CORRECT for drawing a page within another page
const [embeddedPage] = await targetDoc.embedPdf(sourcePdfBytes, [0]);
page.drawPage(embeddedPage, { x: 50, y: 50, width: 300, height: 400 });

// CORRECT for adding full pages to a document
const [copiedPage] = await targetDoc.copyPages(sourceDoc, [0]);
targetDoc.addPage(copiedPage);
```

## AP-9: Using scale() Return Value as a Modifier

`scale()` returns new dimension values — it does NOT modify the PDFImage object.

```typescript
// BROKEN — scale() does not modify the image
image.scale(0.5);
page.drawImage(image, { x: 50, y: 50 }); // Still draws at original size!

// CORRECT — use the returned dimensions
const dims = image.scale(0.5);
page.drawImage(image, {
  x: 50, y: 50,
  width: dims.width,
  height: dims.height,
});
```

## AP-10: Not Handling Embed Errors

Passing corrupt or invalid image data to embed methods will throw. ALWAYS
wrap embedding in try/catch when the image source is not fully trusted.

```typescript
// RISKY — no error handling for user-uploaded images
const image = await pdfDoc.embedPng(userUploadedBytes);

// CORRECT — handle potential failures
let image: PDFImage;
try {
  image = await pdfDoc.embedPng(userUploadedBytes);
} catch (error) {
  // Handle: corrupt file, wrong format, etc.
  throw new Error(`Failed to embed image: ${error.message}`);
}
```
