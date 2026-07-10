# Textmill

A visual PDF splitter for big documents. Open a book-length PDF, see every page as a thumbnail, group pages into named *parts* (a chapter, an article, a reading), and export each part as its own PDF — or one combined PDF in the order you choose. Built for carving scholarly PDFs into course readings, but it works on anything.

**Use it:** open `textmill.html` in Chrome or Edge — it's a single self-contained file, no install, no build step. Drop a PDF in and click pages. A short chain of hint bubbles walks you through the rest the first time.

## Features

Pages can belong to several parts at once, and parts don't need to be contiguous. There's a built-in scrolling reader (double-click any page), page-level text search, printed-page-number labels when the PDF provides them, and a From/To file panel for browsing source folders and choosing where exports land.

## Requirements & privacy

Works best in Chrome or Edge; the folder features use the File System Access API, which Safari and Firefox don't support (exports fall back to your Downloads folder there). Everything runs in your browser — no document data ever leaves your machine. The only network access is loading two libraries (pdf.js and pdf-lib) from a CDN.

## Provenance

This app is entirely vibe-coded: the code was written almost entirely by Claude (Anthropic), directed, designed, and tested by a human. Bugs in judgment are his; bugs in JavaScript are a collaboration. Notes welcome: use **Leave me a note** under Help in the app, or [email](mailto:erickraphael@gmail.com).
