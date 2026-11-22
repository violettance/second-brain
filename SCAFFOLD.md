# Set up a workspace

A workspace is one folder on your computer. This file describes everything that goes into it. Follow the steps by hand, or give this file to your AI agent and ask it to set up the workspace for you.

The examples use `my-workspace`. Run the commands from inside that folder.

## 1. Choose a pack

A pack is a set of modules for one kind of work. Each module is a folder in the workspace.

| Pack | `packId` | Modules |
|---|---|---|
| Personal | `personal` | `daily-notes`, `projects`, `closeouts`, `playbook` |
| B2B Growth | `b2b-growth` | `company-research`, `icp`, `pipeline`, `closeouts`, `playbook` |

Choose at least one module from the pack. The usual start is `daily-notes`, `projects` and `closeouts` for Personal, and `company-research`, `icp` and `pipeline` for B2B Growth.

## 2. Create the folders

```
my-workspace/
├── README.md
├── AGENTS.md
├── workspace.json
├── brain/
│   ├── brain.py
│   ├── README.md
│   ├── routes.json
│   ├── inbox/
│   ├── inbox-archive/
│   ├── sources/
│   ├── records/
│   ├── sessions/
│   ├── closeouts/
│   └── reviews/
└── one folder per chosen module
```

Copy `brain/brain.py` and `brain/README.md` from this repository into `my-workspace/brain/`. Create the seven empty folders next to them.

## 3. Write workspace.json

```json
{
  "schemaVersion": 1,
  "packId": "personal",
  "modules": ["daily-notes", "projects", "closeouts"],
  "role": null,
  "goal": "Keep daily notes, project next steps, and lessons in one place.",
  "audience": null,
  "qualification": null,
  "timezone": "Europe/London"
}
```

| Field | Rule |
|---|---|
| `schemaVersion` | `1` |
| `packId` | `personal` or `b2b-growth` |
| `modules` | At least one module id of the chosen pack |
| `goal` | What the workspace is for, in a sentence or two |
| `role` | Your role, or `null` |
| `audience` | B2B Growth: who you sell to, or `null` |
| `qualification` | B2B Growth: what makes an opportunity qualified, or `null` |
| `timezone` | Your IANA time zone, for example `America/New_York` |

## 4. Create the modules

Create one folder per chosen module. Put a `README.md` in it with the module title as the heading and the description below it.

| Module id | Folder | Title | Description |
|---|---|---|---|
| `daily-notes` | `daily-notes/` | Daily Notes | Keep notes, ideas, and lessons in dated files. |
| `projects` | `projects/` | Projects | Track project goals, status, and next steps. |
| `company-research` | `companies/` | Company Research | Track companies, contacts, and research notes with their sources. |
| `icp` | `icp/` | Ideal Customer Profile | Define target customer criteria, signals, and market hypotheses. |
| `pipeline` | `pipeline/` | Pipeline | Track opportunity stages, open questions, and follow-ups. |
| `closeouts` | `closeouts/` | Daily Closeouts | Save outcomes, decisions, and open questions with their sources. |
| `playbook` | `playbook/` | Playbook | Keep reusable methods and lessons with their sources. |

Each module starts with one file that shows the shape of an entry. Copy it for every new entry and fill it in.

| Folder | Starter file | Heading | Parts |
|---|---|---|---|
| `daily-notes/` | `note-template.md` | Daily note | Sections: Notes · Decisions and sources · Next steps |
| `projects/` | `project-template.md` | Project | Sections: Goal · Status · Next step · Sources |
| `companies/` | `company-template.md` | Company research | List: Company · Contact and role · Observed need · Source and date · Open questions |
| `icp/` | `criteria.md` | Ideal customer profile | Sections: Target audience · Screening criteria · Observations and hypotheses |
| `pipeline/` | `opportunity-template.md` | Opportunity | List: Customer and contact · Current stage and basis · Open questions · Next step and date. Section: Qualified opportunity criteria |
| `closeouts/` | `daily-template.md` | End of day | Sections: Outcomes · Decisions and sources · Open topics · Next steps |
| `playbook/` | `learning-template.md` | Working rule | List: What was observed? · Under which conditions does it hold? · Which sources support it? · Which finding would change our view? |

In `icp/criteria.md`, write the `audience` from `workspace.json` under Target audience. In `pipeline/opportunity-template.md`, write the `qualification` under Qualified opportunity criteria. Write "To be defined." where the value is `null`.

## 5. Write the workspace README.md

Use the pack name as the heading and the goal as the first paragraph. Under an "Areas" heading, list each chosen module with a link to its `README.md`. End with links to `workspace.json` and `brain/README.md`.

## 6. Write AGENTS.md

`AGENTS.md` tells an AI agent how to work in this workspace. Use this text:

```markdown
# Workspace agent instructions

- Start work with `python3 brain/brain.py context --project workspace`. Find related documents through `brain/routes.json`.
- When asked about daily progress, read the matching `brain/closeouts/YYYY-MM-DD.json` record and the latest session note. Treat chat excerpts and raw inbox entries as sources.
- Record decisions, documented facts, open questions and hypotheses separately. When you take lasting information from a daily record, link it to the dated file and, where possible, to the original excerpt.
- Keep domain details in the matching module. Create lasting memory records with `brain/brain.py record` and link them to a local Markdown or JSON source.
- For a conflicting or outdated record, open a new record and link it with `supersedes`. Keep existing files and source history.
- Save daily and weekly reports with their period and sources through `brain/brain.py report`. Usage steps are in the [Brain guide](brain/README.md).
```

## 7. Write brain/routes.json

List the workspace `README.md`, `workspace.json` and the `README.md` of each chosen module. For the Personal example above:

```json
[
  { "project": "workspace", "role": "current", "path": "README.md" },
  { "project": "workspace", "role": "current", "path": "workspace.json" },
  { "project": "workspace", "role": "current", "path": "daily-notes/README.md" },
  { "project": "workspace", "role": "current", "path": "projects/README.md" },
  { "project": "workspace", "role": "current", "path": "closeouts/README.md" }
]
```

Add other files later when you want `search` to read them.

## 8. Check the result

```sh
python3 brain/brain.py check
```

The output is JSON. An empty `errors` list means the workspace is ready. If a path is listed there, create the missing file or correct the path in `brain/routes.json`.

## Set up again

When a workspace already exists, keep every file that is there and add only what is missing. Keep the existing `workspace.json`.
