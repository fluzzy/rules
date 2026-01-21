# Vercel Agent Skills

> **Source**: https://github.com/vercel-labs/agent-skills
> **Archive Date**: 2026-01-19

A collection of skills that extend AI coding agent capabilities. Follows the Agent Skills format; once installed, agents automatically use them when detecting relevant tasks.

## Installation

```bash
npx add-skill vercel-labs/agent-skills
```

## Skill List

### 1. react-best-practices

React and Next.js performance optimization guidelines.

**Features:**

- 40+ rules, categorized into 8 areas
- Prioritized by impact level

**When to use:**

- Writing React components or Next.js pages
- Implementing client/server-side data fetching
- Reviewing code for performance issues
- Optimizing bundle size/loading time

**Categories:**

- Waterfall elimination
- Bundle optimization
- Server performance
- Client data fetching
- Re-render optimization

### 2. web-design-guidelines

Review whether UI code follows web interface best practices.

**Features:**

- Audits 100+ rules
- Covers accessibility, performance, UX

**When to use:**

- "Review my UI"
- "Check accessibility"
- "Run best practices check"

**Categories:**

- Accessibility
- Focus states
- Forms
- Animations
- Typography
- Images
- Dark mode
- Touch interactions

### 3. vercel-deploy-claimable

Deploy applications to Vercel instantly.

**Features:**

- Auto-detects 40+ frameworks
- Returns preview URL and claim URL
- Auto-handles static HTML

**When to use:**

- "Deploy my app"
- "Deploy to production"

**How it works:**

1. Package project tarball
2. Detect framework
3. Upload
4. Return URLs

## License

MIT License
