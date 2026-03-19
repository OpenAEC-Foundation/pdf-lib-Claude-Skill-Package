# Layout Patterns: Margins, Centering, Columns, Flow

## Pattern 1: Full Document Layout with Margins

A complete example showing margin management, header, body, and footer placement.

```typescript
import { PDFDocument, StandardFonts, rgb, PageSizes } from 'pdf-lib';

async function createDocumentWithLayout(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
  const boldFont = await pdfDoc.embedFont(StandardFonts.HelveticaBold);

  const page = pdfDoc.addPage(PageSizes.A4);
  const { width, height } = page.getSize();

  // Define margins (bottom-left origin — y=0 is BOTTOM of page)
  const margin = { top: 72, right: 72, bottom: 72, left: 72 };
  const contentWidth = width - margin.left - margin.right;

  // cursorY starts at the TOP of the content area
  let cursorY = height - margin.top;

  // === HEADER ===
  const title = 'Project Report';
  const titleSize = 24;
  const titleWidth = boldFont.widthOfTextAtSize(title, titleSize);
  page.drawText(title, {
    x: (width - titleWidth) / 2, // Horizontally centered
    y: cursorY,
    size: titleSize,
    font: boldFont,
    color: rgb(0, 0, 0),
  });
  cursorY -= 36;

  // Horizontal rule
  page.drawLine({
    start: { x: margin.left, y: cursorY },
    end: { x: width - margin.right, y: cursorY },
    thickness: 1,
    color: rgb(0.5, 0.5, 0.5),
  });
  cursorY -= 20;

  // === BODY ===
  const bodyText = 'This is the body content of the document. It demonstrates proper margin management and top-down content placement using the cursorY pattern.';
  page.drawText(bodyText, {
    x: margin.left,
    y: cursorY,
    size: 12,
    font,
    maxWidth: contentWidth,
    lineHeight: 16,
    color: rgb(0, 0, 0),
  });

  // Estimate wrapped text height to advance cursor
  const textWidth = font.widthOfTextAtSize(bodyText, 12);
  const estimatedLines = Math.ceil(textWidth / contentWidth);
  cursorY -= estimatedLines * 16;

  // === FOOTER ===
  const footerText = 'Page 1';
  const footerWidth = font.widthOfTextAtSize(footerText, 10);
  page.drawText(footerText, {
    x: (width - footerWidth) / 2,
    y: margin.bottom - 20, // Below content area, in bottom margin
    size: 10,
    font,
    color: rgb(0.5, 0.5, 0.5),
  });

  return pdfDoc.save();
}
```

## Pattern 2: Multi-Column Layout

```typescript
import { PDFDocument, StandardFonts, rgb, PageSizes } from 'pdf-lib';

async function createTwoColumnLayout(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica);

  const page = pdfDoc.addPage(PageSizes.A4);
  const { width, height } = page.getSize();

  const margin = 50;
  const gutter = 20; // Space between columns
  const columnCount = 2;
  const totalGutterWidth = gutter * (columnCount - 1);
  const columnWidth = (width - 2 * margin - totalGutterWidth) / columnCount;

  // Column X positions (bottom-left origin)
  const columns = [
    { x: margin, width: columnWidth },
    { x: margin + columnWidth + gutter, width: columnWidth },
  ];

  const fontSize = 10;
  const lineHeight = 14;

  // Sample data for each column
  const leftLines = [
    'Left column line 1',
    'Left column line 2',
    'Left column line 3',
    'Left column line 4',
  ];

  const rightLines = [
    'Right column line 1',
    'Right column line 2',
    'Right column line 3',
  ];

  // Draw left column (cursorY from top, bottom-left origin)
  let cursorY = height - margin;
  for (const line of leftLines) {
    page.drawText(line, {
      x: columns[0].x,
      y: cursorY,
      size: fontSize,
      font,
      maxWidth: columns[0].width,
      lineHeight,
    });
    cursorY -= lineHeight;
  }

  // Draw right column (reset cursorY to top)
  cursorY = height - margin;
  for (const line of rightLines) {
    page.drawText(line, {
      x: columns[1].x,
      y: cursorY,
      size: fontSize,
      font,
      maxWidth: columns[1].width,
      lineHeight,
    });
    cursorY -= lineHeight;
  }

  return pdfDoc.save();
}
```

## Pattern 3: Three-Column Layout Helper

```typescript
interface ColumnLayout {
  columns: Array<{ x: number; width: number }>;
  topY: number;
  bottomY: number;
}

function createColumnLayout(
  pageWidth: number,
  pageHeight: number,
  columnCount: number,
  margin: number,
  gutter: number,
): ColumnLayout {
  const totalGutterWidth = gutter * (columnCount - 1);
  const columnWidth = (pageWidth - 2 * margin - totalGutterWidth) / columnCount;

  const columns = Array.from({ length: columnCount }, (_, i) => ({
    x: margin + i * (columnWidth + gutter),
    width: columnWidth,
  }));

  return {
    columns,
    topY: pageHeight - margin, // Top of content (bottom-left origin!)
    bottomY: margin,           // Bottom of content
  };
}
```

## Pattern 4: Content Flow Across Multiple Pages

```typescript
import { PDFDocument, PDFFont, PDFPage, StandardFonts, PageSizes } from 'pdf-lib';

interface FlowState {
  page: PDFPage;
  cursorY: number;
}

async function createFlowingDocument(
  paragraphs: string[],
): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
  const boldFont = await pdfDoc.embedFont(StandardFonts.HelveticaBold);

  const margin = 50;
  const fontSize = 11;
  const lineHeight = 15;
  let pageNumber = 1;

  function addNewPage(): FlowState {
    const page = pdfDoc.addPage(PageSizes.A4);
    const { height } = page.getSize();
    return { page, cursorY: height - margin };
  }

  function addPageNumber(page: PDFPage, num: number): void {
    const { width } = page.getSize();
    const text = `Page ${num}`;
    const textWidth = font.widthOfTextAtSize(text, 9);
    page.drawText(text, {
      x: (width - textWidth) / 2,
      y: 25, // Near bottom of page (bottom-left origin)
      size: 9,
      font,
    });
  }

  let state = addNewPage();
  const contentWidth = state.page.getWidth() - 2 * margin;
  const bottomLimit = margin + 20; // Leave room for page number

  for (const paragraph of paragraphs) {
    // Estimate paragraph height
    const textWidth = font.widthOfTextAtSize(paragraph, fontSize);
    const estimatedLines = Math.max(1, Math.ceil(textWidth / contentWidth));
    const paragraphHeight = estimatedLines * lineHeight;

    // Check if paragraph fits on current page
    if (state.cursorY - paragraphHeight < bottomLimit) {
      addPageNumber(state.page, pageNumber);
      pageNumber++;
      state = addNewPage();
    }

    state.page.drawText(paragraph, {
      x: margin,
      y: state.cursorY,
      size: fontSize,
      font,
      maxWidth: contentWidth,
      lineHeight,
    });

    state.cursorY -= paragraphHeight + 8; // 8pt paragraph spacing
  }

  // Add page number to last page
  addPageNumber(state.page, pageNumber);

  return pdfDoc.save();
}
```

## Pattern 5: Centered Title Page

```typescript
import { PDFDocument, StandardFonts, rgb, PageSizes } from 'pdf-lib';

async function createTitlePage(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  const boldFont = await pdfDoc.embedFont(StandardFonts.HelveticaBold);
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica);

  const page = pdfDoc.addPage(PageSizes.A4);
  const { width, height } = page.getSize();

  // Main title — centered both horizontally and vertically
  const title = 'Annual Report 2025';
  const titleSize = 36;
  const titleWidth = boldFont.widthOfTextAtSize(title, titleSize);
  const titleHeight = boldFont.heightAtSize(titleSize);

  page.drawText(title, {
    x: (width - titleWidth) / 2,
    y: (height + titleHeight) / 2, // Center vertically (bottom-left origin)
    size: titleSize,
    font: boldFont,
    color: rgb(0.1, 0.1, 0.4),
  });

  // Subtitle — centered, below title
  const subtitle = 'Confidential Draft';
  const subtitleSize = 18;
  const subtitleWidth = font.widthOfTextAtSize(subtitle, subtitleSize);

  page.drawText(subtitle, {
    x: (width - subtitleWidth) / 2,
    y: (height + titleHeight) / 2 - 50, // 50pt below title
    size: subtitleSize,
    font,
    color: rgb(0.4, 0.4, 0.4),
  });

  return pdfDoc.save();
}
```

## Pattern 6: Table-Like Layout

```typescript
import { PDFDocument, PDFFont, PDFPage, StandardFonts, rgb, PageSizes } from 'pdf-lib';

function drawTableRow(
  page: PDFPage,
  y: number,
  columns: Array<{ x: number; width: number; text: string }>,
  font: PDFFont,
  fontSize: number,
): void {
  for (const col of columns) {
    page.drawText(col.text, {
      x: col.x + 4, // 4pt cell padding
      y: y + 4,
      size: fontSize,
      font,
      maxWidth: col.width - 8,
    });

    // Cell border (bottom-left origin — rectangle drawn from bottom-left corner)
    page.drawRectangle({
      x: col.x,
      y: y,
      width: col.width,
      height: 20,
      borderColor: rgb(0.7, 0.7, 0.7),
      borderWidth: 0.5,
    });
  }
}

async function createTableLayout(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
  const boldFont = await pdfDoc.embedFont(StandardFonts.HelveticaBold);

  const page = pdfDoc.addPage(PageSizes.A4);
  const { width, height } = page.getSize();
  const margin = 50;

  // Define column widths
  const tableColumns = [
    { x: margin, width: 150 },
    { x: margin + 150, width: 200 },
    { x: margin + 350, width: width - margin - 350 - margin },
  ];

  const rowHeight = 20;
  let cursorY = height - margin; // Start at top (bottom-left origin!)

  // Header row
  const headers = ['Name', 'Department', 'Status'];
  drawTableRow(
    page,
    cursorY - rowHeight,
    tableColumns.map((col, i) => ({ ...col, text: headers[i] })),
    boldFont,
    10,
  );
  cursorY -= rowHeight;

  // Data rows
  const data = [
    ['Alice', 'Engineering', 'Active'],
    ['Bob', 'Marketing', 'On Leave'],
    ['Carol', 'Finance', 'Active'],
  ];

  for (const row of data) {
    cursorY -= rowHeight;
    drawTableRow(
      page,
      cursorY,
      tableColumns.map((col, i) => ({ ...col, text: row[i] })),
      font,
      10,
    );
  }

  return pdfDoc.save();
}
```

## Pattern 7: Image with Caption

```typescript
// Place an image centered on the page with a caption below it
// All positions use bottom-left origin coordinates
const dims = image.scaleToFit(400, 300);
const imageX = (page.getWidth() - dims.width) / 2;
const imageY = cursorY - dims.height; // Image bottom edge (bottom-left origin)

page.drawImage(image, {
  x: imageX,
  y: imageY,
  width: dims.width,
  height: dims.height,
});

// Caption below image
const caption = 'Figure 1: System Architecture';
const captionWidth = font.widthOfTextAtSize(caption, 10);
page.drawText(caption, {
  x: (page.getWidth() - captionWidth) / 2,
  y: imageY - 16, // Below the image (bottom-left origin)
  size: 10,
  font: italicFont,
  color: rgb(0.4, 0.4, 0.4),
});

cursorY = imageY - 32; // Continue below caption
```
