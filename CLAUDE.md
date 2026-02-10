# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal archive of rules, guides, and resources for AI coding agents. Markdown only, no executable code.

## Structure

Resources are organized by domain. Each domain shares the same sub-category layout.

**Domains:**

- `frontend/` - Frontend (React, Next.js, UI/UX)
- `backend/` - Backend (API, DB, infrastructure)
- `app/` - App development (mobile, desktop)
- `devops/` - DevOps (CI/CD, cloud, infrastructure)
- `data/` - Data (ML, analytics, pipelines)
- `vscode-extension/` - VS Code extension development
- `common/` - Cross-domain resources
- `role/` - Claude role definitions (copy-paste prompts)

**Sub-categories within each domain:**

- `skills/` - Agent skill definitions
- `mcp/` - MCP server docs
- `rules/` - Coding rules
- `guides/` - Guides and best practices
- `showcase/` - Configuration examples

## Conventions

- All README and structural docs are written in English
- Content files may use Korean or English
- Every directory has a `README.md` index listing its contents
- New roles use `role/_template.md` as a starting point
- New domain resources go under `<domain>/<category>/` and update the domain README
- Each source document (official docs, Context7, etc.) gets its own separate md file — never merge multiple sources into one
