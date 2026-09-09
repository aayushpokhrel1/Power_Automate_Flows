# Downloads Janitor (Power Automate Desktop)

Sorts everything in your Downloads folder into category subfolders (`Images`, `Documents`, `Spreadsheets`, `Presentations`, `Archives`, `Installers`, `Audio`, `Video`) by file extension.

**Pure PAD**: built entirely from native Power Automate Desktop actions, no PowerShell, no scripts, no paid connectors.

## What you get

- A report dialog showing how many files fall into each category.
- A **Dry run** switch (on by default) so the first run only *reports* and moves nothing.
- Safe moves: never overwrites (auto-renames `file (2).pdf`), and in-progress downloads (`.crdownload`, `.part`) are skipped automatically because they don't match any category filter.

---

## Fastest path: paste it in

1. Open **Power Automate Desktop** → **New flow** → name it `Downloads Janitor`.
2. In the flow editor, click once in the empty actions workspace.
3. Open [`flow.txt`](flow.txt), copy all of it, and **paste** (Ctrl+V) into the workspace. The actions reconstruct themselves.
4. Press **Run**. With Dry run on, you'll get a report and nothing moves. Read [Test it safely](#test-it-safely) before turning Dry run off.

If the paste doesn't work on your PAD version, build it by hand below, it's the same flow and takes about ten minutes.

---

## Build it by hand (and understand every step)

The design is: figure out the Downloads path, then for each category, grab the matching files, make the folder if needed, and (unless it's a dry run) move them. Concepts you'll learn here, folder actions, `If`, list `.Count`, string building, get reused in every future flow.

### 1. Set up the variables

**Action: `Get environment variable`**
- Name: `USERPROFILE`
- Save to: `UserProfile`

> This is your user folder (e.g. `C:\Users\you`) without hardcoding your name, which keeps the flow shareable.

**Action: `Set variable`**
- Variable: `DownloadsFolder`
- Value: `%UserProfile%\Downloads`

**Action: `Set variable`**
- Variable: `DryRun`
- Value: `True`  *(type it as a General value; PAD treats it as a boolean)*

**Action: `Set variable`**
- Variable: `Report`
- Value: leave the value **empty**

> `Report` is the running text we'll show at the end. `DryRun = True` means "report only, don't move." You'll flip it to `False` once you trust it.

### 2. One block per category

You'll repeat the same three-action pattern for each category. Here it is in full for **Images**, then a table of the values to plug in for the rest.

**Action: `Get files in folder`**
- Folder: `%DownloadsFolder%`
- File filter: `*.jpg;*.jpeg;*.png;*.gif;*.webp;*.svg;*.bmp;*.heic;*.tiff;*.ico`
- Include subfolders: **Off**
- Save file list to: `MatchedFiles`

**Action: `If`**
- First operand: `%MatchedFiles.Count%`
- Operator: **Greater than**
- Second operand: `0`

Inside that `If`:

&nbsp;&nbsp;**Action: `If folder exists`**
&nbsp;&nbsp;- Folder path: `%DownloadsFolder%\Images`
&nbsp;&nbsp;- Condition: **Folder does not exist**

&nbsp;&nbsp;&nbsp;&nbsp;Inside it, **Action: `Create folder`**
&nbsp;&nbsp;&nbsp;&nbsp;- Create new folder in: `%DownloadsFolder%`
&nbsp;&nbsp;&nbsp;&nbsp;- New folder name: `Images`

&nbsp;&nbsp;**Action: `Set variable`** (append to the report)
&nbsp;&nbsp;- Variable: `Report`
&nbsp;&nbsp;- Value: `%Report%Images: %MatchedFiles.Count%;  `

&nbsp;&nbsp;**Action: `If`**
&nbsp;&nbsp;- First operand: `%DryRun%`  Operator: **Equal to**  Second operand: `False`

&nbsp;&nbsp;&nbsp;&nbsp;Inside it, **Action: `Move file(s)`**
&nbsp;&nbsp;&nbsp;&nbsp;- Files to move: `%MatchedFiles%`
&nbsp;&nbsp;&nbsp;&nbsp;- Destination folder: `%DownloadsFolder%\Images`
&nbsp;&nbsp;&nbsp;&nbsp;- If file exists: **Rename**  *(this is what prevents overwrites)*

That's one category. **Copy the whole `If %MatchedFiles.Count% > 0` block** and paste it 7 more times, changing only the **file filter**, the two **folder names** (`Images` → ...), and the **report label**:

| Category | File filter | Folder |
|----------|-------------|--------|
| Images | `*.jpg;*.jpeg;*.png;*.gif;*.webp;*.svg;*.bmp;*.heic;*.tiff;*.ico` | `Images` |
| Documents | `*.pdf;*.doc;*.docx;*.txt;*.md;*.rtf;*.odt;*.epub` | `Documents` |
| Spreadsheets | `*.xls;*.xlsx;*.csv;*.ods` | `Spreadsheets` |
| Presentations | `*.ppt;*.pptx;*.odp` | `Presentations` |
| Archives | `*.zip;*.rar;*.7z;*.tar;*.gz;*.bz2` | `Archives` |
| Installers | `*.exe;*.msi` | `Installers` |
| Audio | `*.mp3;*.wav;*.flac;*.m4a;*.aac;*.ogg` | `Audio` |
| Video | `*.mp4;*.mov;*.avi;*.mkv;*.webm;*.wmv` | `Video` |

> Files whose extension is in none of these lists are left untouched, on purpose. Nothing gets moved somewhere you didn't ask for.

### 3. Show the report

At the very end (outside all the category blocks):

**Action: `Display message`**
- Title: `Downloads Janitor`
- Message: `Dry run: %DryRun%%Environment.NewLine%%Report%`
- (leave the rest default)

Run it. You'll see something like `Images: 12;  Documents: 3;  Archives: 1;`.

---

## Test it safely

1. **First run with `DryRun = True`** (the default). Nothing moves. Check the report dialog matches what's actually in your Downloads folder.
2. Optionally, make a throwaway folder with a few junk files and point `DownloadsFolder` at it for one run.
3. When you trust it, set `DryRun` to `False` and run for real. Because moves use **Rename on conflict**, a name clash creates `file (2).ext`, it never overwrites.

## Schedule it (optional)

- In the **Power Automate Desktop console**, right-click the flow → schedule, or
- Windows **Task Scheduler** → new task → action **Start a program** → `PAD.Console.Host.exe` with the flow name, on your preferred trigger (e.g. daily, or at logon).

## Ideas to extend later

- Add an `Other` category as a final block (`Get files` with filter `*.*`) to sweep leftovers, only after you're happy with the named categories.
- Add a `%Environment.NewLine%`-separated log written to a file with **Write text to file** instead of a dialog.
- Nest by month: create the destination as `%DownloadsFolder%\Images\%CurrentDateTime%` formatted `yyyy-MM`.
