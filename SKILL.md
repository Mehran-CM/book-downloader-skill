---
name: book-downloader
description: |
  Download books as PDFs from Liber3 (liber3.eth.limo). Use this skill whenever the user asks to find, download, or get a book by title or author. Handles PDF downloads directly and converts EPUB files to PDF automatically using pdf2go.com. Trigger on any mention of downloading a book, getting a book, "find me [book title]", "download [book name]", or any request involving liber3.
---

# Book Downloader

Downloads books from liber3.eth.limo and delivers them as PDFs. If the best available format is EPUB, it converts to PDF via pdf2go.com before delivering.

This skill requires **Claude in Chrome** browser tools. If Chrome isn't connected, tell the user to open Chrome with the extension.

## Important: How Downloads Work on Liber3

Liber3 downloads files **directly to the user's Downloads folder** — it does NOT open a new browser tab. After clicking a Download button, you must poll the Downloads folder using `Glob` to detect when the file appears. The download starts immediately but may take 10-60 seconds depending on file size and IPFS gateway speed.

Before clicking Download, always request access to the user's Downloads folder (via `request_cowork_directory`) if you don't already have it. You'll need this to detect the downloaded file and to move it to the output location.

## Workflow

### Step 1: Ensure Downloads folder access

Request access to `~/Downloads` using `request_cowork_directory` if not already connected. You need this to detect downloaded files.

### Step 2: Search for the book

1. Get browser tab context (`tabs_context_mcp` with `createIfEmpty: true`)
2. Navigate to `https://liber3.eth.limo/`
3. Wait 3 seconds for the page to load
4. Find the search input (placeholder: "Search Book Title or Author's name")
5. Click it, type the book name, press Enter
6. Wait 5-8 seconds — this site is slow to return results. If you see a loading spinner, wait another 5 seconds.

### Step 3: Pick the best result

Use `get_page_text` to read all search results. Each result follows this pattern:

\`\`\`
Title
Author / [Publisher /] Year / Language / format / Size / IPFS
\`\`\`

**Selection priority** (auto-pick the best match):

1. **Language**: English only (unless user specifies otherwise)
2. **Format**: PDF first, then EPUB. Skip mobi, azw3, djvu, fb2, rar.
3. **Relevance**: Title must closely match what the user asked for. Skip summaries, workbooks, or unrelated titles.
4. **Recency**: Prefer newer editions (higher year)
5. **Size**: Prefer moderate sizes (2-15 MB). Very small files (<500 KB) are often summaries. Very large files (>50 MB) are often scanned copies with poor quality.

If no English PDF or EPUB exists, tell the user what's available and ask how to proceed.

### Step 4: Download the file

1. **Snapshot the Downloads folder** before clicking Download. Use `Glob` with pattern `*` in the Downloads folder to get a list of existing files. This lets you detect the new file by comparing before/after.
2. Find the "Download" element for the chosen result using `find` tool (search for "Download" buttons/links). The Download buttons appear in order matching the search results — first Download = first result, second = second, etc.
3. Click the Download element. The file will start downloading directly to the user's Downloads folder (no new tab opens).
4. **Poll for the downloaded file.** Use `Glob` with a pattern matching the book title (e.g., `*Ideaflow*`) in the Downloads folder. Wait 5 seconds between checks. The file may appear with a `.crdownload` extension while still downloading — keep waiting until the final `.pdf` or `.epub` extension appears. Repeat up to 6 times (total ~30 seconds). For large files (>10 MB), extend to 12 attempts (~60 seconds).
5. If a login modal appears instead of a download starting, dismiss it (click the X) and try a different Download button. Not all Download buttons require login — sometimes only the "Recommend" or other actions do. If the correct Download button keeps requiring login, tell the user.
6. If the file doesn't appear after polling, try the next best matching result from the search.

### Step 5: Convert EPUB to PDF (if needed)

If the downloaded file is EPUB:

1. Navigate to `https://www.pdf2go.com/epub-to-pdf` in the browser tab.
2. Wait 3 seconds for the page to load.
3. Find the file input element using `find` (search for "file input for uploading" or "Choose file").
4. Use `file_upload` to upload the EPUB file from the Downloads folder to the file input element.
5. Wait for the file to appear in the upload area (2-3 seconds).
6. Click the "START" button (the one inside the red upload area, or the blue one below it).
7. Wait for conversion to complete (10-60 seconds). The page will redirect to a download page showing the converted file.
8. Click the "Download" button on the results page.
9. Poll the Downloads folder for the new PDF file (same polling approach as Step 4).

### Step 6: Deliver the file

Move the final PDF to the user's workspace. Use the book title and author as the filename (e.g., `Ideaflow - Jeremy Utley, Perry Klebahn.pdf`).

Use `present_files` to share the file with the user.

## Troubleshooting

- **Login modal appears**: Some buttons on Liber3 require login. Dismiss the modal and try a different Download button, or try a different search result for the same book.
- **IPFS gateway timeout / file doesn't download**: The IPFS gateway can be unreliable. Try the next matching result from the search.
- **pdf2go upload fails**: If `file_upload` fails, try clicking the upload area directly and using the native file picker approach.
- **No results on Liber3**: Try variations of the book title (shorter query, author name only, removing subtitles).
- **Page not loading**: liber3.eth.limo uses IPFS/ENS and can be slow. Wait up to 15 seconds before retrying.
- **Duplicate files in Downloads**: Liber3 filenames include the book title and "_liber3" suffix. If there are duplicates (e.g., with "(1)" appended), use the most recent one.

## Notes

- Liber3 is a decentralized book repository on IPFS. Download speeds vary.
- Always prefer PDF over EPUB to avoid the conversion step (faster, preserves original formatting).
- The site has a "File Type" dropdown filter at the top-right of search results — you can use this to filter by format if the results list is very long.
