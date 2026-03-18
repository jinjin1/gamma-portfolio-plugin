# Gamma Portfolio Plugin for Claude Code

Resume-to-portfolio generator powered by the [Gamma API](https://gamma.app). Provide a resume and get a professionally designed portfolio webpage in seconds.

## Features

- Generates portfolio webpages, presentations, or documents from any resume
- Auto-detects resume language and outputs in the same language
- Three style presets with optimized Gamma themes:
  - **Professional** (`commons` theme)
  - **Casual** (`gamma` theme)
  - **Creative** (`electric` theme)
- Automatic PDF export

## Prerequisites

- [Claude Code](https://claude.ai/claude-code) v1.0.33+
- Gamma API key ([Pro, Ultra, Teams, or Business plan](https://gamma.app))

## Installation

In a Claude Code session:

```
/plugin marketplace add jinjin1/gamma-portfolio-plugin
```

Then install the plugin:

```
/plugin install gamma-portfolio@jinjin1-gamma-portfolio-plugin
```

### Local testing

```bash
git clone https://github.com/jinjin1/gamma-portfolio-plugin.git
claude --plugin-dir ./gamma-portfolio-plugin
```

## Usage

```
/gamma-portfolio:portfolio
```

The skill guides you through:

1. **API key setup** — set `GAMMA_API_KEY` environment variable
2. **Resume input** — paste text or provide a file path (.txt, .md, .pdf)
3. **Style selection** — choose format and style
4. **Generation** — calls Gamma API and polls for completion
5. **Result** — portfolio URL and PDF download link

## API Key Setup

```bash
export GAMMA_API_KEY="your-api-key"
```

Get your API key from: [Gamma](https://gamma.app) > Account Settings > API Keys

## License

MIT
