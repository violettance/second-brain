# Brain engine

`brain.py` keeps the source-backed memory of one workspace. It is a single file that uses only the Python standard library. It reads and writes files inside the workspace and makes no network calls.

Run every command from the workspace root. The engine works on the `brain/` folder it sits in and treats the folder above it as the workspace.

```sh
python3 brain/brain.py context --project workspace
python3 brain/brain.py search 'weekly review' --project workspace
python3 brain/brain.py check
```

## Folders

| Folder | Contents |
|---|---|
| `inbox/` | Unprocessed entries written by `capture` |
| `inbox-archive/` | Entries that were turned into a record |
| `sources/` | Source notes and evidence |
| `records/` | Lasting records, one JSON file per record |
| `sessions/` | Handoff notes from work sessions, named `YYYY-MM-DD-….md` |
| `closeouts/` | Daily reports, a JSON file and a Markdown view per day |
| `reviews/` | Weekly reports, a JSON file and a Markdown view per week |

## Commands

| Command | What it does |
|---|---|
| `context` | Prints the current records with their sources and warnings. `--query` narrows by words, `--candidates` includes hypotheses, `--budget` sets the character limit. |
| `search QUERY` | Searches records and the files listed in `routes.json`. `--history` includes replaced and closed records, `--candidates` includes hypotheses. |
| `check` | Reports changed sources, records due for review, open questions and inbox entries as JSON. Exits with 1 when a route or a record is invalid. |
| `capture` | Saves plain text from `--file` or standard input to `inbox/` as an unprocessed entry. |
| `record --file` | Adds one lasting record from JSON. |
| `prepare --kind daily\|weekly` | Prints what a report for the period can build on: recent records, open questions, hypotheses, existing reports and session notes. `--date` sets the last day. It writes nothing. |
| `report --kind daily\|weekly --file` | Validates a report and saves it as JSON and Markdown. |
| `check-report-sources` | Lists local files that reports cite and that are now missing. `--mark-reported` remembers the ones already listed. |
| `backlog` | Lists the daily and weekly periods that have no report yet. It needs `catchup-policy.json` and writes nothing. |

`--file -` reads from standard input. A file passed to `--file` must be inside the workspace.

## Records

A record is a decision, fact, question, hypothesis or outcome. It always names the source it came from.

```json
{
  "id": "decision-weekly-review-day",
  "project": "workspace",
  "kind": "decision",
  "status": "current",
  "evidence": "user",
  "text": "Weekly reviews happen on Fridays.",
  "recorded_on": "2026-01-09",
  "review_on": "2026-02-09",
  "sources": [
    { "path": "daily-notes/2026-01-09.md", "quote": "weekly review on Fridays" }
  ]
}
```

| Field | Rule |
|---|---|
| `id` | 3 to 96 characters: lowercase letters, digits and hyphens. It is also the file name. |
| `project` | `workspace` |
| `kind` and `status` | `decision`, `fact` and `outcome` are `current` or `closed`. A `question` is `open` or `closed`. A `hypothesis` is `candidate` or `closed`. |
| `evidence` | `user` for something the user said, `document` for something a file says, `inference` for a conclusion. An inference can only be a hypothesis or a question. |
| `text` | 1 to 1,600 characters |
| `recorded_on`, `review_on` | Dates as `YYYY-MM-DD`. From the review date on, the record is flagged for a recheck. |
| `sources` | At least one Markdown or JSON file in the workspace, given as a path relative to the workspace root. An optional `quote` must appear exactly once in that file. |
| `supersedes` | Optional list of record ids that this record replaces |

The engine stores a fingerprint of each source: of the quote when there is one, otherwise of the whole file. When the source changes later, `context`, `search` and `check` flag the record.

A saved record is never edited. To correct one, add a new record with a new id and list the old id in `supersedes`. The old file stays as history.

## Reports

A daily report covers one day. A weekly report covers the seven days that end on `period_end`. A report cannot cover a future day.

```json
{
  "period_start": "2026-01-09",
  "period_end": "2026-01-09",
  "scope_note": "Reviewed the daily note for the day.",
  "coverage": [
    { "source": "Daily notes", "scope": "daily-notes/2026-01-09.md", "status": "reviewed" }
  ],
  "entries": [
    {
      "id": "review-day-moved",
      "project": "workspace",
      "category": "decision",
      "text": "Weekly reviews move to Fridays.",
      "evidence": ["daily-notes/2026-01-09.md"],
      "record_ids": ["decision-weekly-review-day"]
    }
  ]
}
```

| Field | Rule |
|---|---|
| `scope_note` | What was reviewed for this period |
| `coverage` | One row per source with `source`, `scope` and a `status` of `reviewed`, `partial` or `unavailable` |
| `entries[].category` | `outcome`, `decision`, `ongoing`, `uncertainty`, `learning`, `hypothesis` or `next`. A hypothesis also needs a `validation` step. |
| `entries[].evidence` | At least one reference: a Markdown or JSON file in the workspace, or a reference that starts with `https://`, `thread:` or `commit:` |
| `entries[].record_ids` | Optional ids of records that already exist |
| `entries[].supersedes` | Optional ids of entries in the same report that this entry corrects |
| `timezone` | Optional. The Markdown view shows it; without it the workspace time zone is shown. |

Saving a report for a period that already has one merges the two. Entries that are already saved cannot change: add a new entry and list the old id in `supersedes`.

## routes.json

`routes.json` lists the files that `search` reads besides the records. Each path must be an existing Markdown or JSON file in the workspace; `check` reports the ones that are missing.

```json
[
  { "project": "workspace", "role": "current", "path": "README.md" },
  { "project": "workspace", "role": "current", "path": "workspace.json" }
]
```

| Role | Searched |
|---|---|
| `current` | Always |
| `history` | Only with `--history` |
| `hypotheses` | Only with `--candidates` |

## Catch-up plan

`backlog` works only when `brain/catchup-policy.json` exists and is enabled. Hours are in the workspace time zone and weekdays count from Monday as 0.

```json
{
  "enabled": true,
  "started_on": "2026-01-05",
  "daily_hour": 18,
  "weekly_hour": 18,
  "weekly_weekday": 4
}
```

## Time zone

Dates follow the `timezone` field of `workspace.json`. Without it the engine uses the computer's time zone, and UTC when that cannot be read.

## Limits

- Python 3.9 or later on macOS or Linux.
- Sources are Markdown and JSON files inside the workspace. Symbolic links are not followed.
- Search matches words; it is not semantic.
- The engine records what it is given. It does not check whether a statement is true.
