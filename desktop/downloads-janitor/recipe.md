# Downloads Janitor (Power Automate Desktop)

Sorts everything in your Downloads folder into category subfolders (`Images`, `Documents`, `Spreadsheets`, `Presentations`, `Archives`, `Installers`, `Audio`, `Video`) by file extension.

**Pure PAD**: built entirely from native Power Automate Desktop actions, no PowerShell, no scripts, no paid connectors.

## Design

One loop over **all** files in Downloads. For each file: lowercase its extension, match it against eight category lists to pick a target folder, then (unless it's a dry run) move it. Files whose extension isn't in any list, including in-progress downloads like `.crdownload` and `.part`, are simply left alone.

## What you get

- A report dialog: how many files were categorized vs left alone.
- A **Dry run** switch (on by default) so the first run only *reports* and moves nothing.
- Safe moves: never overwrites (a same-name clash is left in place), unknown/in-progress files untouched.

---

## Fastest path: paste it in

1. Open **Power Automate Desktop** → **New flow** → name it `Downloads Janitor`.
2. Click once in the empty actions workspace.
3. Open [`flow.txt`](flow.txt), copy the block below its `8<` marker, and **paste** (Ctrl+V).
4. Press **Run**. Dry run is on, so you'll get a report and nothing moves.

This flow uses the `Contains` operator and an If/Else-if chain that some PAD versions paste imperfectly. If the paste looks off, build it by hand below, it's the reliable path and takes ~10 minutes.

---

## Build it by hand (and understand every step)

### 1. Set up variables

**`Get environment variable`** → Name `USERPROFILE`, save to `UserProfile`
**`Set variable`** → `DownloadsFolder` = `%UserProfile%\Downloads`
**`Set variable`** → `DryRun` = `True`
**`Set variable`** → `Categorized` = `0`
**`Set variable`** → `LeftAlone` = `0`

Now the eight category lists. Each is a **`Set variable`** whose value is the extensions wrapped in semicolons (the wrapping `;` is what makes the match exact, so `.pt` never matches inside `.pptx`). **Include the leading dots**, PAD's file extension property returns them.

| Variable | Value |
|----------|-------|
| `ImagesExt` | `;.jpg;.jpeg;.png;.gif;.webp;.svg;.bmp;.heic;.tiff;.ico;` |
| `DocumentsExt` | `;.pdf;.doc;.docx;.txt;.md;.rtf;.odt;.epub;` |
| `SpreadsheetsExt` | `;.xls;.xlsx;.csv;.ods;` |
| `PresentationsExt` | `;.ppt;.pptx;.odp;` |
| `ArchivesExt` | `;.zip;.rar;.7z;.tar;.gz;.bz2;` |
| `InstallersExt` | `;.exe;.msi;` |
| `AudioExt` | `;.mp3;.wav;.flac;.m4a;.aac;.ogg;` |
| `VideoExt` | `;.mp4;.mov;.avi;.mkv;.webm;.wmv;` |

### 2. Get the files

**`Get files in folder`**
- Folder: `%DownloadsFolder%`
- File filter: `*`
- Include subfolders: **Off**
- Save file list to: `Files`

### 3. Loop over each file

**`For each`** → Value to iterate `%Files%`, store in `CurrentFile`.

Everything below goes **inside** the loop.

**`Change text case`** *(this normalizes the extension so `.JPG` matches `.jpg`)*
- Text to convert: `%CurrentFile.Extension%`
- Convert to: **lowercase**
- Save to: `Ext`

**`Set variable`** → `TargetName` = `%''%`  *(empty; the folder we'll decide next)*

**`If`** → First operand `%ImagesExt%`, operator **Contains**, second operand `;%Ext%;`
&nbsp;&nbsp;inside: **`Set variable`** `TargetName` = `Images`

**`Else if`** → `%DocumentsExt%` **Contains** `;%Ext%;` → set `TargetName` = `Documents`
**`Else if`** → `%SpreadsheetsExt%` **Contains** `;%Ext%;` → `TargetName` = `Spreadsheets`
**`Else if`** → `%PresentationsExt%` **Contains** `;%Ext%;` → `TargetName` = `Presentations`
**`Else if`** → `%ArchivesExt%` **Contains** `;%Ext%;` → `TargetName` = `Archives`
**`Else if`** → `%InstallersExt%` **Contains** `;%Ext%;` → `TargetName` = `Installers`
**`Else if`** → `%AudioExt%` **Contains** `;%Ext%;` → `TargetName` = `Audio`
**`Else if`** → `%VideoExt%` **Contains** `;%Ext%;` → `TargetName` = `Video`

Close the If (all the Else-if branches live in one `If` action, add them with the **Else if** button inside it).

> Building the chain: add one `If`, then use its **"New else if"** to add each of the other 7 conditions. Each branch contains a single `Set variable`.

**`If`** → `%TargetName%` operator **Not equal to** `%''%`  *(did we match a category?)*

Inside this `If`:

&nbsp;&nbsp;**`Set variable`** → `TargetPath` = `%DownloadsFolder%\%TargetName%`

&nbsp;&nbsp;**`If folder exists`** → `%TargetPath%`, condition **does not exist**
&nbsp;&nbsp;&nbsp;&nbsp;inside: **`Create folder`** → in `%DownloadsFolder%`, name `%TargetName%`

&nbsp;&nbsp;**`Set variable`** → `Categorized` = `%Categorized + 1%`

&nbsp;&nbsp;**`If`** → `%DryRun%` **Equal to** `False`
&nbsp;&nbsp;&nbsp;&nbsp;inside: **`Move file(s)`** → Files to move `%CurrentFile%`, Destination `%TargetPath%`, If file exists **Do nothing**

Add an **`Else`** to that outer `If` (the category check):
&nbsp;&nbsp;**`Set variable`** → `LeftAlone` = `%LeftAlone + 1%`

### 4. Show the report

**After** the loop (`Display message`, in the **Message boxes** group):
- Title: `Downloads Janitor`
- Message: `Dry run: %DryRun%    Categorized: %Categorized%    Left alone: %LeftAlone%`

---

## Test it safely

1. **First run, `DryRun = True`.** Nothing moves. The dialog shows how many files *would* be sorted vs left alone.
2. If **Categorized is 0** but you have known files, your PAD returns extensions *without* the leading dot, remove the dots from the eight `...Ext` list variables and rerun.
3. When happy, set `DryRun` to `False` and run for real. Moves use **Do nothing on conflict**, so a same-name clash is left in Downloads, never overwritten.

## Schedule it (optional)

- **Power Automate Desktop console** → right-click the flow → schedule, or
- Windows **Task Scheduler** → **Start a program** → `PAD.Console.Host.exe` with the flow name, on your trigger (daily, at logon, ...).

## Ideas to extend later

- Sweep unknowns into an `Other` folder: give `TargetName` a final `Else` = `Other` before the move check.
- Nest by month: set `TargetPath` to `%DownloadsFolder%\%TargetName%\` plus the current date formatted `yyyy-MM`.
- Write the report to a file with **Write text to file** instead of a dialog.
