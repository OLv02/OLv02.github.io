# Theme Switcher Documentation

## Overview

This site supports switching between **Modern** and **Retro** themes with persistence across page navigations and browser sessions.

## How It Works

### 1. Theme Persistence - localStorage

We use **localStorage** (not cookies) for the following reasons:
- **Simpler implementation**: No server-side handling required (this is a static site)
- **Better performance**: No HTTP overhead on every request
- **Larger storage**: 5-10MB vs 4KB for cookies
- **Client-side only**: No data sent to server on every request
- **Persistent**: Survives browser restarts

Storage key: `'site-theme'`
Valid values: `'modern'` | `'retro'`
Default: `'modern'`

### 2. Early Theme Application (No FOUC)

To prevent Flash of Unstyled Content (FOUC), we use a tiny inline synchronous script that runs BEFORE any CSS loads:

**Location**: `src/components/ThemeScript.astro`
**Included in**: `src/layouts/BaseLayout.astro` (first thing in `<head>`)

```javascript
// Runs immediately on page load
const savedTheme = localStorage.getItem('site-theme') || 'modern';
document.documentElement.setAttribute('data-theme', savedTheme);
```

This script is inlined (not external) and uses `is:inline` to ensure it's not bundled or deferred.

### 3. Theme Toggle UI

**Component**: `src/components/ThemeToggle.astro`
**Locations**:
- Modern theme: `src/layouts/Layout.astro` navigation (desktop and mobile)
- Retro theme: `src/components/Header.astro` navigation

The toggle shows both options with the current theme highlighted:
- **Modern theme**: Blue highlight on "Modern", gray on "Retro"
- **Retro theme**: Yellow highlight on "Retro", gray button on "Modern"

**Accessibility**:
- Proper `aria-label` that updates with current state
- `aria-live="polite"` for screen reader announcements
- Keyboard navigable (button element)
- Focus visible states

### 4. Theme Switching Logic

When the toggle button is clicked:
1. Read current theme from `html[data-theme]`
2. Toggle to opposite theme
3. Update `html[data-theme]` attribute immediately (instant visual change)
4. Save new preference to `localStorage`
5. Update `aria-label` for accessibility

## Files Modified

### Core Implementation
- `src/components/ThemeScript.astro` - Early-loading theme detection
- `src/components/ThemeToggle.astro` - UI toggle component
- `src/layouts/BaseLayout.astro` - Includes ThemeScript in `<head>`

### UI Integration
- `src/layouts/Layout.astro` - ThemeToggle in modern theme nav
- `src/components/Header.astro` - ThemeToggle in retro theme nav

### Styling
- `src/styles/theme-retro.css` - All retro styles scoped to `html[data-theme="retro"]`
- `src/styles/global.css` - Minimal global styles, imports retro theme

## Page Coverage

### ✅ Pages with Full Theme Support
All pages using `BaseLayout.astro`:
- `/blog` - Blog index
- `/blog/*` - Individual blog posts
- `/about` - About page

These pages have:
- ✅ Early theme script (no FOUC)
- ✅ Theme toggle in navigation
- ✅ Persistence across navigation
- ✅ Both themes fully functional

### ⚠️ Homepage Note

`src/pages/index.astro` currently creates its own `<html>` structure instead of using `BaseLayout.astro`.

**Current state**:
- ✅ Has ThemeToggle (can switch themes)
- ❌ Missing ThemeScript (potential FOUC on reload)
- ❌ Not using unified layout system

**To fix**: Update `index.astro` to use `Layout.astro` or `BaseLayout.astro` wrapper.

## Testing the Theme Switcher

### Manual Testing
1. Visit any page (e.g., `/blog`)
2. Click "Retro" in the theme toggle
3. Observe instant theme change to 1990s aesthetic
4. Navigate to another page → theme persists
5. Refresh the page → theme persists
6. Close and reopen browser → theme persists
7. Clear localStorage → resets to "modern" default

### Developer Testing
```javascript
// Check current theme
document.documentElement.getAttribute('data-theme')

// Check saved preference
localStorage.getItem('site-theme')

// Manually set theme
localStorage.setItem('site-theme', 'retro')
location.reload()

// Clear preference
localStorage.removeItem('site-theme')
location.reload()
```

## Browser Support

**localStorage**: IE8+, all modern browsers
**Graceful degradation**: If localStorage is blocked, theme will still work per-page (just won't persist)

## Future Enhancements

Potential improvements:
- [ ] System preference detection (`prefers-color-scheme`)
- [ ] Smooth theme transition animations
- [ ] Keyboard shortcut for theme toggle (e.g., `Ctrl+Shift+T`)
- [ ] Theme preview before switching
- [ ] Add third theme option (e.g., "auto")
