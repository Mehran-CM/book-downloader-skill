---
name: book-downloader
description: |
  Download books as PDFs from Liber3 (liber3.eth.limo). Use this skill whenever the user asks to find, download, or get a book by title or author. Handles PDF downloads directly and converts EPUB files to PDF automatically using pdf2go.com. Trigger on any mention of downloading a book, getting a book, "find me [book title]", "download [book name]", or any request involving liber3.
---

# Book Downloader

Downloads books from liber3.eth.limo and delivers them as PDFs. If the best available format is EPUB, it converts to PDF via pdf2go.com before delivering.

This skill requires **Claude in Chrome** browser tools. If Chrome isn't connected, tell the user to open Chrome with the extension.

## Workflow

### Step 1: Search for the book

1. Get browser tab context (`tabs_context_mcp` with `createIfEmpty: true`)
2. Navigate to `https://liber3.eth.limo/`
3. Wait 3 seconds for the page to load
4. Find the search input (placeholder: "Search Book Title or Author's name")
5. Click it, type the book name, press Enter
6. Wait 5-8 seconds — this site is slow to return results. If you see a loading spinner, wait another 5 seconds.

### Step 2: Pick the best result

Use `get_page_text` to read all search results. Each result follows this pattern:

```
Title
Author / [Publisher /] Year / Language / format / Size / IPFS
```

**Selection priority** (auto-pick the best match):

1. **Language**: English only (unless user specifies otherwise)
2. **Format**: PDF first, then EPUB. Skip mobi, azw3, djvu, fb2, rar.
3. **Relevance**: Title must closely match what the user asked for. Skip summaries, workbooks, or unrelated titles.
4. **Recency**: Prefer newer editions (higher year)
5. **Size**: Prefer moderate sizes (2-15 MB). Very small files (<500 KB) are often summaries. Very large files (>50 MB) are often scanned copies with poor quality.

If no English PDF or EPUB exists, tell the user what's available and ask how to proceed.

### Step 3a: Download PDF (if format is PDF)

1. Find the "Download" element for the chosen result using `find` tool (search for "Download" buttons/links).
2. The Download buttons appear in order matching the search results. Count from the top to click the right one. The first Download corresponds to the first search result, second to second, etc.
3. Click the Download element. This opens a **new tab** with the PDF via an IPFS gateway.
4. The IPFS gateway is slow. Wait 10 seconds, then check the new tab. If still loading (blank page with progress bar), wait another 10 seconds. Repeat up to 3 times (total ~30 seconds for large files).
5. The new tab URL looks like: `https://gateway-ipfs.st/ipfs/{hash}?filename={BookName}.pdf`
6. Once the PDF loads in Chrome's viewer, the toolbar may be hidden. **Hover the mouse near the top of the page** to reveal it. The toolbar shows page numbers, zoom controls, and a download icon (↓) in the top-right corner.
7. Click the download icon (↓) in the top-right of the PDF viewer toolbar. This saves the file to the user's Downloads folder.
8. The file will be saved as `{BookName}.pdf` in `~/Downloads/`. Move it to the user's workspace `OUTPUTS/books/` folder.

**Important**: The IPFS gateway can be slow or fail. If the tab shows an error or stays blank after 30 seconds, go back to the search results and try the next best PDF result. The site sometimes shows a popup "Download failed? Try switching source." — dismiss it (click OK) and try a different result.

### Step 3b: Download EPUB and convert to PDF

If the best result is EPUB:

1. Click the Download element for the EPUB result. A new tab opens.
2. Wait 5 seconds. Copy the full URL from the new tab (visible in `tabs_context_mcp`).
3. Open a new tab and navigate to `https://www.pdf2go.com/epub-to-pdf`
4. Wait 3 seconds for the page to load.
5. Find the "URL" icon/label at the bottom of the red upload area (use `find` to search for "URL" element).
6. Click it. A URL input field appears with placeholder "http://www.example.com/examplefile.pdf".
7. Click the URL input field, paste the IPFS URL from step 2.
8. Click the "+ ADD" button.
9. Wait for the file to upload (watch for the file to appear in the upload area, may take 10-30 seconds depending on file size).
10. Click the "START" button.
11. Wait for conversion to complete. The page will redirect to a download page. This can take 15-60 seconds.
12. On the download page, click the "Download" button to get the converted PDF.
13. Save the file to the user's OUTPUTS folder.

### Step 4: Deliver the file

Save the PDF to `OUTPUTS/books/` in the user's workspace folder. Use the book title as the filename (e.g., `Atomic Habits - James Clear.pdf`).

Use `present_files` to share the file with the user.

## Troubleshooting

- **IPFS gateway timeout**: The gateway (`gateway-ipfs.st`) can be unreliable. If one download fails, try the next matching result from the search.
- **pdf2go upload fails**: The IPFS URL might be too long or the gateway might block external fetches. If URL upload fails, try downloading the EPUB file first and then uploading it directly via the file upload on pdf2go.
- **No results on Liber3**: Try variations of the book title (shorter query, author name only, removing subtitles).
- **Page not loading**: liber3.eth.limo uses IPFS/ENS and can be slow. Wait up to 15 seconds before retrying.

## Notes

- Liber3 is a decentralized book repository on IPFS. Download speeds vary.
- Always prefer PDF over EPUB to avoid the conversion step (faster, preserves original formatting).
- The site has a "File Type" dropdown filter at the top-right of search results — you can use this to filter by format if the results list is very long.
