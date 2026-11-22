# Second Brain

Second Brain is a framework for keeping your working memory in plain files on your own computer. An AI agent such as Codex reads and writes those files through one small engine, and every lasting record names the source it came from.

## What is in this repository

| Path | Contents |
|---|---|
| [`brain/brain.py`](brain/brain.py) | The memory engine: one file, Python standard library only |
| [`brain/README.md`](brain/README.md) | Engine guide: commands, record and report formats |
| [`SCAFFOLD.md`](SCAFFOLD.md) | How to set up a workspace and what the two packs contain |

## How it works

- A **workspace** is a folder you choose. Notes, sources and memory stay in it as Markdown and JSON files.
- A **pack** sets up the folders for one kind of work. Two packs are described today: Personal and B2B Growth.
- A **record** is a decision, fact, question, hypothesis or outcome. It has a date, a review date and at least one source file.
- The engine keeps a fingerprint of each source. When the source changes, the record is flagged for a recheck.
- A record is never edited. A correction is a new record that names the one it replaces, so the history stays.
- A **closeout** is the report of one day and a **review** is the report of one week. Both list which sources were reviewed and how completely.

## Start

1. Set up a workspace with [SCAFFOLD.md](SCAFFOLD.md). You can follow it by hand or give it to your agent.
2. From the workspace folder, run:

   ```sh
   python3 brain/brain.py check
   ```

3. Ask your agent to start its work with:

   ```sh
   python3 brain/brain.py context --project workspace
   ```

The [engine guide](brain/README.md) describes every command.

## Requirements

Python 3.9 or later on macOS or Linux. Nothing to install.

## The app

The Second Brain app at [secondbrainos.app](https://secondbrainos.app) is a separate interface for the same files. The framework works without it.

## License

You may use, copy and change Second Brain free of charge. You may not sell it: that covers the software itself and any product or service whose value comes mainly from it. The terms are the MIT License with the Commons Clause condition, in [LICENSE](LICENSE).

For a commercial license, write to [growth@productora.co](mailto:growth@productora.co).
