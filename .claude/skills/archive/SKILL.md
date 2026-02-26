---
name: archive
description: "Archives external documents, blog posts, links, and PDFs into this repository as structured markdown files. Use when the user says \"archive this\", \"save this\", \"아카이빙해\", \"이거 읽고 정리해\", \"이거 추가해\", or provides a URL/document to be preserved. Handles content fetching, summarization, file placement, and README index updates."
argument-hint: "[url-or-path]"
---

# Archive

Read external content (URLs, PDFs, documents) and archive it as a well-structured markdown file in the appropriate location within this repository.

## Critical

- Each source document gets its own separate md file — NEVER merge multiple sources into one
- File names MUST be kebab-case: `descriptive-topic-name.md`
- ALWAYS update the relevant `README.md` index after creating the file
- All content MUST be written in English

## Steps

### 1. Fetch and Read Content

- For URLs: use `WebFetch` to retrieve the content
- For PDFs: use `Read` with the file path. For large PDFs (10+ pages), read in chunks using the `pages` parameter (e.g., `pages: "1-20"`). Max 20 pages per request.
- For local files: use `Read` directly
- Extract all key points, structure, examples, and recommendations

### 2. Determine Target Location

Read `references/repo-structure.md` to understand the repository layout.

**Decision logic:**
1. Identify the content type (guide, rule, MCP doc, showcase, skill)
2. Identify the domain (frontend, backend, app, devops, data, vscode-extension, or cross-domain)
3. If cross-domain → use root-level directory (`guides/`, `mcp/`, `rules/`, `showcase/`)
4. If domain-specific → use `<domain>/<category>/`
5. If no fitting domain exists and the content clearly warrants one → create a new domain directory with the standard sub-category layout and a README.md

**Category mapping (existing):**
- Technical how-to, best practices, research → `guides/`
- Coding conventions, rules, standards → `rules/`
- MCP server documentation → `mcp/`
- Configuration examples, reference implementations → `showcase/`
- Agent skill definitions → `skills/`

**If content does not fit any existing category**, create a new one. Examples:
- Interview/essay/opinion pieces → `essays/`
- Cheatsheets, quick references → `cheatsheets/`
- Tool comparisons, benchmarks → `benchmarks/`

When creating a new category: create the directory, add a `README.md` index, and update the root `README.md` to list it.

### 3. Create the Archive File

Follow the format in `references/archive-format.md`.

**File naming**: derive from the content title or topic, kebab-case, descriptive.
- Good: `agents-md-stop-using-init.md`, `react-server-components-guide.md`
- Bad: `blog-post.md`, `article1.md`, `temp.md`

### 4. Update README Index

Find the `README.md` in the target directory and add an entry to the file table.

Match the existing table format. Typical format:

```markdown
| [filename.md](./filename.md) | Description | [Source Name](url) |
```

### 5. Verify

- Confirm the file was created at the correct path
- Confirm the README index was updated
- Confirm the content follows the archive format template
