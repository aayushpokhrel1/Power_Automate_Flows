# Resume Router (Power Automate Desktop)

Manual-run flow that moves downloaded resumes out of Downloads and into their per-job application folder, one or many per run. Free tier, no premium triggers.

## How it works

A downloaded resume is named `resume - <slug>` (e.g. `resume - google-swe.pdf`). The flow splits the filename on ` - ` to get the `<slug>` token, builds the destination `<applications root>\<slug>`, and if that folder exists, moves the file in. If not, it tells you to file it by hand. Moving out of Downloads means it self-cleans.

### The naming consensus (why there's no fuzzy matching)

This flow does an **exact-name match** on purpose. It works because of a convention written into the `job-search` project's CLAUDE.md: when a resume is enhanced for a job, the folder `documents/applications/<slug>/` is created **and** the Google Doc is titled exactly `resume - <slug>`. So the download's filename token and the folder name are byte-identical, and the move routes with zero guessing.

### Deferred gaps (known, on purpose)

- **Exact match only.** No slug fuzzy-matching, if the token doesn't equal a folder name exactly, it's left for manual filing.
- **No duplicate/collision handling.** A browser re-download named `resume - google-swe (1).pdf` splits to the token `google-swe (1)`, which won't match the folder. Move also uses *Do nothing* on conflict, so a same-name file in the destination is left in Downloads.

---

## Fastest path: paste it in

Open a **New flow**, click the workspace, paste the block below the `8<` marker in [`flow.txt`](flow.txt). Then set `AppRoot` (see step 1) to your applications folder and Run.

---

## Build it by hand

### 1. Variables

**`Set variable`** → `AppRoot` = `%UserProfile%\dev\Projects\job-search\documents\applications`
&nbsp;&nbsp;*(change this to wherever your per-job folders live)*

> `%UserProfile%` avoids hardcoding your username. `Get environment variable USERPROFILE → UserProfile` first if `%UserProfile%` isn't already available in your PAD.

### 2. Get the resumes

**`Get files in folder`**
- Folder: `%UserProfile%\Downloads`
- File filter: `resume - *`
- Include subfolders: **Off**
- Save file list to: `ResumeFiles`

### 3. Guard: nothing to do

**`If`** → `%ResumeFiles.Count%` **Equal to** `0`
&nbsp;&nbsp;**`Display message`** → Title `Resume Router`, Message `No resumes found in Downloads.`
&nbsp;&nbsp;**`Exit`** *(Flow control → Exit, ends the run cleanly)*

Everything below is **after** this guard (or wrap the rest in the `Else`).

### 4. Route each resume

**`For each`** → iterate `%ResumeFiles%`, store in `CurrentFile`.

Inside the loop:

**`Split text`**
- Text to split: `%CurrentFile.NameWithoutExtension%`
- Delimiter type: **Custom**, delimiter: ` - ` *(space-dash-space)*
- Save to: `NameParts`

> The slug can contain its own hyphens (`google-swe`); splitting on ` - ` keeps it intact and just peels off the `resume` prefix.

**`Trim text`**
- Text: `%NameParts[1]%`  *(second element, the slug token)*
- Trim: leading and trailing whitespace
- Save to: `Slug`

**`Set variable`** → `Dest` = `%AppRoot%\%Slug%`

**`If folder exists`** → `%Dest%`, condition **exists**
&nbsp;&nbsp;**`Move file(s)`** → Files to move `%CurrentFile%`, Destination `%Dest%`, If file exists **Do nothing**
**`Else`**
&nbsp;&nbsp;**`Display message`** → Title `Resume Router`, Message `Move by hand: %CurrentFile.Name% — no folder for "%Slug%" under %AppRoot%`

Close the loop.

---

## Test it safely

1. Drop a throwaway `resume - test-slug.pdf` in Downloads and make a matching `test-slug` folder under your applications root. Run, confirm it moves.
2. Rename the file to a slug with no matching folder, run, confirm you get the "move by hand" message and the file stays put.

## Closing the deferred gaps later

- **Collision handling:** before the move, if `%Dest%\%CurrentFile.Name%` exists, append a timestamp to the filename with **Rename file(s)**.
- **Fuzzy match:** loop the folders under `AppRoot` and pick the best `Contains`/similarity match instead of requiring exact equality, only if the exact-name convention ever slips.
