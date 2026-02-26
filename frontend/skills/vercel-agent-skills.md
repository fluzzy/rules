# Vercel Agent Skills

> **Source**: [GitHub](https://github.com/vercel-labs/agent-skills/tree/main/skills)
> **Author**: Vercel
> **License**: MIT
> **Archive Date**: 2026-02-26

A collection of AI coding agent skills from Vercel Labs. Each skill follows the Agent Skills format — once installed, agents automatically apply them when detecting relevant tasks. 5 skills across frontend, mobile, design, and deployment.

---

## Installation

```bash
npx add-skill vercel-labs/agent-skills
```

### Claude Code

```bash
/plugin marketplace add vercel-labs/agent-skills
/plugin install <skill-name>@vercel-agent-skills
```

### Cursor

Settings → Rules → Add Rule → Remote Rule (GitHub):

```
https://github.com/vercel-labs/agent-skills.git
```

---

## Skill List

| # | Skill | Domain | Rules | Impact |
|---|-------|--------|-------|--------|
| 1 | [react-best-practices](./vercel-react-best-practices.md) | Frontend (React/Next.js) | 57 rules, 8 categories | Waterfalls, bundle size, re-renders |
| 2 | [react-native-skills](../../app/skills/vercel-react-native-skills.md) | App (React Native/Expo) | 35+ rules, 13 categories | Rendering, lists, animation, state |
| 3 | [composition-patterns](./vercel-composition-patterns.md) | Frontend (React) | 8 patterns, 4 categories | Component architecture, scalability |
| 4 | [web-design-guidelines](./vercel-web-design-guidelines.md) | Frontend (UI/UX) | 100+ rules, 16 categories | Accessibility, performance, UX |
| 5 | [vercel-deploy-claimable](../../devops/skills/vercel-deploy-claimable.md) | DevOps | Deploy script | Instant deploy, 50+ frameworks |

### 1. react-best-practices

React and Next.js performance optimization. 57 rules across 8 categories prioritized by impact: waterfall elimination, bundle optimization, server performance, client data fetching, re-render optimization, rendering patterns, JS performance, and advanced patterns.

### 2. react-native-skills

Mobile performance optimization for React Native and Expo. 35+ rules covering rendering crash prevention, list/scroll performance, Reanimated animation, navigation, state management, UI patterns, design system, and monorepo setup.

### 3. composition-patterns

React component architecture and scalable design patterns. 8 patterns including boolean prop elimination, compound components, generic context interfaces, state/UI decoupling, provider state lifting, explicit variants, children preference, and React 19 API updates.

### 4. web-design-guidelines

Web interface audit skill with 100+ rules across 16 categories: accessibility, focus states, forms, animation, typography, images, dark mode, touch interactions, navigation, performance, hydration, content/copy, and internationalization.

### 5. vercel-deploy-claimable

Instant deployment to Vercel without authentication. Auto-detects 50+ frameworks, returns preview URL and claim URL. Packages project as tarball, uploads, and returns deployment URLs.

---

## References

- [vercel-labs/agent-skills (Full Source)](https://github.com/vercel-labs/agent-skills)
- [Skills Directory](https://github.com/vercel-labs/agent-skills/tree/main/skills)
