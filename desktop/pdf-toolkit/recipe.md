# PDF Toolkit (Power Automate Desktop)

One manual-run flow with a menu: **Merge**, **Split**, or **Compress** PDFs. Merge and split use native PAD actions (free, no dependencies). Compress uses Ghostscript (free, open-source, installed once).

## What it looks like when you run it

1. A menu pops up: *What do you want to do?* → **Merge / Split / Compress**.
2. Depending on your pick, it asks for a folder or a file (native picker dialogs).
3. It does the operation and shows a result message. Nothing is deleted, outputs are written as new files next to the source.

## Prerequisite (Compress only)

Merge and Split need nothing. **Compress** needs **Ghostscript** (free): install from ghostscript.com, then either add its `bin` folder to your PATH or note the full path to `gswin64c.exe` (e.g. `C:\Program Files\gs\gs10.03.1\bin\gswin64c.exe`). If it's missing, the Compress branch just shows a "install Ghostscript" message, the rest of the flow is unaffected.

---

## Fastest path: paste it in

Open a **New flow**, click the workspace, paste the block below the `8<` marker in [`flow.txt`](flow.txt). If you'll use Compress and Ghostscript isn't on PATH, set the `GsExe` variable to the full `gswin64c.exe` path. Run.

---

## Build it by hand

### 1. The menu

**`Display selection dialog`** (Message boxes group)
- Dialog title: `PDF Toolkit`
- Message: `What do you want to do?`
- List of options (one per line):
  ```
  Merge
  Split
  Compress
  ```
- Saves: `Choice` (selected item) and `ButtonPressed`.

**`If`** → `%ButtonPressed%` **Not equal to** `OK`  → **`Exit`**  *(cancelled, stop cleanly)*

**`Set variable`** → `GsExe` = `gswin64c.exe`  *(or the full path to Ghostscript's console exe)*

### 2. Branch on the choice

**`Switch`** → value to check `%Choice%`. Then a **`Case`** for each option below.

---

### Case `Merge`

**`Display select folder dialog`** → Description `Pick a folder of PDFs to merge` → saves `PdfFolder`, `ButtonPressed`
**`If`** → `%ButtonPressed%` **Not equal to** `Select` (folder dialog's OK button) → **`Next loop`/skip**: simplest is wrap the rest of the case in `If %ButtonPressed% = 'Select'`.

**`Get files in folder`**
- Folder: `%PdfFolder%`
- File filter: `*.pdf`
- Sort by: **Name**, ascending
- Save to: `PdfFiles`

**`If`** → `%PdfFiles.Count%` **Less than or equal to** `1`
&nbsp;&nbsp;**`Display message`** → `Need at least 2 PDFs in that folder to merge.` then **`Exit`** (or skip)

**`Merge PDF files`**
- PDF files to merge: `%PdfFiles%`
- Merged PDF path: `%PdfFolder%\merged.pdf`
- Save to: `MergedPdf`

**`Display message`** → `Merged %PdfFiles.Count% files into merged.pdf` *(re-running overwrites merged.pdf; rename it if you want to keep it)*

---

### Case `Split`

**`Display select file dialog`**
- Title: `Pick a PDF to split`
- File filter: `*.pdf`
- Save to: `SrcFile`, `ButtonPressed`

**`Display input dialog`**
- Title: `Split`
- Message: `Pages to extract (e.g. 1-3 or 2,5,7)`
- Save to: `Pages`, `ButtonPressed`

**`Extract PDF file pages to new PDF file`**
- PDF file: `%SrcFile.FullName%`
- Page selection: `%Pages%`
- Extracted PDF path: `%SrcFile.Directory%\%SrcFile.NameWithoutExtension%_pages.pdf`
- Save to: `ExtractedPdf`

**`Display message`** → `Extracted pages %Pages% → %SrcFile.NameWithoutExtension%_pages.pdf`

---

### Case `Compress`

**`Display select file dialog`** → Title `Pick a PDF to compress`, filter `*.pdf` → saves `SrcFile`, `ButtonPressed`

**`Set variable`** → `OutPdf` = `%SrcFile.Directory%\%SrcFile.NameWithoutExtension%_small.pdf`

**`Run application`**  *(set On error → Continue flow run, so a missing Ghostscript doesn't halt)*
- Application path: `%GsExe%`
- Command line arguments:
  ```
  -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 -dPDFSETTINGS=/ebook -dNOPAUSE -dQUIET -dBATCH -sOutputFile="%OutPdf%" "%SrcFile.FullName%"
  ```
- Window style: **Hidden**
- Wait for the application to complete: **On**

**`If file exists`** → `%OutPdf%` **exists**
&nbsp;&nbsp;**`Get file info`** on `%OutPdf%` → `NewInfo`, then
&nbsp;&nbsp;**`Display message`** → `Compressed: %SrcFile.Size% bytes → %NewInfo.Size% bytes  (%SrcFile.NameWithoutExtension%_small.pdf)`
**`Else`**
&nbsp;&nbsp;**`Display message`** → `Compress failed. Is Ghostscript installed? Set GsExe to the full path of gswin64c.exe.`

> Quality knob: `/ebook` (150 dpi, good default). Use `/screen` for smallest (72 dpi) or `/printer` for higher quality/larger.

Close the `Switch` (`End`).

---

## Test it safely

- **Merge:** put 2-3 PDFs in a folder, run → Merge → confirm `merged.pdf` opens with all pages in order.
- **Split:** pick a multi-page PDF, enter `1-2`, confirm `<name>_pages.pdf` has just those pages.
- **Compress:** pick a large PDF, confirm `<name>_small.pdf` appears and is smaller. If you get the "install Ghostscript" message, install it or fix `GsExe`.

Outputs are always new files; originals are never modified or deleted.
