# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-file web app (`index.html`) for nurses: a Japanese-language "情報収集ワークシート" (patient information worksheet). The user photographs a paper chart (カルテ) with their phone camera; the app runs OCR on-device via Tesseract.js, parses out patient fields (name, age, sex, medical history, ADL, treatment), and renders editable patient cards that can be exported as text to the clipboard.

There is no build system, package manager, linter, or test suite. The entire app — CSS, markup, and JavaScript — lives in `index.html`.

## Development

- Run locally with any static server, e.g. `python3 -m http.server`, then open `http://localhost:8000`. Opening the file directly also works, but camera capture and clipboard APIs behave best over HTTP(S).
- Tesseract.js is loaded from the jsdelivr CDN at runtime (`tesseract.js@5`), and the Japanese (`jpn`) language model is downloaded on first OCR run — network access is required at runtime even though OCR itself executes locally.
- The app targets mobile Safari/Chrome (viewport is locked, `capture="environment"` camera input, safe-area insets), so verify UI changes at phone widths.

## Critical Constraint: No Persistence or Transmission

This app handles patient medical data. The notice banner promises users that data is never saved or sent anywhere and that everything is erased when the page closes. This is a deliberate design decision, not an omission:

- All state lives in the in-memory `patients` array only. Do NOT add localStorage, sessionStorage, IndexedDB, cookies, or any network calls that send OCR results or patient data off-device.
- The `beforeunload` handler warns before losing unsaved data; the only export path is clipboard copy (`exportText()`).

## Code Structure (within index.html)

- `<style>` block: mobile-first CSS organized by commented sections (header, capture, progress, patient cards, etc.).
- `<script>` block, organized by `// ───` section comments:
  - **State**: `patients` array plus a lazily-created shared Tesseract `worker` (`initWorker()` creates it once and reuses it).
  - **Capture pipeline**: `startCapture()` → hidden file input `change` handler → `prepareImage()` (resize to max 2000px, grayscale + contrast boost on a canvas for OCR accuracy) → `worker.recognize()` → `parsePatientInfo()` → push to `patients` → `renderCards()`.
  - **`parsePatientInfo()`**: keyword/regex-based extraction of Japanese chart fields (氏名, 年齢, 性別, 既往歴, ADL, 治療). When adding fields, update the field list in `editCard()`, the rows in `renderCards()`, and the output in `exportText()` together.
  - **Rendering**: `renderCards()` rebuilds all cards from `patients` via innerHTML templates. All OCR-derived/user-edited values must go through `escapeHtml()` before interpolation (raw text in the `<textarea>` is the one existing exception — keep that in mind if touching it).

## Conventions

- All UI text, alerts, and code comments are in Japanese; keep new user-facing strings and section comments in Japanese to match.
- Keep the app self-contained in `index.html` — no frameworks, no build step. Vanilla DOM APIs and inline `onclick` handlers are the established pattern.
