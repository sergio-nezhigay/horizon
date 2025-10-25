# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Horizon 3.0.1** - Modern Shopify theme built with component-based architecture, web components, and TypeScript-enabled JavaScript.

**Store**: c2da09-15.myshopify.com
**Theme ID**: 181526692156

## Common Commands

### Development
```bash
npm run dev    # Start local dev server with hot reload
npm run push   # Push theme changes to Shopify store
npm run pull   # Pull latest theme from Shopify store
```

### Testing
After making changes to TypeScript/JavaScript files, check for type errors:
```bash
# TypeScript checking is enabled via jsconfig.json with checkJs: true
# VS Code will show errors inline, or use:
npx tsc --noEmit
```

## Architecture & Structure

### Directory Organization

- **`assets/`** (101 files) - CSS, JavaScript modules, SVG icons, TypeScript definitions
- **`blocks/`** (87 files) - Reusable block components
  - Public blocks: `product-card.liquid`, `image.liquid`, `text.liquid`, `button.liquid`
  - Private blocks (prefixed `_`): Sub-components not shown in theme editor
- **`sections/`** (36 files) - Page sections with settings schema
  - Main content: `product-information.liquid`, `hero.liquid`, `slideshow.liquid`
  - Layout sections: `header.liquid`, `footer.liquid`
  - JSON group sections: `header-group.json`, `footer-group.json`
- **`snippets/`** (111 files) - Liquid template partials organized by feature
- **`templates/`** - JSON templates (12) + `gift_card.liquid`
- **`layout/`** - `theme.liquid` (master layout), `password.liquid`
- **`config/`** - `settings_schema.json` (editable), `settings_data.json` (auto-generated)
- **`locales/`** (51 files) - i18n translations for 26+ languages

### Component Architecture

**Three-tier component system:**

1. **Sections** - Top-level containers with settings schema, can be added/reordered in theme editor
2. **Blocks** - Modular components added to sections, defined in section's `{% schema %}`
3. **Snippets** - Reusable template partials rendered via `{% render 'snippet-name' %}`

**Content-For Pattern:**
```liquid
{% content_for 'blocks', closest.product: product %}
  <!-- Block content with context passing -->
{% endcontent_for %}
```

### JavaScript Architecture

**Web Components:**
- Base class: `Component` (extends `DeclarativeShadowElement`)
- Components use ES6 modules with `type="module"`
- Declarative Shadow DOM for SSR-friendly components
- Path alias: `@theme/*` maps to theme root (configured in `jsconfig.json`)

**Key JavaScript patterns:**
- Critical path: `critical.js` loaded with `blocking="render"`
- Component hydration: `section-hydration.js`
- Custom events: `events.js`
- View transitions: `view-transitions.js`
- Type definitions: `global.d.ts` provides Shopify API types

**JavaScript organization by feature:**
- Checkout: `quick-add.js`, `product-form.js`, `cart-drawer.js`
- Navigation: `header.js`, `header-drawer.js`, `header-menu.js`
- Media: `media-gallery.js`, `slideshow.js`, `video-background.js`
- Product: `product-card.js`, `variant-picker.js`, `product-recommendations.js`
- Search: `predictive-search.js`, `search-page-input.js`
- Filtering: `facets.js`, `collection-links.js`

### CSS Organization

**Main stylesheets:**
- `base.css` - Global styles, CSS variables, component base styles (preloaded)
- `overflow-list.css` - Utility styles
- `template-giftcard.css` - Gift card specific styles

**CSS variable strategy:**
- Color schemes: `--color-foreground`, `--color-background`, `--color-primary`
- Layout: `--page-width`, `--section-height-*`
- Animation: `--hover-transition-duration`, `--ease-out-quad`
- Alpha/opacity: `rgb(var(--color-foreground-rgb) / var(--opacity-40))`

**CSS-in-Liquid pattern:**
```liquid
{% style %}
  .selector { property: {{ section.settings.value }}; }
{% endstyle %}
```

### Schema-Driven Components

Every section/block includes `{% schema %}` JSON block defining:
- Settings with types (text, color, image, select, etc.)
- Block types and their allowed children
- Presets for theme editor
- Conditional visibility: `"visible_if": "{{ section.settings.setting_name }}"`

## Important Patterns & Conventions

### Liquid Best Practices

**Image handling:**
```liquid
{{ image | image_url: width: 500, height: 500 }}
```
Use `image_url` filter (NOT deprecated `img_url`)

**Localization:**
```liquid
{{ 'translation.key' | t }}
```
All user-facing text uses translation keys from `locales/` files

**Context detection:**
- `request.design_mode` - Theme editor active
- `request.visual_preview_mode` - Visual preview in editor
- `shop.customer_accounts_enabled` - Customer accounts enabled
- `template.name` - Current template type

### TypeScript Configuration

`jsconfig.json` enables strict type checking:
- `checkJs: true` - Type check JavaScript files
- `strictNullChecks: true`
- `noImplicitAny: true`
- Path alias: `"@theme/*": ["./*"]`

After making changes to `.js` files, verify no type errors were introduced.

### Block System

**Public blocks** - Shown in theme editor, no underscore prefix
**Private blocks** - Internal sub-components, prefixed with `_`

Blocks use `content_for` to define content rendered in sections:
```liquid
{% content_for 'blocks', closest.product: product %}
```

### Section Groups

`header-group.json` and `footer-group.json` manage related section collections. These are JSON files that reference multiple sections to be rendered together.

## Configuration Files

### `config/settings_schema.json`
Defines theme-level settings in Shopify Admin:
- Logo & favicon settings
- Color scheme groups
- Typography settings (font families, sizes, weights)
- Layout options (page width, card hover effects, section spacing)
- Header/footer configuration
- Shopping cart settings

### `config/settings_data.json`
**Auto-generated by Shopify** - Contains actual configuration values. Do not manually edit.

## Performance Optimizations

- **Critical CSS/JS**: `critical.js` loaded with `blocking="render"`
- **View Transitions API**: Enabled via `view-transitions.js`
- **Section Hydration**: Progressive enhancement via `section-hydration.js`
- **Lazy Loading**: Images and media with `loading="lazy"`
- **Device Detection**: `isLowPowerDevice()` checks CPU/RAM for performance adaptations

## Accessibility Features

- Skip-to-content link via `skip-to-content-link.liquid`
- Focus management in `focus.js`
- `role="main"` on content areas
- ARIA labels throughout components

## Development Workflow

1. Make changes locally in your editor
2. Use `npm run dev` to preview changes with hot reload
3. For TypeScript/JavaScript changes, verify no type errors
4. Test in theme editor at `c2da09-15.myshopify.com`
5. Use `npm run push` to deploy to theme
6. Use `npm run pull` to sync from remote theme

## File Naming Conventions

- **Sections**: `kebab-case.liquid` or `kebab-case.json`
- **Blocks**: `kebab-case.liquid` (public) or `_kebab-case.liquid` (private)
- **Snippets**: `kebab-case.liquid`
- **JavaScript**: `kebab-case.js` (web component files match component name)
- **CSS**: `kebab-case.css`

## Key Snippets Reference

**Layout:**
- `section.liquid` - Section wrapper with styling/background support
- `group.liquid` - Flex container for grouping
- `spacing-style.liquid` - Dynamic padding/margin utilities

**Navigation:**
- `header-menu.liquid`, `mega-menu.liquid` - Menu components
- `header-drawer.liquid` - Mobile menu drawer

**Product Display:**
- `product-card.liquid` - Main product card
- `product-grid.liquid` - Grid layout
- `variant-picker.liquid`, `variant-swatches.liquid` - Variant selection
- `quick-add.liquid`, `quick-add-modal.liquid` - Quick add to cart
- `price.liquid`, `unit-price.liquid` - Price display

**Cart:**
- `cart-drawer.liquid` - Slide-out cart
- `cart-products.liquid` - Cart items
- `cart-summary.liquid` - Cart totals

**Search:**
- `predictive-search.liquid` - Autocomplete search
- `search-modal.liquid` - Modal search interface
- `facets-actions.liquid` - Filter controls

**Media:**
- `image.liquid` - Image with lazy loading
- `media.liquid` - Generic media renderer
- `icon.liquid` - SVG icon renderer

**Utilities:**
- `scripts.liquid`, `stylesheets.liquid`, `fonts.liquid` - Asset loading
- `color-schemes.liquid` - Color scheme variables
- `meta-tags.liquid` - SEO meta tags
