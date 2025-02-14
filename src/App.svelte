<script>
  import { PDFDocument } from 'pdf-lib';
  
  let templatePdfFile;
  let contentPdfFile;
  let mergedPdfUrl = null;

  async function mergePdfs() {
    if (!templatePdfFile || !contentPdfFile) {
      alert("Please select both Template and Content PDFs.");
      return;
    }

    // Read files as ArrayBuffers
    const templateArrayBuffer = await templatePdfFile.arrayBuffer();
    const contentArrayBuffer = await contentPdfFile.arrayBuffer();

    // Load PDFs
    const templatePdf = await PDFDocument.load(templateArrayBuffer);
    const contentPdf = await PDFDocument.load(contentArrayBuffer);

    // Ensure template has enough pages
    if (templatePdf.getPageCount() < contentPdf.getPageCount()) {
      alert("Template PDF has fewer pages than Content PDF!");
      return;
    }

    // Merge PDFs
    for (let i = 0; i < contentPdf.getPageCount(); i++) {
      const templatePage = templatePdf.getPage(i);

      // ✅ Embed the content page instead of copying
      const embeddedPage = await templatePdf.embedPage(contentPdf.getPage(i));

      // Draw the embedded content page onto the template page
      templatePage.drawPage(embeddedPage, {
        x: 0,
        y: 0,
        width: templatePage.getWidth(),
        height: templatePage.getHeight(),
      });
    }

    // Save merged PDF
    const mergedPdfBytes = await templatePdf.save();
    const blob = new Blob([mergedPdfBytes], { type: "application/pdf" });
    mergedPdfUrl = URL.createObjectURL(blob);
  }
</script>

<main>
  <h2>PDF Merger</h2>

  <label>
    Template PDF:
    <input type="file" accept="application/pdf" on:change="{e => templatePdfFile = e.target.files[0]}" />
  </label>

  <label>
    Content PDF:
    <input type="file" accept="application/pdf" on:change="{e => contentPdfFile = e.target.files[0]}" />
  </label>

  <button on:click="{mergePdfs}">Merge PDFs</button>

  {#if mergedPdfUrl}
    <p><a href="{mergedPdfUrl}" download="merged_output.pdf">Download Merged PDF</a></p>
  {/if}
</main>
