# Power Automate Flows

A growing library of [Power Automate](https://learn.microsoft.com/power-automate/) flows, both **cloud** (Power Automate online) and **desktop** (Power Automate Desktop / RPA), that automate real, repetitive tasks. Each flow ships with the built artifact *and* a plain-English recipe so you can rebuild and understand it yourself.

## Guiding principle: plumbing vs judgment

Automation is worth building when the work is **deterministic plumbing**: moving files, syncing data, detecting an email, chasing a deadline. That work needs no AI and no human clicking, so it goes to Power Automate. Anything that needs actual judgment (scoring, writing, deciding) stays with a human or an LLM. The whole library is organised around handing the plumbing to Power Automate so the expensive brain-work is all that's left.

## Layout

```
cloud/<flow-name>/      # Power Automate online flows: export.zip + recipe.md
desktop/<flow-name>/    # Power Automate Desktop flows: flow.txt (paste-in) + recipe.md
templates/              # generic, shareable copies with personal details stripped
```

Each flow folder is self-contained:

- **`recipe.md`** the authoritative build guide, action by action. Follow this to build the flow yourself in the designer and understand what each step does.
- **`flow.txt`** (desktop) a paste-in convenience. Select actions in a Power Automate Desktop flow, and you can *paste* this text to reconstruct the flow. If a paste ever fails (PAD versions differ), `recipe.md` is the source of truth.

## Flows

| Flow | Type | What it does |
|------|------|--------------|
| [downloads-janitor](desktop/downloads-janitor/) | Desktop | Sorts your Downloads folder into category subfolders (Images, Documents, ...). Safe: never overwrites, skips in-progress downloads, has a dry-run. |
| [resume-router](desktop/resume-router/) | Desktop | Moves downloaded `resume - <slug>` files into their matching per-job application folder by exact-name match. Self-cleans Downloads. |

More on the way: stale-application reminders, receipt logging, web-table scraping.

## How to use a desktop flow

1. Open **Power Automate Desktop** and create a new flow.
2. Either follow the flow's `recipe.md` step by step, or open the flow, select the workspace, and paste the contents of `flow.txt`.
3. Read the recipe's **Safety** and **Test** sections before the first real run.
4. (Optional) Schedule it from the Power Automate Desktop console, or via Windows Task Scheduler.

## License

MIT. See [LICENSE](LICENSE).
