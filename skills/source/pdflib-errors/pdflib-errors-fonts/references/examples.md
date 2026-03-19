# Font Error Reproduction and Fix Examples

## Example 1: WinAnsi Encoding Error — Cyrillic Text

### Reproduce the error

```typescript
import { PDFDocument, StandardFonts } from 'pdf-lib';

async function brokenCyrillic(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
  const page = pdfDoc.addPage();

  // THROWS: "WinAnsi cannot encode" for Cyrillic characters
  page.drawText('Cyrillic text here', { font, size: 16 });

  return await pdfDoc.save();
}
```

**Error**: `WinAnsi cannot encode "X" (0x0422)` (or similar code point)

### Fix

```typescript
import { PDFDocument } from 'pdf-lib';
import fontkit from '@pdf-lib/fontkit';
import fs from 'fs';

async function fixedCyrillic(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  pdfDoc.registerFontkit(fontkit);

  // Use a font that supports Cyrillic (e.g., Noto Sans)
  const fontBytes = fs.readFileSync('fonts/NotoSans-Regular.ttf');
  const font = await pdfDoc.embedFont(fontBytes, { subset: true });

  const page = pdfDoc.addPage();
  page.drawText('Cyrillic text here', { font, size: 16 });

  return await pdfDoc.save();
}
```

## Example 2: Missing fontkit Registration

### Reproduce the error

```typescript
import { PDFDocument } from 'pdf-lib';
import fs from 'fs';

async function brokenFontkitMissing(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  // fontkit NOT registered

  const fontBytes = fs.readFileSync('fonts/CustomFont.ttf');
  // THROWS: No fontkit instance registered
  const font = await pdfDoc.embedFont(fontBytes);

  const page = pdfDoc.addPage();
  page.drawText('Hello', { font, size: 12 });
  return await pdfDoc.save();
}
```

### Fix

```typescript
import { PDFDocument } from 'pdf-lib';
import fontkit from '@pdf-lib/fontkit';
import fs from 'fs';

async function fixedFontkitRegistered(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  pdfDoc.registerFontkit(fontkit); // Register BEFORE embedFont

  const fontBytes = fs.readFileSync('fonts/CustomFont.ttf');
  const font = await pdfDoc.embedFont(fontBytes);

  const page = pdfDoc.addPage();
  page.drawText('Hello', { font, size: 12 });
  return await pdfDoc.save();
}
```

## Example 3: Form Field Font Not Applied

### Reproduce the error

```typescript
import { PDFDocument } from 'pdf-lib';

async function brokenFormFont(existingPdfBytes: Uint8Array): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.load(existingPdfBytes);
  const form = pdfDoc.getForm();

  // Sets text but default Helvetica cannot render non-Latin characters
  form.getTextField('client_name').setText('Non-Latin name');

  // save() calls updateFieldAppearances() with Helvetica by default
  // THROWS WinAnsi error or renders blank/garbage
  return await pdfDoc.save();
}
```

### Fix

```typescript
import { PDFDocument } from 'pdf-lib';
import fontkit from '@pdf-lib/fontkit';
import fs from 'fs';

async function fixedFormFont(existingPdfBytes: Uint8Array): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.load(existingPdfBytes);
  pdfDoc.registerFontkit(fontkit);

  const fontBytes = fs.readFileSync('fonts/NotoSans-Regular.ttf');
  const customFont = await pdfDoc.embedFont(fontBytes, { subset: true });

  const form = pdfDoc.getForm();
  form.getTextField('client_name').setText('Non-Latin name');

  // Update appearances with the custom font
  form.updateFieldAppearances(customFont);

  // Skip automatic appearance update since we did it manually
  return await pdfDoc.save({ updateFieldAppearances: false });
}
```

## Example 4: Diacritics with Standard Fonts — Partial Failure

### Reproduce the error

```typescript
import { PDFDocument, StandardFonts } from 'pdf-lib';

async function brokenDiacritics(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
  const page = pdfDoc.addPage();

  // This MAY work (common WinAnsi diacritics)
  page.drawText('cafe, naive', { font, size: 12, y: 700 });

  // This WILL FAIL (characters outside WinAnsi)
  page.drawText('Text with unsupported diacritics', { font, size: 12, y: 680 });

  return await pdfDoc.save();
}
```

### Fix

```typescript
import { PDFDocument } from 'pdf-lib';
import fontkit from '@pdf-lib/fontkit';
import fs from 'fs';

async function fixedDiacritics(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  pdfDoc.registerFontkit(fontkit);

  const fontBytes = fs.readFileSync('fonts/NotoSans-Regular.ttf');
  const font = await pdfDoc.embedFont(fontBytes, { subset: true });
  const page = pdfDoc.addPage();

  // ALL diacritics work with custom font (if font contains the glyphs)
  page.drawText('cafe, naive, Munchen, Malmo', { font, size: 12, y: 700 });

  return await pdfDoc.save();
}
```

## Example 5: Character Support Pre-Check

### Proactive error prevention

```typescript
import { PDFDocument, StandardFonts } from 'pdf-lib';
import fontkit from '@pdf-lib/fontkit';
import fs from 'fs';

async function safeTextRendering(text: string): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();

  // Try standard font first, fall back to custom if needed
  const standardFont = pdfDoc.embedStandardFont(StandardFonts.Helvetica);
  const charSet = new Set(standardFont.getCharacterSet());

  let font = standardFont;
  let needsCustomFont = false;

  for (let i = 0; i < text.length; i++) {
    if (!charSet.has(text.charCodeAt(i))) {
      needsCustomFont = true;
      break;
    }
  }

  if (needsCustomFont) {
    pdfDoc.registerFontkit(fontkit);
    const fontBytes = fs.readFileSync('fonts/NotoSans-Regular.ttf');
    font = await pdfDoc.embedFont(fontBytes, { subset: true });
  }

  const page = pdfDoc.addPage();
  page.drawText(text, { font, size: 12, x: 50, y: 700 });

  return await pdfDoc.save();
}
```

## Example 6: Multiple Fonts for Mixed Content

### Using different fonts for different text segments

```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib';
import fontkit from '@pdf-lib/fontkit';
import fs from 'fs';

async function mixedFontDocument(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  pdfDoc.registerFontkit(fontkit);

  // Standard font for ASCII-only content (no fontkit needed for this)
  const helvetica = pdfDoc.embedStandardFont(StandardFonts.Helvetica);

  // Custom font for Unicode content
  const fontBytes = fs.readFileSync('fonts/NotoSans-Regular.ttf');
  const notoSans = await pdfDoc.embedFont(fontBytes, { subset: true });

  const page = pdfDoc.addPage();
  const { height } = page.getSize();

  // ASCII header — standard font is fine
  page.drawText('Document Title', {
    font: helvetica,
    size: 24,
    x: 50,
    y: height - 60,
    color: rgb(0, 0, 0),
  });

  // Unicode body — MUST use custom font
  page.drawText('International content here', {
    font: notoSans,
    size: 12,
    x: 50,
    y: height - 100,
    color: rgb(0, 0, 0),
  });

  return await pdfDoc.save();
}
```

## Example 7: Form Fields with Custom Font (Complete Workflow)

### End-to-end form filling with Unicode support

```typescript
import { PDFDocument } from 'pdf-lib';
import fontkit from '@pdf-lib/fontkit';
import fs from 'fs';

async function fillFormWithUnicode(
  templateBytes: Uint8Array,
  fieldData: Record<string, string>
): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.load(templateBytes);
  pdfDoc.registerFontkit(fontkit);

  const fontBytes = fs.readFileSync('fonts/NotoSans-Regular.ttf');
  const customFont = await pdfDoc.embedFont(fontBytes, { subset: true });

  const form = pdfDoc.getForm();

  // Fill all fields
  for (const [fieldName, value] of Object.entries(fieldData)) {
    try {
      const field = form.getTextField(fieldName);
      field.setText(value);
    } catch (error) {
      console.error(`Field "${fieldName}" not found or is not a text field`);
    }
  }

  // CRITICAL: Update appearances with custom font AFTER all setText() calls
  form.updateFieldAppearances(customFont);

  // Save without automatic appearance update (we already did it)
  return await pdfDoc.save({ updateFieldAppearances: false });
}
```

## Example 8: Save Triggers WinAnsi Error on Form Fields

### Understanding the save() pitfall

```typescript
import { PDFDocument } from 'pdf-lib';

async function brokenSave(existingPdfBytes: Uint8Array): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.load(existingPdfBytes);
  const form = pdfDoc.getForm();

  form.getTextField('notes').setText('Text with special chars');

  // save() with default options calls updateFieldAppearances() internally
  // This uses Helvetica, which THROWS for non-WinAnsi characters
  return await pdfDoc.save(); // CRASHES HERE
}
```

### Fix: Either provide custom font or skip appearance update

```typescript
import { PDFDocument } from 'pdf-lib';
import fontkit from '@pdf-lib/fontkit';

async function fixedSave(existingPdfBytes: Uint8Array): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.load(existingPdfBytes);
  pdfDoc.registerFontkit(fontkit);

  const fontBytes = fs.readFileSync('fonts/NotoSans-Regular.ttf');
  const customFont = await pdfDoc.embedFont(fontBytes, { subset: true });

  const form = pdfDoc.getForm();
  form.getTextField('notes').setText('Text with special chars');
  form.updateFieldAppearances(customFont);

  return await pdfDoc.save({ updateFieldAppearances: false });
}
```
