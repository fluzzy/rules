# Claude Roles

Copy the role you need and paste at the start of conversation to assign Claude a specific role.

## Available Roles

| Role | File | Tech Stack |
|------|------|------------|
| CTO | [cto.md](./cto.md) | General |
| Semantic SEO Expert | [semantic-seo-expert.md](./semantic-seo-expert.md) | HTML, Schema.org, JSON-LD |
| React Performance Engineer | [react-performance-engineer.md](./react-performance-engineer.md) | React, Next.js |
| Frontend Code Reviewer | [frontend-code-reviewer.md](./frontend-code-reviewer.md) | Frontend |
| Debugger | [debugger.md](./debugger.md) | DevTools |
| Refactoring Expert | [refactoring-expert.md](./refactoring-expert.md) | General |
| RESTful API Designer | [restful-api-designer.md](./restful-api-designer.md) | REST, OpenAPI, Zod |
| Jest Test Engineer | [jest-test-engineer.md](./jest-test-engineer.md) | Jest, Vitest, Playwright |
| Frontend UX Engineer | [frontend-ux-engineer.md](./frontend-ux-engineer.md) | WCAG, ARIA |
| Three.js Creative Developer | [threejs-creative-developer.md](./threejs-creative-developer.md) | Three.js, WebGL, GSAP |

## Usage

```
1. Open the role file you need
2. Copy the entire content
3. Paste at the beginning of your Claude conversation
4. Start your task
```

## Creating New Roles

Use [_template.md](./_template.md) as a starting point.

## Structure (Best Practices)

Based on Anthropic's prompt engineering guidelines:

```markdown
# Role: [Name]

[Identity + rejected excuses in one paragraph]

## Tone
[One line describing communication tone]

## Responsibilities
- [What this role does]

## Expertise
- [Specific skills and tools]

## Communication Style
- [How to respond]

## Constraints
- [What NOT to do]
```
