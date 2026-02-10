# Claude Code Power Tips: 70 Tips from Hackathon Winners

> **Source**: [ykdojo/claude-code-tips](https://github.com/ykdojo/claude-code-tips) | [Advent of Claude](https://adocomplete.com/advent-of-claude-2025/)
> **Authors**: ykdojo (Anthropic hackathon winner), Ado Kukic (Anthropic DevRel)
> **Archive Date**: 2026-02-10

Compiled 70 tips in 13 parts covering mindset, setup, context management, shortcuts, git workflows, advanced features, and real-world patterns.

## Part 1: Agentic Developer Mindset

- **Break problems into smaller sub-problems** — decompose incrementally, not one-shot
- **Choose the right abstraction level** — match detail to complexity
- **Balance planning with rapid prototyping** — sketch first, then build iteratively
- **Embrace iterative problem-solving** — explore incrementally in uncertain domains
- **Invest in personal workflow optimization** — refine CLAUDE.md, build scripts, learn features

## Part 2: Setup and Configuration

### CLAUDE.md and Rules

- `/init` generates `CLAUDE.md` with build/test commands, key directories, conventions
- `.claude/rules/` for modular, topic-specific instructions (auto-loaded as project memory)
- YAML frontmatter for conditional rules:

```yaml
---
paths: src/api/**/*.ts
---
# API Development Rules
- All API endpoints must include input validation
```

- `CLAUDE.md` = general project guide; `.claude/rules/` = focused supplements
- Tell Claude to update CLAUDE.md directly: "Update CLAUDE.md: always use bun instead of npm"

### @ Mentions

- `@src/auth.ts` — add files to context
- `@src/components/` — reference directories
- `@mcp:github` — enable/disable MCP servers
- ~3x faster in git repos, supports fuzzy matching

### Terminal Aliases

```bash
alias c='claude'
alias ch='claude --chrome'
alias gb='github-desktop'
alias co='code'
alias q='cd ~/projects'
```

### Key Distinctions

| Concept | Purpose |
| --- | --- |
| CLAUDE.md | Project-level instructions (always loaded) |
| Skills | On-demand specialized instructions |
| Slash commands | Built-in commands |
| Plugins | Extend functionality (bundle commands, agents, skills, hooks) |
| MCP servers | Integrate external tools |

## Part 3: Context Management

- **Start fresh conversations** for new topics — context degrades over length
- **`/compact`** to compress conversation; create HANDOFF.md before switching
- **`/context`** to see exactly what consumes tokens (system prompt, MCP, memory, skills, history)
- **Slim system prompt**: disable auto-compaction via `/config` to reclaim 45k context
- **Clone/fork**: copy context from old conversation to start fresh with clean state
- **Search history**: stored in `~/.claude/projects/`, searchable with grep

## Part 4: Keyboard Shortcuts

| Shortcut | Action |
| --- | --- |
| `!command` | Execute bash immediately (no model processing) |
| `Esc Esc` | Rewind conversation/code changes |
| `Ctrl+R` | Reverse search history |
| `Ctrl+S` | Stash current prompt (like `git stash`) |
| `Shift+Tab` x2 | Toggle plan mode |
| `Alt+P` / `Option+P` | Switch model |
| `Ctrl+O` | Toggle verbose mode |
| `Tab` / `Enter` | Accept prompt suggestion |

### Session Management

```bash
claude --continue          # Resume last conversation
claude --resume            # Pick a past session
claude --resume api-fix    # Resume by name
claude --teleport id       # Resume a web session
/rename api-migration      # Name current session
/export                    # Export to markdown
```

- Customize retention: `cleanupPeriodDays` (default 30, 0 to disable)

## Part 5: Git Workflows

- Use Git and GitHub CLI for commits, branching, PR management
- Create **draft PRs** for low-risk review
- **Git worktrees** for parallel branch development without switching directories
- **Interactive PR reviews**: have Claude examine diffs, suggest improvements, iterate

## Part 6: Advanced Features

### Hooks (Lifecycle Events)

Configure via `/hooks` or `.claude/settings.json`:

| Event | Description |
| --- | --- |
| `PreToolUse` / `PostToolUse` | Before/after tool execution |
| `PermissionRequest` | Auto approve/deny permissions |
| `Notification` | React to notifications |
| `SubagentStart` / `SubagentStop` | Monitor agent spawning |

Use cases: block dangerous commands, send notifications, log actions, integrate external systems. "Deterministic control over probabilistic AI."

### Agent Skills

- Folders of instructions, scripts, resources teaching Claude specialized tasks
- Packaged once, usable everywhere
- Open standard: https://agentskills.io

### Subagents

- Each gets its own 200k context window
- Run in parallel, merge output back to main agent
- Full access to MCP tools

### Plugins

```
/plugin install my-setup
```

Bundle commands, agents, skills, hooks, and MCP servers into one package.

### LSP Integration

Language Server Protocol support: instant diagnostics, code navigation (go to definition, find references), type information, documentation for symbols.

### Claude Agent SDK

```javascript
import { query } from '@anthropic-ai/claude-agent-sdk';

for await (const msg of query({
  prompt: "Generate markdown API docs for all public functions in src/",
  options: {
    allowedTools: ["Read", "Write", "Glob"],
    permissionMode: "acceptEdits"
  }
})) {
  if (msg.type === 'result') console.log("Docs generated:", msg.result);
}
```

## Part 7: Permissions and Safety

- **`/sandbox`**: Define permission boundaries once, Claude works freely inside
- **YOLO mode**: `claude --dangerously-skip-permissions` — isolated environments only
- **Audit approved commands** regularly for security

## Part 8: Testing Patterns

- **Containers**: Isolate risky or long-running operations
- **tmux write-test cycle**: Start sessions, send commands, capture output, verify results
- **TDD**: Write tests first for reliable code
- **Verification**: Automated tests, manual inspection, comparative analysis, staged rollouts

## Part 9: Browser and External Tools

- **`/chrome`**: Navigate pages, click buttons, fill forms, read console errors, inspect DOM, take screenshots
- **Voice transcription**: SuperWhisper, MacWhisper, or local models
- **Clipboard**: Cmd+A to copy inaccessible content (Reddit, Gmail) directly into context
- **Gemini CLI**: Fallback for sites WebFetch can't access

## Part 10: Thinking and Planning

### Plan Mode (Shift+Tab x2)

- Claude reads, searches, analyzes architecture, drafts plans — won't edit until approved
- "I default to plan mode 90% of the time"
- Latest version lets you provide feedback when rejecting plans

### Ultrathink

```
ultrathink: design a caching layer for our API
```

Allocates up to 32k tokens for internal reasoning. Only works when MAX_THINKING_TOKENS is not set.

## Part 11: Productivity Patterns

- **`/vim`**: Vim mode for prompt editing (h/j/k/l, ciw, dd, w/b, A)
- **`/stats`**: Usage dashboard with patterns and streaks
- **`/usage`**: Rate limit check with visual progress bars
- **Multiple terminal tabs**: Multitask during long operations
- **Background processes**: `&` operator and tmux for long-running tasks
- **Meta-automation**: Automate repetitive automation tasks

## Part 12: Headless Mode and CI/CD

```bash
claude -p "Fix the lint errors"
claude -p "List all functions" | grep "async"
git diff | claude -p "Explain these changes"
echo "Review this PR" | claude -p --json
```

### Reusable Commands

Create markdown files that become slash commands:

```
/daily-standup              # Run morning routine prompt
/explain $ARGUMENTS         # /explain src/auth.ts
```

## Part 13: Philosophy

> "The developers who get the most out of Claude Code aren't the ones who type 'do everything for me.' They're the ones who've learned when to use Plan mode, how to structure prompts, when to invoke ultrathink, and how to set up hooks that catch mistakes before they happen."

> "AI is a lever. These features help you find the right grip."

## Essential Commands Quick Reference

| Command | Purpose |
| --- | --- |
| `/init` | Generate CLAUDE.md |
| `/context` | View token consumption |
| `/stats` | Usage statistics |
| `/usage` | Check rate limits |
| `/vim` | Enable vim mode |
| `/config` | Open configuration |
| `/hooks` | Configure lifecycle hooks |
| `/sandbox` | Set permission boundaries |
| `/export` | Export to markdown |
| `/resume` | Resume past session |
| `/rename` | Name current session |
| `/compact` | Compress conversation |
| `/chrome` | Toggle browser integration |
| `/mcp` | Manage MCP servers |
