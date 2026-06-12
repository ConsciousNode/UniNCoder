# UniNCoder
### Forensic Unicode Analyzer
*Tama Hagane Seam Labs · ConsciousNode SoftWorks · v1.0.0*

---

## What It Is

UniNCoder is a forensic document tool for detecting, locating, and stripping hidden or anomalous Unicode characters from text — the kind that don't show up visually but can carry injection vectors, manipulate text rendering, or corrupt corpus integrity.

It runs entirely in the browser. No server. No dependencies. No network requests. One HTML file.

---

## Quick Start

1. Open `unincode.html` in any modern browser.
2. Paste text into the **Paste Text** tab, or drag and drop a file into **Upload File**.
3. Press **⌖ Analyze** (or `Ctrl+Enter` / `Cmd+Enter`).
4. Review flagged characters in the **Annotated View** and **Findings** table.
5. Select categories to strip, press **Strip Selected**, review the diff.
6. **Copy** or **Download** the clean output.

---

## Supported Input Formats

| Format | Notes |
|--------|-------|
| `.txt` | Full UTF-8 text extraction |
| `.json` | Treated as plaintext — structure preserved |
| `.md` | Treated as plaintext |
| `.csv` | Treated as plaintext |
| `.pdf` | Text layer extracted in-browser. Works on text-layer PDFs. Scanned/image-only PDFs are not supported in browser mode. |

Paste input accepts any text directly — including content copied from terminals, documents, or web pages.

---

## What Gets Flagged

UniNCoder detects six categories of anomaly:

### 🔴 Zero-Width / Invisible
Characters that occupy no visual space but exist in the byte stream. Common injection and steganography vectors.

| Character | Codepoint | Name |
|-----------|-----------|------|
| ​ | U+200B | Zero-Width Space |
| ‌ | U+200C | Zero-Width Non-Joiner |
| ‍ | U+200D | Zero-Width Joiner |
| ﻿ | U+FEFF | BOM / Zero-Width No-Break Space |
| ⁠ | U+2060 | Word Joiner |
| ­ | U+00AD | Soft Hyphen |
| ͏ | U+034F | Combining Grapheme Joiner |
| ᠎ | U+180E | Mongolian Vowel Separator |
| + 4 more invisible formatting chars | | |

### 🟡 Direction Overrides
Unicode bidirectional control characters. The Right-to-Left Override (U+202E) in particular is a well-documented attack vector — it can make a filename or string display as the reverse of what it actually is.

Includes: LRE, RLE, PDF, LRO, **RLO**, LRI, RLI, FSI, PDI, LRM, RLM.

### 🟣 Homoglyphs
Characters from other scripts that are visually indistinguishable (or nearly so) from ASCII. Used in phishing, lookalike domain attacks, and prompt injection.

Detected scripts: **Cyrillic** (е→e, о→o, р→p, с→c, а→a, and uppercase equivalents), **Greek** (α→a, ο→o, ρ→p, etc.), **Fullwidth Latin** (Ａ→A, ａ→a, etc.), and common lookalike punctuation (curly quotes, em-dashes, interpunct).

When stripping, homoglyphs are *replaced* with their ASCII equivalent rather than deleted — the text stays readable.

### 🔵 Encoded Content
Runs of Base64-encoded text ≥ 20 characters that decode to readable content. Flags the run and shows a preview of the decoded value in the tooltip and findings table.

### 🔴 Control Characters
Non-printable control characters outside of normal whitespace (tab, LF, CR):
- C0 controls: U+0000–U+001F
- DEL: U+007F  
- C1 controls: U+0080–U+009F

### ⚫ Other Anomalies
Catch-all for anything else flagged by future detection rules.

---

## The Interface

### Annotated View
The full input text rendered with flagged characters replaced by **inline colored badges** showing:
- The character itself (or `␣` for invisible/control chars)
- A short label (e.g. `ZWS`, `RLO`, `≈e`)
- Hover tooltip: full name, codepoint, line:col, and surrounding context

### Findings Table
A sortable record of every anomaly:

| Column | Description |
|--------|-------------|
| Line:Col | Exact position in the source text |
| Char | The flagged character with its badge |
| Codepoint | Unicode codepoint in U+XXXX format |
| Type | Human-readable category label |
| Context | ~24 characters of surrounding text |

### Summary Sidebar
Live counts by category after analysis. Anomaly count turns red if anything is found.

### Strip Controls
Select which categories to remove, then hit **Strip Selected**. Options:

- ☑ Zero-width / invisible — *on by default*
- ☑ Direction overrides — *on by default*
- ☐ Homoglyphs → ASCII — *off by default* (replaces rather than deletes; review before enabling)
- ☑ Control characters — *on by default*
- ☐ Other anomalies — *off by default*

Strip is **non-destructive** — the original text is never modified. A clean copy is generated separately.

### Diff View
Appears after stripping. Shows a line-by-line before/after comparison:
- <span style="background:#FFE5E5">Red strikethrough</span> = original line with anomalies
- <span style="background:#D4EDDA">Green</span> = cleaned version

### Report
Plain-text exportable summary: timestamp, source filename, total character count, anomaly counts by category, and a full findings list with positions. Copy to clipboard or include in audit documentation.

---

## Output Options

| Button | Action |
|--------|--------|
| Copy Clean Text | Copies stripped text to clipboard |
| Download .txt | Downloads stripped text as `[original_name]_clean.txt` |
| Copy (Report) | Copies the full report to clipboard |

---

## Keyboard Shortcut

`Ctrl+Enter` / `Cmd+Enter` — Run analysis from anywhere on the page.

---

## Xinu Compliance

UniNCoder is fully Xinu-compliant:

- **Single file** — everything in `unincode.html`
- **Zero external dependencies** — no CDN, no npm, no network requests after load
- **Offline-first** — works with no internet connection once the file is open
- **One job** — forensic Unicode detection, done well
- **System fonts only** — `ui-monospace` stack, no webfont loading

---

## Mobile

UniNCoder is fully usable on mobile browsers. Layout adapts to small screens:
- Sidebar panels reflow to a 2-column grid
- Touch targets sized for fingers (≥44px)
- Context column hidden in findings table (available via tooltip on tap)
- Font sizes and spacing adjusted for readability

---

## Limitations & Known Gaps

- **PDF support is browser-side text extraction only.** Scanned PDFs (image-only, no text layer) will return little or no content. For those, extract text first with an external OCR tool and paste the result.
- **Homoglyph detection covers common scripts** (Cyrillic, Greek, Fullwidth Latin) but is not exhaustive. The Unicode Consortium's full confusables list is ~7,000 entries; UniNCoder ships a curated high-risk subset.
- **Base64 detection has false-positive risk** on long alphanumeric strings (API keys, hashes, etc.) that happen to be valid Base64. Review B64 findings before stripping.
- **No server-side processing.** All analysis runs in the browser JS engine. Very large files (>1MB text) may be slow on low-end devices.

---

## Provenance

Commissioned by **Tama Hagane Seam Labs** (SD Rapp).  
Built by **Kehai Interim / ConsciousNode SoftWorks**.  
Specification passed May–June 2026.

*Spiritually adjacent to Once Bytten (Ed Interim, ConsciousNode SoftWorks) — byte-level forensics, different layer.*

---

## Version History

| Version | Date | Notes |
|---------|------|-------|
| v1.0.0 | June 2026 | Initial release. Full detection suite, strip, diff, report, mobile layout, Xinu compliant. |

---

*ConsciousNode SoftWorks — bespoke artifacts built from principles, not assembled from parts.*
