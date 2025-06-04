# Tailwind CSS v4 Complete Migration and Usage Guide

## Overview
Tailwind CSS v4 represents a major shift towards modern CSS with built-in features, eliminating the need for PostCSS plugins and complex configuration files. This guide covers everything you need to know for migrating from v3 and working with v4.

## Installation Methods

### Method 1: Vite Plugin (Recommended)
```bash
npm install tailwindcss @tailwindcss/vite
```

```javascript
// vite.config.ts
import { defineConfig } from 'vite'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [
    tailwindcss(),
  ],
})
```

### Method 2: PostCSS Plugin
```bash
npm install tailwindcss @tailwindcss/postcss
```

```javascript
// postcss.config.mjs
const config = {
  plugins: ["@tailwindcss/postcss"],
};

export default config;
```

### Method 3: CLI
```bash
npm install @tailwindcss/cli
```

### Method 4: CDN (for vanilla html/css and experimenting)
```html
<script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
```

## CSS Import Changes

### v3 Syntax (OLD - Don't use)
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### v4 Syntax (NEW - Use this)
```css
@import "tailwindcss";
```

## Configuration Approach

### v3 Approach (OLD - Don't use)
- Required `tailwind.config.js`
- Complex JavaScript configuration
- PostCSS plugins for custom utilities

### v4 Approach (NEW - Use this)
- No `tailwind.config.js` required
- CSS-native configuration using `@theme`
- Built-in modern CSS features

## Custom Utilities

### v3 Syntax (OLD)
```css
@layer utilities {
  .tab-4 {
    tab-size: 4;
  }
}

@layer components {
  .btn {
    border-radius: 0.5rem;
    padding: 0.5rem 1rem;
    background-color: ButtonFace;
  }
}
```

### v4 Syntax (NEW)
```css
@utility tab-4 {
  tab-size: 4;
}

@utility btn {
  border-radius: 0.5rem;
  padding: 0.5rem 1rem;
  background-color: ButtonFace;
}
```

## Theme Configuration

### Basic Theme Setup
```css
@import "tailwindcss";

:root {
  --background: #ffffff;
  --foreground: #171717;
}

@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --font-sans: var(--font-geist-sans);
  --font-mono: var(--font-geist-mono);
}

@media (prefers-color-scheme: dark) {
  :root {
    --background: #0a0a0a;
    --foreground: #ededed;
  }
}
```

### Loading External JavaScript Config
```css
@config "../../tailwind.config.js";
```

## CSS Variables in Arbitrary Values

### v3 Syntax (OLD)
```html
<div class="bg-[--brand-color]"></div>
```

### v4 Syntax (NEW)
```html
<div class="bg-(--brand-color)"></div>
```

## Accessing Theme Values

### v3 Approach (OLD)
```css
.my-class {
  background-color: theme(colors.red.500);
}

@media (min-width: theme(screens.xl)) {
  /* styles */
}
```

### v4 Approach (NEW)
```css
.my-class {
  background-color: var(--color-red-500);
}

@media (width >= theme(--breakpoint-xl)) {
  /* styles */
}
```

## Utility Renames

| v3 Class | v4 Class | Description |
|----------|----------|-------------|
| `shadow-sm` | `shadow-xs` | Extra small shadow |
| `shadow` | `shadow-sm` | Small shadow |
| `drop-shadow-sm` | `drop-shadow-xs` | Extra small drop shadow |
| `drop-shadow` | `drop-shadow-sm` | Small drop shadow |
| `blur-sm` | `blur-xs` | Extra small blur |
| `blur` | `blur-sm` | Small blur |
| `backdrop-blur-sm` | `backdrop-blur-xs` | Extra small backdrop blur |
| `backdrop-blur` | `backdrop-blur-sm` | Small backdrop blur |
| `rounded-sm` | `rounded-xs` | Extra small border radius |
| `rounded` | `rounded-sm` | Small border radius |
| `outline-none` | `outline-hidden` | Hidden outline |
| `ring` | `ring-3` | 3px ring (was default) |

## Removed Deprecated Utilities

| Removed Class | Replacement | Example |
|---------------|-------------|---------|
| `bg-opacity-*` | Opacity modifiers | `bg-black/50` |
| `text-opacity-*` | Opacity modifiers | `text-black/50` |
| `border-opacity-*` | Opacity modifiers | `border-black/50` |
| `divide-opacity-*` | Opacity modifiers | `divide-black/50` |
| `ring-opacity-*` | Opacity modifiers | `ring-black/50` |
| `placeholder-opacity-*` | Opacity modifiers | `placeholder-black/50` |
| `flex-shrink-*` | `shrink-*` | `shrink-0` |
| `flex-grow-*` | `grow-*` | `grow-1` |
| `overflow-ellipsis` | `text-ellipsis` | `text-ellipsis` |
| `decoration-slice` | `box-decoration-slice` | `box-decoration-slice` |
| `decoration-clone` | `box-decoration-clone` | `box-decoration-clone` |

## Behavior Changes

### Default Colors
- **Border/Divide**: Now use `currentColor` (was gray-200)
- **Ring**: Now uses `currentColor` (was blue-500)
- **Placeholder**: Uses current text color at 50% opacity (was gray-400)

### Default Sizing
- **Ring width**: Now 1px (was 3px)
- **Outline width**: Now 1px by default

### Variant Stacking Order
- **v3**: Right-to-left (`first:*:pt-0`)
- **v4**: Left-to-right (`*:first:pt-0`)

### Hover Behavior
- Only applies on devices that support hover (`hover: hover` media query)

### Space Utilities Recommendation
```html
<!-- v3 approach (still works but not recommended) -->
<div class="space-y-4">
  <div>Item 1</div>
  <div>Item 2</div>
</div>

<!-- v4 recommended approach -->
<div class="flex flex-col gap-4">
  <div>Item 1</div>
  <div>Item 2</div>
</div>
```

## Component Framework Integration

### Vue/Svelte/CSS Modules
```vue
<style>
  @reference "../../app.css";

  h1 {
    @apply text-2xl font-bold text-red-500;
  }
</style>
```

Or use CSS variables directly:
```vue
<style>
  h1 {
    color: var(--text-red-500);
    font-size: var(--text-2xl);
    font-weight: var(--font-bold);
  }
</style>
```

## Browser Support
Tailwind CSS v4 requires modern browsers:
- **Chrome 111+** (March 2023)
- **Safari 16.4+** (March 2023)  
- **Firefox 128+** (July 2024)

## CSS Preprocessors
**Not supported**: Sass, Less, Stylus are not needed or supported. v4 includes built-in support for:
- CSS nesting
- CSS variables
- CSS imports
- Modern CSS features

## Responsive Design
Mobile-first breakpoint system remains the same:

```html
<div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3">
  <!-- Responsive grid -->
</div>
```

Breakpoints:
- `sm`: 40rem (640px)
- `md`: 48rem (768px)
- `lg`: 64rem (1024px)
- `xl`: 80rem (1280px)
- `2xl`: 96rem (1536px)

## Dark Mode
Dark mode syntax remains the same:

```html
<div class="bg-white dark:bg-gray-800 text-gray-900 dark:text-white">
  <h1 class="text-2xl font-bold">Hello World</h1>
  <p class="text-gray-600 dark:text-gray-300">Welcome to v4!</p>
</div>
```

## Migration Steps

### Automatic Migration
```bash
npx @tailwindcss/upgrade@next
```

**Prerequisites**:
- Node.js 20+
- Create a new branch
- Review all changes

### Manual Migration Checklist

1. **Update Dependencies**
   ```bash
   npm uninstall tailwindcss autoprefixer postcss
   npm install tailwindcss @tailwindcss/vite
   ```

2. **Update CSS Import**
   ```css
   /* Replace this */
   @tailwind base;
   @tailwind components;
   @tailwind utilities;
   
   /* With this */
   @import "tailwindcss";
   ```

3. **Update Build Configuration**
   - Remove `tailwind.config.js` (if not using JavaScript config)
   - Update Vite/PostCSS configuration
   - Remove autoprefixer

4. **Update Class Names**
   - Replace renamed utilities
   - Remove deprecated opacity classes
   - Update arbitrary value syntax for CSS variables

5. **Test and Verify**
   - Check responsive design
   - Verify dark mode
   - Test custom utilities

## Best Practices for v4

### 1. Use CSS-Native Features
```css
@import "tailwindcss";

/* Use modern CSS nesting */
.card {
  @utility rounded-lg p-6 bg-white shadow-sm;
  
  &:hover {
    @utility shadow-md;
  }
  
  .title {
    @utility text-xl font-semibold mb-2;
  }
}
```

### 2. Leverage CSS Variables
```css
:root {
  --brand-primary: #3b82f6;
  --brand-secondary: #10b981;
}

@theme inline {
  --color-brand-primary: var(--brand-primary);
  --color-brand-secondary: var(--brand-secondary);
}
```

### 3. Use Flex/Grid with Gap
```html
<!-- Preferred approach -->
<div class="flex flex-col gap-4">
  <div>Item</div>
  <div>Item</div>
</div>

<!-- Instead of space utilities -->
<div class="space-y-4">
  <div>Item</div>
  <div>Item</div>
</div>
```

### 4. Modern Container Pattern
```css
@utility container {
  margin-inline: auto;
  padding-inline: 2rem;
  max-width: 1200px;
}
```

### 5. Component-First Utilities
```css
@utility btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border-radius: 0.375rem;
  padding: 0.5rem 1rem;
  font-weight: 500;
  transition: all 0.2s;
  
  &.btn-primary {
    background-color: var(--color-blue-600);
    color: white;
    
    &:hover {
      background-color: var(--color-blue-700);
    }
  }
}
```

## Common Gotchas

### 1. CSS Variable Syntax
```html
<!-- Wrong -->
<div class="bg-[--custom-color]"></div>

<!-- Correct -->
<div class="bg-(--custom-color)"></div>
```

### 2. Variant Order
```html
<!-- v3 style (wrong in v4) -->
<div class="first:*:pt-0"></div>

<!-- v4 style (correct) -->
<div class="*:first:pt-0"></div>
```

### 3. No @apply Directive
```css
/* Wrong - @apply doesn't exist in v4 */
.my-class {
  @apply text-center font-bold;
}

/* Correct - use @utility or CSS variables */
@utility my-class {
  text-align: center;
  font-weight: 700;
}
```

### 4. Ring Width Default
```html
<!-- v3 behavior -->
<div class="ring"></div> <!-- 3px ring -->

<!-- v4 behavior -->
<div class="ring"></div>   <!-- 1px ring -->
<div class="ring-3"></div> <!-- 3px ring (explicit) -->
```

## Complete Example: v3 to v4 Migration

### Before (v3)
```css
/* styles.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer components {
  .btn {
    @apply inline-flex items-center px-4 py-2 bg-blue-500 text-white rounded-md;
  }
}

@layer utilities {
  .text-shadow {
    text-shadow: 2px 2px 4px rgba(0,0,0,0.1);
  }
}
```

```javascript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        brand: '#ff6b6b'
      }
    }
  }
}
```

### After (v4)
```css
/* styles.css */
@import "tailwindcss";

:root {
  --brand-color: #ff6b6b;
}

@theme inline {
  --color-brand: var(--brand-color);
}

@utility btn {
  display: inline-flex;
  align-items: center;
  padding: 0.5rem 1rem;
  background-color: var(--color-blue-500);
  color: white;
  border-radius: 0.375rem;
}

@utility text-shadow {
  text-shadow: 2px 2px 4px rgba(0,0,0,0.1);
}
```

```html
<!-- Usage remains mostly the same -->
<button class="btn hover:bg-blue-600">
  Click me
</button>

<div class="bg-brand text-white p-4">
  Brand colored section
</div>
```

This guide covers all the essential changes and best practices for Tailwind CSS v4. The key is embracing CSS-native features and moving away from JavaScript configuration towards a more modern, CSS-first approach.
