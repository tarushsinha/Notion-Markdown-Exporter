# Notion Markdown Exporter

Standalone interactive exporter for syncing Notion articles to Markdown files with YAML front matter.

This project is designed for a simple local workflow:

1. Connect to a Notion database or data source
2. List exportable articles
3. Select one or more articles to export
4. Choose where to save the generated Markdown files
5. Write the files into the selected destination

## What This Tool Does

- Queries a Notion data source for available pages
- Fetches nested block content recursively
- Converts supported Notion blocks into Markdown
- Preserves front matter fields used by the exporter
- Allows multi-file export in one run
- Lets you choose the output directory at runtime

## Requirements

To run this successfully, you need:

- Python 3 available on your system `PATH`
- Python `venv` support
- `pip` available for that Python installation
- Network access to `api.notion.com`
- A Notion integration token with access to the target database
- A valid `.env` file in this project directory

## Notion Setup

This exporter needs a Notion integration token plus either a database ID or a data source ID.

### 1. Create a Notion integration and get the API key

General process:

1. Go to the Notion integrations settings page and create an internal integration for the workspace you want to use.
2. Open the integration's `Configuration` tab.
3. Copy the integration token and place it in `.env` as `NOTION_API_KEY`.
4. In Notion, open the target page or database and manually share it with that integration using `Add connections`.

Official documentation:

- Authorization guide: https://developers.notion.com/guides/get-started/authorization
- API introduction: https://developers.notion.com/reference/intro

### 2. Find the database ID

General process:

1. Open the target database in Notion.
2. Copy the page URL.
3. The database ID is the long identifier in the URL between the workspace path and the query string.
4. Set that value as `NOTION_DB_ID` if you want the script to resolve the data source automatically.

Official documentation:

- Retrieve a database: https://developers.notion.com/reference/database-retrieve

### 3. Find the data source ID

This project supports `NOTION_DATA_SOURCE_ID` directly, which is the most explicit option on newer Notion API versions.

General process:

1. Open the target database in Notion.
2. Use the database settings menu and copy the data source ID if the UI exposes it.
3. If you only have the database ID, call the Retrieve a database API and inspect the returned `data_sources` array.
4. Set the chosen value as `NOTION_DATA_SOURCE_ID`.

Official documentation:

- Working with databases: https://developers.notion.com/docs/working-with-databases
- Retrieve a data source: https://developers.notion.com/reference/retrieve-a-data-source
- Data source object reference: https://developers.notion.com/reference/data-source

### Required Environment Variables

Create a `.env` file in the project root. You can start from `.env.example`.

Expected variables:

```env
NOTION_API_KEY=your_notion_api_key
NOTION_DB_ID=your_notion_database_id
NOTION_DATA_SOURCE_ID=your_data_source_id
```

Notes:

- `NOTION_API_KEY` is required.
- `NOTION_DATA_SOURCE_ID` is used directly if present.
- If `NOTION_DATA_SOURCE_ID` is not set, the script will use `NOTION_DB_ID` to resolve the data source from the database metadata.
- If neither `NOTION_DATA_SOURCE_ID` nor `NOTION_DB_ID` is usable, the exporter will exit with an error.
- `.env` should remain local and must not be committed.

## Recommended Way To Run

Use the included launcher:

```bash
./sync_article
```

Run it from the project root:

```bash
cd Notion-Markdown-Exporter
./sync_article
```

## What `./sync_article` Does

The launcher is intended to remove setup friction for users running the exporter locally.

It will:

- detect `python3` or `python`
- reuse the local `.venv` if it already exists
- create `.venv` in the project if it does not exist
- verify that `pip` is available
- install dependencies from `requirements.txt`
- run `sync_notion_article.py`

If Python is missing, it prints a clear message and exits.

If `pip` is missing or unusable in the selected environment, it prints a clear message and exits.

If dependency installation fails for another reason, it reports that failure and stops.

## Interactive Flow

Once the exporter starts successfully, the interaction is:

1. The script loads candidate articles from Notion.
2. It prints a numbered list of exportable articles.
3. It prompts you to choose one or more articles.
4. It prompts you to choose an output location.
5. It exports each selected article.
6. It prints a short summary showing how many files were written and where.

## Selecting Articles

At the article prompt, you can choose:

- a single article number: `1`
- multiple specific articles: `1,3,5`
- a range: `2-4`
- a mixed selection: `1,3-5,8`

Rules:

- selections are validated before export starts
- invalid tokens are rejected cleanly
- out-of-range numbers are rejected
- reversed ranges such as `5-2` are rejected
- duplicates are removed while preserving the user’s selection order

Prompt example:

```text
Enter article number(s) to export (example: 1,3-5) or 'q' to quit:
```

To quit at this stage, enter:

```text
q
```

## Choosing The Output Directory

After article selection, the exporter prompts for an output destination.

Available choices:

```text
1. Current directory
2. Choose existing folder in current directory
3. Create new folder in current directory
q. Quit
```

Behavior:

- `Current directory` writes files directly into the directory where you launched the exporter
- `Choose existing folder` lists subdirectories in the current working directory and lets you pick one
- `Create new folder` creates a new directory inside the current working directory and writes files there
- entering `q` aborts cleanly without exporting anything

Constraints:

- destination selection is scoped to the current working directory
- new folder creation accepts a folder name only, not a path
- if the requested folder already exists and is a directory, it can be used
- if no valid destination is selected, the exporter does nothing

## Output Behavior

The exporter preserves its existing Markdown and front matter generation logic.

It does not change:

- slug generation
- front matter schema
- Notion block rendering behavior
- nested content fetch/render behavior
- table rendering
- list rendering
- article query/filter behavior

The only output-path change is that files are written to the directory you select at runtime instead of being forced into `_articles`.

## Canceling

The exporter handles normal cancellation cleanly.

- Enter `q` at the selection prompt to abort
- Enter `q` at the output-directory prompt to abort
- Press `Ctrl+C` during interactive prompts to exit without a Python traceback

## Troubleshooting

### Python not found

If `./sync_article` reports that Python is missing:

- install Python 3
- make sure `python3` or `python` is on your `PATH`
- run `./sync_article` again

### pip not available

If `./sync_article` reports that `pip` is unavailable:

- install Python with `venv` and `pip` support
- verify `python3 -m pip --version`
- run `./sync_article` again

### Notion API failures

If the exporter starts but fails while loading articles:

- confirm network access to `api.notion.com`
- verify your Notion token is valid
- confirm the integration has access to the target database
- verify `NOTION_DB_ID` and `NOTION_DATA_SOURCE_ID`

### Security notes

- Keep `.env` local and out of version control.
- Do not paste live integration tokens into README files, code comments, or committed config files.
- If a token is ever exposed outside your machine or copied into a tracked file, rotate it in Notion immediately and replace the local value.

### No pages found

If the script reports no pages found:

- verify the database or data source contains pages
- confirm you are pointing at the correct database or data source

## Repository Layout

- `sync_article`: workspace launcher
- `sync_notion_article.py`: exporter implementation
- `requirements.txt`: Python dependencies
- `.env.example`: example environment file

## Typical Usage

```bash
cd Notion-Markdown-Exporter
cp .env.example .env
# edit .env with your real Notion values
./sync_article
```
