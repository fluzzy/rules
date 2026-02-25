# Archive

A curated collection of rules, guides, skills, and resources for coding agents.

## Structure

Resources are organized by **domain**, each containing the same sub-categories:

| Sub-category | Description |
| --- | --- |
| `skills/` | Agent skill definitions |
| `mcp/` | MCP server documentation |
| `rules/` | Coding rules and conventions |
| `guides/` | How-to guides and best practices |
| `showcase/` | Configuration examples |

```
<domain>/
├── skills/
├── mcp/
├── rules/
├── guides/
└── showcase/
```

## Domains

| Domain | Description | Contents |
| --- | --- | --- |
| [frontend/](./frontend/) | React, Next.js, UI/UX, etc. | 3 docs |
| [backend/](./backend/) | API, DB, infrastructure, etc. | - |
| [app/](./app/) | Mobile, desktop, etc. | 5 docs |
| [devops/](./devops/) | CI/CD, cloud, infrastructure, etc. | - |
| [data/](./data/) | ML, analytics, pipelines, etc. | - |
| [vscode-extension/](./vscode-extension/) | VS Code extension development | 22 docs |

## Cross-domain Resources

Resources not tied to a specific domain.

| Directory | Description |
| --- | --- |
| [guides/](./guides/) | How-to guides and best practices |
| [mcp/](./mcp/) | MCP server documentation |
| [rules/](./rules/) | Coding rules and conventions |
| [showcase/](./showcase/) | Configuration examples |
| [skills/](./skills/) | Agent skill definitions |

## Roles

Standalone Claude role prompts for specialized personas. See [role/](./role/) for all available roles.

| Role | Tech Stack |
| --- | --- |
| [CTO](./role/cto.md) | General |
| [Debugger](./role/debugger.md) | DevTools |
| [Refactoring Expert](./role/refactoring-expert.md) | General |
| [React Performance Engineer](./role/react-performance-engineer.md) | React, Next.js |
| [Frontend Code Reviewer](./role/frontend-code-reviewer.md) | Frontend |
| [Frontend UX Engineer](./role/frontend-ux-engineer.md) | WCAG, ARIA |
| [Three.js Creative Developer](./role/threejs-creative-developer.md) | Three.js, WebGL, GSAP |
| [RESTful API Designer](./role/restful-api-designer.md) | REST, OpenAPI, Zod |
| [Jest Test Engineer](./role/jest-test-engineer.md) | Jest, Vitest, Playwright |
| [Semantic SEO Expert](./role/semantic-seo-expert.md) | HTML, Schema.org, JSON-LD |
| [Senior Product Planner](./role/senior-product-planner.md) | General |

## Contributing

- Add domain-specific resources under the appropriate `<domain>/<category>/` directory
- Add cross-domain resources under the root-level directories (`guides/`, `mcp/`, `rules/`, `showcase/`, `skills/`)
- Add new roles using [role/_template.md](./role/_template.md)
- Update the relevant README when adding new files
