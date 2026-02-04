# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

NotionNext is a static blog system built with Next.js 13.3.1 and the Notion API, designed to be deployed on Vercel. It transforms Notion pages into a fully-featured blog with multiple theme support.

**Key Technologies:**
- Framework: Next.js 13.3.1 (Pages Router)
- Styling: Tailwind CSS
- Content Source: Notion API via `notion-client` and `notion-utils`
- Rendering: `react-notion-x` for Notion block rendering
- Package Manager: Yarn 1.22.22

## Development Commands

```bash
# Start development server
yarn dev

# Build for production
yarn build

# Start production server
yarn start

# Build and generate static export with sitemap
yarn export

# Analyze bundle size
yarn bundle-report
```

## Architecture

### Theme System

The project uses a **dynamic theme system** where themes are completely isolated and swappable:

- **Location**: `/themes/` directory contains 18+ themes (simple, hexo, medium, next, fukasawa, etc.)
- **Structure**: Each theme has its own `index.js`, `config.js`, `style.js`, and `components/` directory
- **Configuration**: Active theme is set via `THEME` in `blog.config.js` (default: 'simple')
- **Webpack Alias**: `next.config.js` dynamically maps `@theme-components` to the active theme directory
- **Theme Switching**: The `getLayoutByTheme()` function in `/themes/theme.js` loads the appropriate layout component

### Data Flow

1. **Notion as CMS**: Content is fetched from Notion using the page ID configured in `NOTION_PAGE_ID`
2. **Data Layer**: `/lib/db/getSiteData.js` provides `getGlobalData()` which fetches all site data
3. **Notion Utilities**: `/lib/notion/` contains helpers for fetching posts, metadata, categories, tags, and page properties
4. **Caching**: Uses `memory-cache` with configurable revalidation (`NEXT_REVALIDATE_SECOND`)
5. **SSG/ISR**: Pages use `getStaticProps` with Incremental Static Regeneration

### Page Structure

- **Pages Router**: Uses Next.js Pages Router (not App Router)
- **Dynamic Routes**:
  - `/pages/[prefix]/[...slug].js` - Main post/page handler
  - `/pages/category/[category].js` - Category pages
  - `/pages/tag/[tag].js` - Tag pages
  - `/pages/archive/` - Archive pages
  - `/pages/search/` - Search functionality
- **Layout Resolution**: Each page imports `getLayoutByTheme()` to render the theme-specific layout

### Configuration System

- **Main Config**: `blog.config.js` - Central configuration with 200+ options
- **Environment Variables**: All configs support `process.env.NEXT_PUBLIC_*` overrides
- **Multi-language**: Supports multiple Notion page IDs for different languages (format: `pageId,zh:pageId,en:pageId`)
- **Theme Configs**: Each theme has its own `config.js` for theme-specific settings

### Key Directories

- `/lib/` - Core utilities and business logic
  - `/lib/notion/` - Notion API integration
  - `/lib/cache/` - Caching layer
  - `/lib/plugins/` - Plugin system (analytics, comments, etc.)
  - `/lib/lang/` - Internationalization
  - `/lib/utils/` - Helper functions
- `/components/` - Shared React components
- `/themes/` - Theme implementations
- `/pages/` - Next.js pages (routing)
- `/public/` - Static assets
- `/styles/` - Global styles

## Important Patterns

### Adding a New Theme

1. Create directory in `/themes/[theme-name]/`
2. Add `index.js` (main layout component)
3. Add `config.js` (theme-specific configuration)
4. Add `components/` directory for theme components
5. Theme will be auto-detected by `next.config.js`

### Working with Notion Data

- Use `getGlobalData({ from, locale })` to fetch all site data
- Post filtering: `allPages.filter(page => page.type === 'Post' && page.status === 'Published')`
- Block content: `getPostBlocks(postId, 'slug', previewLines)`
- All Notion utilities are in `/lib/notion/`

### Configuration Access

```javascript
import { siteConfig } from '@/lib/config'
const value = siteConfig('CONFIG_KEY', defaultValue, notionConfig)
```

## Build & Deployment

- **Target Platform**: Vercel (optimized for)
- **Static Export**: Use `yarn export` for static hosting
- **RSS Generation**: Automatically generated during build if `ENABLE_RSS` is true
- **Sitemap**: Generated via `next-sitemap` during post-build
- **Image Optimization**: Next.js Image component with AVIF/WebP support

## Multi-language Support

- Configured via `i18n` in `next.config.js`
- Supports multiple Notion databases for different languages
- URL rewrites handle language prefixes (e.g., `/zh/`, `/en/`)
- Language detection from `NOTION_PAGE_ID` format

## Notes

- This is a **Pages Router** project, not App Router
- Theme components should be imported from `@theme-components` alias
- The project uses ISR (Incremental Static Regeneration) with configurable revalidation
- All themes must implement the same layout interface for consistency
