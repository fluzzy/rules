# Repository Structure

## Layout

```
rules/
├── frontend/          # Frontend (React, Next.js, UI/UX)
├── backend/           # Backend (API, DB, infrastructure)
├── app/               # App development (mobile, desktop)
├── devops/            # DevOps (CI/CD, cloud, infrastructure)
├── data/              # Data (ML, analytics, pipelines)
├── vscode-extension/  # VS Code extension development
├── role/              # Claude role definitions
├── guides/            # Cross-domain guides and best practices
├── mcp/               # Cross-domain MCP server docs
├── rules/             # Cross-domain coding rules
├── showcase/          # Cross-domain configuration examples
├── skills/            # Cross-domain agent skill definitions
└── .claude/skills/    # Active Claude Code skills (auto-loaded)
```

## Sub-categories within each domain

Each domain directory uses these sub-categories:

- `skills/` - Agent skill definitions
- `mcp/` - MCP server docs
- `rules/` - Coding rules
- `guides/` - Guides and best practices
- `showcase/` - Configuration examples

## Conventions

- Every directory has a `README.md` index listing its contents
- File names are kebab-case: `descriptive-topic-name.md`
- Each source document gets its own separate md file
- All README and structural docs are in English
- Content files may use Korean or English
