# Gamma Portfolio Plugin for Claude Code

Resume-to-portfolio generator powered by the [Gamma API](https://gamma.app). Provide a resume and get a professionally designed portfolio webpage in seconds.

## Features

- Generates portfolio webpages, presentations, or documents from any resume
- Three style presets with optimized Gamma themes:
  - **Professional** (`commons` theme) — clean and trustworthy
  - **Casual** (`gamma` theme) — warm and approachable
  - **Creative** (`electric` theme) — bold and vibrant
- Korean language support by default
- Automatic PDF export
- Section-based card layout with `inputTextBreaks`

## Prerequisites

- [Claude Code](https://claude.ai/claude-code) v1.0.33+
- Gamma API key ([Pro, Ultra, Teams, or Business plan](https://gamma.app))

## Installation

In a Claude Code session, run:

```
/plugin add https://github.com/jinjin1/gamma-portfolio-plugin
```

Or for local testing:

```bash
claude --plugin-dir ./gamma-portfolio-plugin
```

## Usage

```
/gamma-portfolio:portfolio
```

The skill will guide you through:

1. **API key setup** — set `GAMMA_API_KEY` environment variable
2. **Resume input** — paste text or provide a file path (.txt, .md, .pdf)
3. **Style selection** — choose format (webpage/presentation/document) and style (professional/casual/creative)
4. **Generation** — calls Gamma API and polls for completion
5. **Result** — provides the portfolio URL and PDF download link

## API Key Setup

```bash
export GAMMA_API_KEY="your-api-key"
```

Get your API key from: Gamma > Account Settings > API Keys

## Gamma API Parameters Used

| Parameter | Value |
|-----------|-------|
| `textMode` | `generate` |
| `cardSplit` | `inputTextBreaks` |
| `textOptions.amount` | `medium` |
| `textOptions.language` | `ko` |
| `imageOptions.source` | `aiGenerated` |
| `exportAs` | `pdf` |

## License

MIT
