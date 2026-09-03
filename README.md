# Paper Trail — question bank & exam paper generator

A tool for teachers to build up a long-lived question bank (from uploaded
.docx question papers, OCR'd scans/images/PDFs, or typed in by hand),
verify it, and generate exam papers and answer keys from it — with almost
no manual paper-editing beyond design/customization.

## What's in this folder

| File | Purpose |
|---|---|
| `index.html` | The entire app — upload/OCR, question bank, pattern, generate & download. This is the only file your browser loads; everything else here is deployment/config. |
| `firebase.json` | Tells the Firebase CLI what to deploy (Hosting) and which rules files to use. |
| `firestore.rules` | Security rules — each teacher can only read/write their own data. |
| `storage.rules` | Security rules for uploaded question/answer images. |
| `package.json` | Just wires up `npm run deploy` via the Firebase CLI. There's no build step — `index.html` is plain HTML/CSS/JS. |
| `SETUP.md` | Step-by-step: create your Firebase project, wire up the config, deploy. **Start here before first use.** |

## How it works

- **Everything runs in the browser.** Parsing uploaded .docx files (via
  mammoth.js), OCR of scanned PDFs/images (via Tesseract.js + pdf.js), and
  generating the final .docx paper/key (via docx.js) all happen client-side
  — no file ever goes to a server for processing.
- **Accounts and storage are Firebase.** Signing in (email/password) gates
  the app; each teacher's question bank and generated sets are saved to
  their own Firestore document and follow them to any device. Question/
  answer images are stored in Firebase Storage (not inline in Firestore)
  to stay under Firestore's 1MB-per-document limit.
- **The question bank is subject-categorized** (Physics/Chemistry/Biology,
  etc. — taken from each file's own header, or typed in for manual/OCR
  entries), not by the exam paper's Section A/B/C/D labels.
- **Generated papers are ordered by marks, ascending** (all 1-mark
  questions together, then all 2-mark, and so on), formatted close to an
  official paper — marks shown right-aligned as `[N marks]`.
- **Nothing is auto-added to the usable bank.** Parsed/OCR'd/manual
  questions start in the bank as "verified" by default (so a clean upload
  needs no per-question ticking) — you flag/edit the ones that need a
  second look instead of ticking everything by hand.

## Known limits (v1)

- A teacher's whole bank + generated sets are stored as **one Firestore
  document**, capped at 1MB by Firestore itself. This comfortably covers
  hundreds of questions once images are offloaded to Storage, but a very
  large bank could eventually need splitting into per-question documents
  — not implemented yet.
- OCR extracts raw text for you to review and copy into the manual-entry
  form; it does not attempt to automatically detect question boundaries,
  marks, or answers from freeform scanned text (only the strict
  `Qn. [X Marks]` docx format that mammoth produces is auto-parsed).
- Docx export supports the same limited HTML the bank uses internally
  (paragraphs, bold/italic, bullet/numbered lists, images) — not arbitrary
  formatting.
