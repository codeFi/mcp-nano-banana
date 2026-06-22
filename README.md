# mcp-nano-banana

MCP server for AI-powered image generation using Google Gemini Nano Banana models. Generate, edit, and combine images from text prompts — directly from your AI coding assistant.

![npm version](https://img.shields.io/badge/version-1.2.0-blue.svg)
![license](https://img.shields.io/badge/license-MIT-green.svg)

## Features

- **Text-to-image generation** — Create images from natural language descriptions
- **Image editing** — Upload images and modify them with text instructions
- **Multi-image combination** — Mix up to 14 reference images into one composite
- **Resolution control** — Generate at 512px, 1K, 2K, or 4K resolution
- **13+ aspect ratios** — From square (1:1) to ultra-wide (21:9) and tall (1:8)
- **Auto-fallback** — Pro model falls back to Flash on rate limits (429/403/402)
- **File output** — Save to specific paths or return inline base64
- **Favicon generation** — Generate complete favicon packages with one command
- **Multi-format output** — PNG, JPEG, and WebP support

## Quick Start

### Prerequisites

1. **Get a Gemini API key** from [Google AI Studio](https://aistudio.google.com/apikey)
2. **Enable billing** on your Google Cloud project — image generation requires a paid tier (~$0.04-0.05/image for NB2 at standard resolution)

### Installation

```bash
npm install -g mcp-nano-banana
```

Set your API key:

```bash
export GEMINI_API_KEY=your-api-key
```

For MCP Registry publication metadata, the registry name is `io.github.codefi/mcp-nano-banana`.

## MCP Configuration

### Qwen Code

Add to `~/.qwen/settings.json`:

```json
{
  "mcpServers": {
    "nano-banana": {
      "command": "npx",
      "args": ["-y", "mcp-nano-banana@latest"],
      "env": {
        "GEMINI_API_KEY": "your-api-key"
      }
    }
  }
}
```

### Claude Code

```bash
claude mcp add nano-banana -e GEMINI_API_KEY=your-api-key -- npx -y mcp-nano-banana@latest
```

Or add manually to `~/.claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "nano-banana": {
      "command": "npx",
      "args": ["-y", "mcp-nano-banana@latest"],
      "env": {
        "GEMINI_API_KEY": "your-api-key"
      }
    }
  }
}
```

### Local / Development Build

```bash
git clone https://github.com/codeFi/mcp-nano-banana.git
cd mcp-nano-banana
npm install
npm run build
```

Then register with your MCP client:

```json
{
  "mcpServers": {
    "nano-banana": {
      "command": "node",
      "args": ["/path/to/mcp-nano-banana/dist/index.js"],
      "env": {
        "GEMINI_API_KEY": "your-api-key"
      }
    }
  }
}
```

## Models

| Model | MCP Name | Description | Max Ref Images |
|-------|----------|-------------|----------------|
| `gemini-3-pro-image-preview` | Nano Banana Pro | High quality, thinking mode, professional assets | 6 object + 5 character |
| `gemini-3.1-flash-image-preview` | Nano Banana 2 | Fast, high-efficiency, 1:4/4:1/1:8/8:1 ratios | 10 object + 4 character |
| `gemini-2.5-flash-image` | Nano Banana | Fastest, optimized for high-volume tasks | - |

The server defaults to **Nano Banana Pro** for best quality and automatically falls back to Flash on rate limits.

## Available Parameters

### Resolution

| Value | Approx. Pixels | Supported Models |
|-------|---------------|------------------|
| `"512"` | ~512px | Nano Banana 2 only |
| `"1K"` | ~1024px | All models |
| `"2K"` | ~2048px | All models |
| `"4K"` | ~4096px | All models |

Default: `"1K"`

### Aspect Ratio

| Value | Dimensions at 1K | Common Use Case |
|-------|-----------------|-----------------|
| `1:1` | 1024×1024 | Social posts, icons |
| `16:9` | 1344×768 | Headers, presentations |
| `9:16` | 768×1344 | Stories, mobile |
| `4:3` | 1152×896 | Desktop, photos |
| `3:4` | 896×1152 | Portraits |
| `2:3` | ~896×1344 | Print, magazines |
| `3:2` | ~1344×896 | Photos |
| `4:5` | ~896×1120 | Instagram posts |
| `5:4` | ~1120×896 | Instagram landscape |
| `21:9` | ~2048×878 | Ultra-wide banners |
| `1:4` | ~512×2048 | Nano Banana 2 only — tall strips |
| `4:1` | ~2048×512 | Nano Banana 2 only — wide strips |
| `1:8` | ~512×4096 | Nano Banana 2 only — extra tall |
| `8:1` | ~4096×512 | Nano Banana 2 only — extra wide |

## Tools

### `generate_image`

Generate or edit images from text prompts.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `prompt` | string | ✅ Yes | — | Text description or editing instructions |
| `imagePaths` | string[] | No | — | Reference image paths on disk (up to 14) |
| `model` | string | No | `gemini-3-pro-image-preview` | Model to use |
| `aspectRatio` | string | No | `1:1` | See table above for all options |
| `resolution` | string | No | `1K` | `512`, `1K`, `2K`, or `4K` |
| `outputFormat` | string | No | `png` | `png`, `jpeg`, `webp` |
| `outputPath` | string | No | — | Save image to this file path |

#### Examples

**Generate from text:**
```json
{
  "prompt": "A sunset over mountains with dramatic lighting",
  "aspectRatio": "16:9",
  "resolution": "2K",
  "outputPath": "/Users/razvan/Desktop/sunset.png"
}
```

**Edit an existing image:**
```json
{
  "prompt": "Change the background to a coffee shop interior",
  "imagePaths": ["/Users/razvan/photo.jpg"],
  "outputPath": "/Users/razvan/edited.png"
}
```

**Combine multiple images:**
```json
{
  "prompt": "Put these glasses on this person's face",
  "imagePaths": ["/path/to/face.jpg", "/path/to/glasses.jpg"],
  "outputPath": "/path/to/result.png"
}
```

**High-res wallpaper:**
```json
{
  "prompt": "A futuristic cityscape at night with neon lights and rain",
  "aspectRatio": "16:9",
  "resolution": "4K",
  "outputPath": "/path/to/wallpaper.png"
}
```

### `generate_favicons`

Generate a complete favicon package from a prompt or existing image.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `prompt` | string | No\* | — | Icon description (required if `imagePath` not set) |
| `imagePath` | string | No\* | — | Path to existing master icon image |
| `model` | string | No | `gemini-3-pro-image-preview` | Model for prompt generation |
| `outputDir` | string | No | — | Directory for individual favicon files |
| `outputPath` | string | No | — | Save ZIP package to this path |
| `returnZipBase64` | boolean | No | `false` | Return ZIP as base64 text |
| `appName` | string | No | `My App` | Manifest `name` |
| `shortName` | string | No | `App` | Manifest `short_name` |
| `themeColor` | string | No | `#ffffff` | Manifest theme color |
| `backgroundColor` | string | No | `#ffffff` | Manifest background color |
| `includeReadme` | boolean | No | `true` | Include README.md in package |

#### Example

```json
{
  "prompt": "A minimalist banana icon with a soft gradient",
  "outputDir": "/path/to/favicon",
  "outputPath": "/path/to/favicon.zip",
  "appName": "MyBrand",
  "shortName": "Brand",
  "themeColor": "#1a1a2e",
  "backgroundColor": "#ffffff"
}
```

**Generated files:**
- `favicon.ico` — Multi-size icon (16/32/48px)
- `favicon-16x16.png` — Browser tabs
- `favicon-32x32.png` — Higher-res tabs
- `apple-touch-icon.png` — iOS home screen
- `android-chrome-192x192.png` — Android
- `android-chrome-512x512.png` — Android HD
- `site.webmanifest` — PWA manifest

**HTML snippet (included in response):**
```html
<link rel="icon" href="/favicon.ico" sizes="any">
<link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png">
<link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png">
<link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png">
<link rel="manifest" href="/site.webmanifest">
```

## Pricing

Image generation requires a **paid Google AI Studio account**. Approximate costs:

| Model | 1K | 2K | 4K |
|-------|-----|-----|-----|
| Nano Banana 2 | ~$0.04 | ~$0.08 | ~$0.15 |
| Nano Banana Pro | ~$0.06 | ~$0.13 | ~$0.24 |

You only pay for what you generate — no monthly minimum.

## Supported Image Formats

**Input (reference images):** `png`, `jpg`, `jpeg`, `webp`, `gif`
**Output:** `png`, `jpeg`, `webp`

## Development

```bash
# Build TypeScript
npm run build

# Watch mode
npm run dev

# Run tests
npm test

# Run the server directly
node dist/index.js
```

## Release

First release flow:

1. Update the version in `package.json` and keep `server.json` in sync.
2. Commit, tag the release as `v<version>`, and push the tag.
3. Let GitHub Actions publish the npm package with `NPM_TOKEN`.
4. If you need to regenerate registry scaffolding, run `mcp-publisher init`; otherwise validate the existing `server.json`.
5. Authenticate locally with `mcp-publisher login github`.
6. Publish to the MCP Registry with `mcp-publisher publish`.
7. Verify npm and MCP Registry resolution after publish.

Quick checks:

```bash
npm test
npm run build
npm pack --dry-run
curl "https://registry.modelcontextprotocol.io/v0.1/servers?search=io.github.codefi/mcp-nano-banana"
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `GEMINI_API_KEY is not set` | Export the env var or add it to your MCP config |
| `429 Too Many Requests` | Server auto-falls back to Flash model. Add billing to your account. |
| `403/402 Quota exceeded` | Enable billing at [Google Cloud Console](https://console.cloud.google.com/billing) |
| `503 Unavailable` | High demand — retry in a moment. Common with 4K requests. |
| Image not updating after edit | Make sure you rebuilt the TypeScript (`npm run build`) |
| `imagePaths` not found | Use absolute paths. Relative paths may not resolve correctly. |

## Architecture

```
AI Assistant (MCP Client)
       │  (MCP Protocol over stdio)
       ▼
mcp-nano-banana Server
       │  (HTTPS POST)
       ▼
Google Gemini API
       │
       ▼
Generated Image → saved to outputPath (or returned inline base64)
```

## License

[MIT](LICENSE)

## Author

[codeFi](https://github.com/codeFi)
