# Vercel Deploy Claimable

> **Source**: [GitHub](https://github.com/vercel-labs/agent-skills/tree/main/skills/claude.ai/vercel-deploy-claimable)
> **Author**: Vercel
> **License**: MIT
> **Archive Date**: 2026-02-26

A Claude Code skill that deploys web projects to Vercel instantly without authentication. Returns a live preview URL and a claim URL for transferring the deployment to a personal Vercel account.

---

## How to Use

### Claude Code (Skills Platform)

```bash
/plugin marketplace add vercel-labs/agent-skills
/plugin install vercel-deploy-claimable@vercel-agent-skills
```

### Direct Invocation

```bash
# Deploy current directory
bash /mnt/skills/user/vercel-deploy/scripts/deploy.sh

# Deploy a specific directory
bash /mnt/skills/user/vercel-deploy/scripts/deploy.sh /path/to/project

# Deploy a pre-built tarball
bash /mnt/skills/user/vercel-deploy/scripts/deploy.sh /path/to/project.tgz
```

### Manual Setup

```bash
# Download the deploy script
curl -o deploy.sh https://raw.githubusercontent.com/vercel-labs/agent-skills/main/skills/claude.ai/vercel-deploy-claimable/scripts/deploy.sh
chmod +x deploy.sh
```

---

## When to Apply

- Quick deployment of prototypes and demos without Vercel authentication
- Sharing a live preview URL during pair programming or code review
- AI agent workflows that need to deploy and return a URL to the user
- Deploying static sites, SPAs, or full-stack apps for instant feedback
- Claimable deployments that users can later transfer to their own Vercel account

---

## How It Works

### Deployment Flow

1. **Package** -- The script creates a `.tgz` tarball of the project, excluding `node_modules/` and `.git/`
2. **Detect Framework** -- Scans `package.json` dependencies to identify the framework (50+ supported)
3. **Upload** -- Sends the tarball via `POST` to `https://claude-skills-deploy.vercel.com/api/deploy`
4. **Return URLs** -- Returns a preview URL (live site) and a claim URL (ownership transfer)

### Output Format

The script produces two outputs:

| Output | Description |
| --- | --- |
| Human-readable text | Confirmation message with preview and claim URLs |
| JSON (stdout) | Structured data for programmatic use |

**JSON response fields:**

| Field | Description |
| --- | --- |
| `previewUrl` | Live deployment URL |
| `claimUrl` | URL to transfer deployment to a Vercel account |
| `deploymentId` | Unique deployment identifier |
| `projectId` | Unique project identifier |

### Static Site Handling

For projects without `package.json`, the script:
- Detects static HTML files
- Automatically renames a single HTML file to `index.html`
- Sets framework to `null` for static serving

---

## Supported Frameworks

| Category | Frameworks |
| --- | --- |
| React | Next.js, Gatsby, Create React App, Remix, React Router |
| Vue | Nuxt, VitePress, VuePress, Gridsome |
| Svelte | SvelteKit, Svelte |
| Angular | Angular |
| Backend | Express, NestJS, and others |
| Build Tools | Vite, Astro, Parcel, and others |
| Static | Plain HTML (no `package.json` required) |

---

## Configuration

### Network Requirements

If deployment fails due to network restrictions, add `*.vercel.com` to the allowed domains in your environment settings.

### Script Dependencies

The deploy script uses standard Unix utilities only:

- `tar` -- project packaging
- `curl` -- HTTP upload
- `grep` -- response parsing

No additional dependencies or authentication tokens required.

---

## References

- [vercel-labs/agent-skills (Full Source)](https://github.com/vercel-labs/agent-skills/tree/main/skills/claude.ai/vercel-deploy-claimable)
- [Vercel Documentation](https://vercel.com/docs)
- [Vercel CLI](https://vercel.com/docs/cli)
