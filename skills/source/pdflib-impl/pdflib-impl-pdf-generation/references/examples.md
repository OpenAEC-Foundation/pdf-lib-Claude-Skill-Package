# Complete PDF Generation Examples

## Example 1: Invoice Template

A complete invoice with company header, customer details, line item table, and totals.

```typescript
import { PDFDocument, StandardFonts, rgb, PageSizes } from 'pdf-lib';

interface InvoiceLineItem {
  description: string;
  quantity: number;
  unitPrice: number;
}

interface InvoiceData {
  invoiceNumber: string;
  date: string;
  dueDate: string;
  companyName: string;
  companyAddress: string[];
  customerName: string;
  customerAddress: string[];
  items: InvoiceLineItem[];
  taxRate: number; // e.g., 0.21 for 21%
}

async function generateInvoice(data: InvoiceData): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
  const boldFont = await pdfDoc.embedFont(StandardFonts.HelveticaBold);
  const page = pdfDoc.addPage(PageSizes.A4);
  const { width, height } = page.getSize();

  const margin = 50;
  let y = height - margin;

  // --- Company Header ---
  page.drawText(data.companyName, {
    x: margin,
    y,
    size: 22,
    font: boldFont,
    color: rgb(0.1, 0.1, 0.4),
  });
  y -= 20;

  for (const line of data.companyAddress) {
    page.drawText(line, {
      x: margin,
      y,
      size: 9,
      font,
      color: rgb(0.4, 0.4, 0.4),
    });
    y -= 12;
  }

  // --- Invoice Title ---
  y -= 20;
  page.drawText('INVOICE', {
    x: margin,
    y,
    size: 28,
    font: boldFont,
    color: rgb(0, 0, 0),
  });

  // Invoice details (right-aligned)
  const detailX = width - margin - 200;
  page.drawText(`Invoice #: ${data.invoiceNumber}`, {
    x: detailX,
    y,
    size: 10,
    font,
    color: rgb(0, 0, 0),
  });
  y -= 14;
  page.drawText(`Date: ${data.date}`, {
    x: detailX,
    y,
    size: 10,
    font,
    color: rgb(0, 0, 0),
  });
  y -= 14;
  page.drawText(`Due: ${data.dueDate}`, {
    x: detailX,
    y,
    size: 10,
    font,
    color: rgb(0, 0, 0),
  });

  // --- Bill To ---
  y -= 20;
  page.drawText('Bill To:', {
    x: margin,
    y,
    size: 10,
    font: boldFont,
    color: rgb(0, 0, 0),
  });
  y -= 14;
  page.drawText(data.customerName, {
    x: margin,
    y,
    size: 10,
    font: boldFont,
    color: rgb(0, 0, 0),
  });
  y -= 14;
  for (const line of data.customerAddress) {
    page.drawText(line, { x: margin, y, size: 10, font, color: rgb(0, 0, 0) });
    y -= 14;
  }

  // --- Line Items Table ---
  y -= 20;
  const colWidths = [250, 60, 90, 95];
  const tableX = margin;
  const rowHeight = 22;
  const fontSize = 9;
  const totalTableWidth = colWidths.reduce((s, w) => s + w, 0);

  // Header row background
  page.drawRectangle({
    x: tableX,
    y: y - rowHeight,
    width: totalTableWidth,
    height: rowHeight,
    color: rgb(0.1, 0.1, 0.4),
  });

  // Header text
  const headers = ['Description', 'Qty', 'Unit Price', 'Amount'];
  let cellX = tableX;
  for (let i = 0; i < headers.length; i++) {
    page.drawText(headers[i], {
      x: cellX + 5,
      y: y - rowHeight + 7,
      size: fontSize,
      font: boldFont,
      color: rgb(1, 1, 1),
    });
    cellX += colWidths[i];
  }
  y -= rowHeight;

  // Data rows
  let subtotal = 0;
  for (let r = 0; r < data.items.length; r++) {
    const item = data.items[r];
    const amount = item.quantity * item.unitPrice;
    subtotal += amount;

    // Alternating row background
    if (r % 2 === 0) {
      page.drawRectangle({
        x: tableX,
        y: y - rowHeight,
        width: totalTableWidth,
        height: rowHeight,
        color: rgb(0.95, 0.95, 0.97),
      });
    }

    // Row border
    page.drawRectangle({
      x: tableX,
      y: y - rowHeight,
      width: totalTableWidth,
      height: rowHeight,
      borderColor: rgb(0.85, 0.85, 0.85),
      borderWidth: 0.5,
    });

    const rowValues = [
      item.description,
      item.quantity.toString(),
      item.unitPrice.toFixed(2),
      amount.toFixed(2),
    ];

    cellX = tableX;
    for (let c = 0; c < rowValues.length; c++) {
      let textX = cellX + 5;
      // Right-align numeric columns
      if (c >= 1) {
        const tw = font.widthOfTextAtSize(rowValues[c], fontSize);
        textX = cellX + colWidths[c] - tw - 5;
      }
      page.drawText(rowValues[c], {
        x: textX,
        y: y - rowHeight + 7,
        size: fontSize,
        font,
        color: rgb(0, 0, 0),
      });
      cellX += colWidths[c];
    }
    y -= rowHeight;
  }

  // --- Totals ---
  y -= 10;
  const totalsX = tableX + colWidths[0] + colWidths[1];
  const totalsValueX = totalsX + colWidths[2] + colWidths[3];
  const tax = subtotal * data.taxRate;
  const total = subtotal + tax;

  const totalsLines = [
    { label: 'Subtotal:', value: subtotal.toFixed(2) },
    { label: `Tax (${(data.taxRate * 100).toFixed(0)}%):`, value: tax.toFixed(2) },
    { label: 'Total:', value: total.toFixed(2), bold: true },
  ];

  for (const line of totalsLines) {
    const f = line.bold ? boldFont : font;
    const sz = line.bold ? 11 : fontSize;

    page.drawText(line.label, {
      x: totalsX,
      y,
      size: sz,
      font: f,
      color: rgb(0, 0, 0),
    });

    const valWidth = f.widthOfTextAtSize(line.value, sz);
    page.drawText(line.value, {
      x: totalsValueX - valWidth,
      y,
      size: sz,
      font: f,
      color: rgb(0, 0, 0),
    });
    y -= 16;
  }

  // --- Metadata ---
  pdfDoc.setTitle(`Invoice ${data.invoiceNumber}`, { showInWindowTitleBar: true });
  pdfDoc.setAuthor(data.companyName);
  pdfDoc.setCreator('Invoice Generator');
  pdfDoc.setCreationDate(new Date());

  return pdfDoc.save();
}
```

## Example 2: Multi-Page Report with Headers and Footers

A report that dynamically creates pages as content overflows, then stamps headers and footers.

```typescript
import { PDFDocument, StandardFonts, rgb, PageSizes, PDFFont, PDFPage } from 'pdf-lib';

interface ReportSection {
  title: string;
  paragraphs: string[];
}

async function generateReport(
  reportTitle: string,
  sections: ReportSection[],
): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
  const boldFont = await pdfDoc.embedFont(StandardFonts.HelveticaBold);

  const margin = 60;
  const headerSpace = 50; // Reserved for header
  const footerSpace = 60; // Reserved for footer
  const pageWidth = PageSizes.A4[0];
  const pageHeight = PageSizes.A4[1];
  const contentWidth = pageWidth - 2 * margin;
  const topY = pageHeight - margin - headerSpace;
  const bottomY = margin + footerSpace;

  let page = pdfDoc.addPage(PageSizes.A4);
  let currentY = topY;

  // Helper: check if we need a new page
  function ensureSpace(needed: number): void {
    if (currentY - needed < bottomY) {
      page = pdfDoc.addPage(PageSizes.A4);
      currentY = topY;
    }
  }

  // --- Title Page Content ---
  currentY -= 100;
  const titleWidth = boldFont.widthOfTextAtSize(reportTitle, 28);
  page.drawText(reportTitle, {
    x: (pageWidth - titleWidth) / 2,
    y: currentY,
    size: 28,
    font: boldFont,
    color: rgb(0.1, 0.1, 0.4),
  });
  currentY -= 40;

  const dateStr = new Date().toLocaleDateString('en-US', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  });
  const dateWidth = font.widthOfTextAtSize(dateStr, 12);
  page.drawText(dateStr, {
    x: (pageWidth - dateWidth) / 2,
    y: currentY,
    size: 12,
    font,
    color: rgb(0.4, 0.4, 0.4),
  });

  // Start content on new page
  page = pdfDoc.addPage(PageSizes.A4);
  currentY = topY;

  // --- Sections ---
  for (const section of sections) {
    // Section title needs at least 60pt of space
    ensureSpace(60);

    page.drawText(section.title, {
      x: margin,
      y: currentY,
      size: 16,
      font: boldFont,
      color: rgb(0.1, 0.1, 0.4),
    });
    currentY -= 8;

    // Underline
    page.drawLine({
      start: { x: margin, y: currentY },
      end: { x: margin + contentWidth, y: currentY },
      thickness: 1,
      color: rgb(0.1, 0.1, 0.4),
    });
    currentY -= 18;

    // Paragraphs
    for (const para of section.paragraphs) {
      // Estimate lines needed (rough: 80 chars per line at size 11)
      const estimatedLines = Math.ceil(para.length / 80);
      const spaceNeeded = estimatedLines * 16;
      ensureSpace(Math.min(spaceNeeded, 50)); // At least start on this page

      page.drawText(para, {
        x: margin,
        y: currentY,
        size: 11,
        font,
        color: rgb(0, 0, 0),
        maxWidth: contentWidth,
        lineHeight: 16,
      });

      // Approximate Y drop (drawText does NOT return the Y position after rendering)
      currentY -= spaceNeeded + 10;

      // Safety check after drawing
      if (currentY < bottomY) {
        page = pdfDoc.addPage(PageSizes.A4);
        currentY = topY;
      }
    }

    currentY -= 10; // Space between sections
  }

  // --- Add Headers and Footers to ALL pages ---
  const pages = pdfDoc.getPages();
  const totalPages = pages.length;

  for (let i = 0; i < totalPages; i++) {
    const p = pages[i];
    const { width, height } = p.getSize();

    // Header
    p.drawText(reportTitle, {
      x: margin,
      y: height - margin + 10,
      size: 9,
      font,
      color: rgb(0.5, 0.5, 0.5),
    });

    p.drawLine({
      start: { x: margin, y: height - margin },
      end: { x: width - margin, y: height - margin },
      thickness: 0.5,
      color: rgb(0.8, 0.8, 0.8),
    });

    // Footer
    p.drawLine({
      start: { x: margin, y: margin + footerSpace - 10 },
      end: { x: width - margin, y: margin + footerSpace - 10 },
      thickness: 0.5,
      color: rgb(0.8, 0.8, 0.8),
    });

    const pageNum = `Page ${i + 1} of ${totalPages}`;
    const pageNumWidth = font.widthOfTextAtSize(pageNum, 9);
    p.drawText(pageNum, {
      x: width - margin - pageNumWidth,
      y: margin + footerSpace - 25,
      size: 9,
      font,
      color: rgb(0.5, 0.5, 0.5),
    });
  }

  // --- Metadata ---
  pdfDoc.setTitle(reportTitle, { showInWindowTitleBar: true });
  pdfDoc.setCreator('Report Generator');
  pdfDoc.setCreationDate(new Date());

  return pdfDoc.save();
}
```

## Example 3: Document with Text, Images, and Shapes Combined

Demonstrates combining multiple content types on a single page.

```typescript
import { PDFDocument, StandardFonts, rgb, PageSizes } from 'pdf-lib';

async function generateProductSheet(
  productName: string,
  description: string,
  imageBytes: Uint8Array, // PNG or JPEG
  imageFormat: 'png' | 'jpg',
  specs: Array<{ label: string; value: string }>,
): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
  const boldFont = await pdfDoc.embedFont(StandardFonts.HelveticaBold);

  // Embed image ONCE before any page loop
  const image =
    imageFormat === 'png'
      ? await pdfDoc.embedPng(imageBytes)
      : await pdfDoc.embedJpg(imageBytes);

  const page = pdfDoc.addPage(PageSizes.A4);
  const { width, height } = page.getSize();
  const margin = 50;
  let y = height - margin;

  // --- Title bar (colored rectangle + white text) ---
  page.drawRectangle({
    x: 0,
    y: y - 40,
    width,
    height: 50,
    color: rgb(0.15, 0.15, 0.45),
  });

  const titleWidth = boldFont.widthOfTextAtSize(productName, 24);
  page.drawText(productName, {
    x: (width - titleWidth) / 2,
    y: y - 30,
    size: 24,
    font: boldFont,
    color: rgb(1, 1, 1),
  });
  y -= 70;

  // --- Product image (fit within 300x200, centered) ---
  const imgDims = image.scaleToFit(300, 200);
  const imgX = (width - imgDims.width) / 2;
  page.drawImage(image, {
    x: imgX,
    y: y - imgDims.height,
    width: imgDims.width,
    height: imgDims.height,
  });
  y -= imgDims.height + 20;

  // --- Description ---
  page.drawText('Description', {
    x: margin,
    y,
    size: 14,
    font: boldFont,
    color: rgb(0.15, 0.15, 0.45),
  });
  y -= 18;

  page.drawText(description, {
    x: margin,
    y,
    size: 10,
    font,
    color: rgb(0, 0, 0),
    maxWidth: width - 2 * margin,
    lineHeight: 14,
  });

  // Approximate Y drop for description
  const descLines = Math.ceil(description.length / 85);
  y -= descLines * 14 + 20;

  // --- Specifications table ---
  page.drawText('Specifications', {
    x: margin,
    y,
    size: 14,
    font: boldFont,
    color: rgb(0.15, 0.15, 0.45),
  });
  y -= 18;

  const labelWidth = 200;
  const valueWidth = width - 2 * margin - labelWidth;
  const rowH = 22;

  for (let i = 0; i < specs.length; i++) {
    // Alternating background
    if (i % 2 === 0) {
      page.drawRectangle({
        x: margin,
        y: y - rowH,
        width: width - 2 * margin,
        height: rowH,
        color: rgb(0.95, 0.95, 0.97),
      });
    }

    page.drawText(specs[i].label, {
      x: margin + 5,
      y: y - rowH + 7,
      size: 9,
      font: boldFont,
      color: rgb(0, 0, 0),
    });

    page.drawText(specs[i].value, {
      x: margin + labelWidth + 5,
      y: y - rowH + 7,
      size: 9,
      font,
      color: rgb(0, 0, 0),
    });

    y -= rowH;
  }

  pdfDoc.setTitle(`${productName} - Product Sheet`);
  pdfDoc.setCreationDate(new Date());

  return pdfDoc.save();
}
```

## Example 4: Saving and Outputting PDFs

### Node.js File Output

```typescript
import { writeFileSync } from 'fs';

const pdfBytes = await pdfDoc.save();
writeFileSync('output.pdf', pdfBytes);
```

### Express.js HTTP Response

```typescript
app.get('/invoice/:id', async (req, res) => {
  const pdfBytes = await generateInvoice(invoiceData);
  res.set({
    'Content-Type': 'application/pdf',
    'Content-Disposition': `attachment; filename="invoice-${req.params.id}.pdf"`,
    'Content-Length': pdfBytes.length,
  });
  res.send(Buffer.from(pdfBytes));
});
```

### Browser Display (iframe)

```typescript
const dataUri = await pdfDoc.saveAsBase64({ dataUri: true });
const iframe = document.getElementById('pdf-viewer') as HTMLIFrameElement;
iframe.src = dataUri;
```

### Browser Download (Blob)

```typescript
const pdfBytes = await pdfDoc.save();
const blob = new Blob([pdfBytes], { type: 'application/pdf' });
const url = URL.createObjectURL(blob);
const link = document.createElement('a');
link.href = url;
link.download = 'document.pdf';
link.click();
URL.revokeObjectURL(url);
```
