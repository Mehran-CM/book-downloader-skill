# book-downloader-skill

A Claude Cowork skill that downloads books as PDFs from [Liber3](https://liber3.eth.limo/).

## What it does

- Searches Liber3 for books by title or author
- Auto-picks the best result (English, PDF preferred, relevant title, recent edition)
- Downloads PDFs directly via IPFS gateway
- If only EPUB is available, converts to PDF using [pdf2go.com](https://www.pdf2go.com/epub-to-pdf)
     
## Requirements
     
  - **Claude Code** (or Claude Desktop with Cowork mode)
  - **Claude in Chrome extension** — the skill uses browser automation to navigate Liber3 and handle downloads. Install it from the Chrome Web Store.
         
      - Without the Chrome extension connected, this skill won't work.
         
## Installation
         
  - Download `book-downloader.skill` and install it in Claude Desktop via Settings > Skills, or place the `book-downloader/` folder in your Claude skills directory.
         
## Usage
         
  - Just tell Claude the book you want:
         
    ```
    Download "Atomic Habits" by James Clear
    ```

    ```
    Get me Creative Confidence by Tom Kelley
    ```

    The skill handles searching, picking the best match, downloading, and delivering the PDF.

## Notes
  - Liber3 runs on IPFS, so downloads can be slow (10-30 seconds for large files)
  - The skill prefers PDFs over EPUBs to skip the conversion step
  - If no English PDF/EPUB exists, it'll ask you what to do
