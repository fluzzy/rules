# Gemini Voyager

> **Source**: [GitHub](https://github.com/Nagi-ovo/gemini-voyager) | [Website](https://voyager.nagi.fun/en) | [Discussion](https://news.hada.io/topic?id=26548)
> **Author**: Jesse Zhang (@Nagi-ovo)
> **License**: GPL-3.0
> **Archive Date**: 2026-02-10

All-in-one browser extension that enhances Google Gemini and AI Studio with organization, productivity, and export features.

## Overview

Browser extension (Chrome, Firefox, Edge, Safari planned) that adds folder management, timeline navigation, prompt vault, chat export, and power tools to Google Gemini. 5.3k+ GitHub stars.

## Features

### Shared (Gemini & AI Studio)

| Feature | Description |
| --- | --- |
| Folder Organization | Two-level hierarchy with drag-and-drop, custom colors (Gemini only) |
| Prompt Vault | Save and reuse prompts across Gemini, AI Studio, and custom websites |
| Cloud Sync | Synchronize folders and prompts to Google Drive |
| Formula Copy | One-click LaTeX and MathML (Word) source code copying |

### Gemini-Exclusive

| Feature | Description |
| --- | --- |
| Timeline Navigation | Visual nodes for message jumping, starring, conversation branch management |
| Chat Export | Export to JSON, Markdown, PDF with image support |
| Mermaid Rendering | Auto-render flowcharts and sequence diagrams |
| Markdown Fix | Corrects broken bold syntax from Gemini's HTML injection |
| NanoBanana | Lossless watermark removal for AI-generated images |
| Deep Research | Extract thinking processes and research links |

### Power Tools

| Feature | Description |
| --- | --- |
| Batch Delete | Delete multiple conversations at once |
| Quote Reply | Reply with text selection context |
| Tab Title Sync | Sync tab title with conversation title |
| Input Collapse | Collapse input area for more reading space |
| Default Model | Set a default model selection |
| Hide Recent Items | Hide the recent items sidebar |

## Installation

### Browser Extension Stores

- **Chrome Web Store** (also works on Edge, Opera, Brave, Vivaldi, Arc)
- **Firefox Add-ons**
- **Microsoft Edge Add-ons**
- **Safari** — coming soon

### Development Build

```bash
bun i                    # Install dependencies
bun run dev:chrome       # Chrome development mode
bun run dev:firefox      # Firefox development mode
bun run dev:safari       # Safari development (macOS only)
bun run build:chrome     # Production build for Chrome
bun run build:all        # Production build for all browsers
```

## Tech Stack

- **TypeScript** + **Vite** build system
- **Bun** package manager
- **Vitest** for testing
- **ESLint** + **Prettier** for code quality
- Separate Vite configs per browser target
- Browser extension APIs (content scripts, background workers, storage)

## Supported Languages

English, Chinese (Simplified/Traditional), Japanese, French, Spanish, Portuguese, Russian, Arabic, Korean

## Related Projects

- **DeepSeek Voyager** — fork adapted for DeepSeek
- **ChatGPT Conversation Timeline** — original inspiration for the timeline concept
