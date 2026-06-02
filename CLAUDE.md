# CLAUDE.md – Knowledge Wiki Configuration

## Purpose

This file defines the structure, conventions, and workflows for maintaining a persistent, LLM‑maintained knowledge wiki within this project. It acts as the schema that guides the LLM in ingesting documentation, organizing insights, and synthesizing answers.

## Wiki Layout

- **Raw Sources** – Immutable documents placed in `raw/` (e.g., articles, PDFs, images). Never modified by the LLM.
- **Wiki Directory** – Markdown files that constitute the living knowledge base. Includes:
  - `index.md` – Catalog of all wiki pages with links and brief summaries.
  - `log.md` – Append‑only chronological log of ingests, queries, and maintenance actions.
  - Individual page files (e.g., `azure-integration.md`, `test-automation-patterns.md`) organized by topic.

## Ingest Workflow

1. **Add Source** – Place a new source file in `raw/`.
2. **Process** – LLM reads the source, extracts key points, and discusses takeaways with the user.
3. **Update Wiki** – LLM creates or revises relevant wiki pages:
   - Generates or revises `index.md` entries.
   - Updates affected entity/concept pages.
   - Appends an entry to `log.md` with timestamp, source title, and brief description.
4. **Cross‑Reference** – Ensure all new content links to existing pages where appropriate and checks for contradictions.

## Maintenance & Linting

- Periodically run a **lint** pass where the LLM reviews the wiki for:
  - Contradictions or stale claims.
  - Orphan pages with no inbound links.
  - Missing cross‑references.
  - Opportunities for new sources or queries.
- Resolve findings promptly and log actions in `log.md`.

## Query Handling

- When answering user questions, the LLM:
  - Searches `index.md` to locate relevant pages.
  - Reads the pertinent wiki files.
  - Synthesizes an answer with citations.
  - Optionally creates a new wiki page to capture valuable outputs (e.g., comparisons, analyses).

## Conventions

- All markdown files use UTF‑8 encoding and follow the project's style guide (headings, code fences, list formatting).
- Page titles match the file name (e.g., `azure-integration.md` → `# Azure Integration`).
- Use `[[Link]]` style internal links for cross‑referencing other wiki pages.
- Preserve existing tags/metadata in `index.md` when adding or modifying entries.

## CLI / MCP Tools (Optional)

- Build small search utilities to query the wiki efficiently as it grows.
- Expose the wiki operations via MCP servers for integration with other tools.

*This configuration enables a self‑maintaining knowledge base where the LLM handles the bookkeeping, cross‑linking, and incremental synthesis, while the user curates sources and guides high‑level strategy.*
