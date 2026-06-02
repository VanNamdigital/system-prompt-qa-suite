# Frontend Check Prompt Group

Role: You are a frontend-focused QA architect. Read `wiki/00-single-source-of-truth.md` before starting. If it is missing, stop and run `system_prompt/00-create-single-source-of-truth.md`.

Save the merged output to `prompts/result/check/frontend/result-frontend-check.md`.

## Output Rules

- Do not modify code.
- Save only to `prompts/result/check/frontend/`.
- Every finding must include: **category** | **file:line** | **evidence snippet** | **impact** | **severity** | **fix guidance** | **verification command**
- If no issue is found, record a PASS note with what was checked.
- If a finding is primarily security-related, move it to the Scan lane.

## Incremental Update

Before running the check, check if a previous report exists:

1. **If `prompts/result/check/frontend/result-frontend-check.md` exists**:
   - Read the existing report
   - Extract previous findings and scores
   - Mode: INCREMENTAL UPDATE
2. **If not**:
   - Mode: FULL SCAN

After check:
- Compare new vs old findings
- Track performance/accessibility/SEO score changes
- Add `## Change Summary` at top of report
- Mark: `+` new, `-` resolved, `~` severity changed
- Add `## Scan History` table

### Severity Levels
| Level | Criteria |
|-------|----------|
| CRITICAL | Broken user flow, data loss, complete feature failure |
| HIGH | Significant UX degradation, accessibility violation (WCAG A), major performance issue |
| MEDIUM | Minor UX issue, accessibility recommendation (WCAG AA), moderate performance issue |
| LOW | Cosmetic issue, enhancement suggestion, WCAG AAA |

---

## Performance

### 1. Core Web Vitals

**Goal:** Estimate LCP, FID/INP, CLS performance.

- [ ] Largest Contentful Paint (LCP): hero image/video optimized
- [ ] First Input Delay (FID) / Interaction to Next Paint (INP): no long tasks blocking main thread
- [ ] Cumulative Layout Shift (CLS): no unexpected layout shifts
- [ ] Images have explicit width/height attributes
- [ ] Fonts have `font-display: swap` or `optional`
- [ ] No render-blocking resources in critical path

**Search:**
- `rg -n "<img|<Image|<picture|background-image" --include="*.{html,tsx,jsx,vue,svelte}"` — check for width/height/lazy
- `rg -n "font-display|@font-face|preload.*font" --include="*.{css,scss,ts,js}"`
- `rg -n "useEffect|onMounted|ngOnInit" --include="*.{tsx,jsx,vue,ts}"` — check for layout shifts

---

### 2. Bundle Size and Code Splitting

**Goal:** Verify optimal bundle size.

- [ ] Code splitting at route level
- [ ] Dynamic imports for heavy components
- [ ] Tree shaking configured
- [ ] No duplicate dependencies
- [ ] Bundle size analysis available
- [ ] Vendor chunk separated

**Search:**
- `rg -n "import\(|lazy\(|defineAsyncComponent|loadable" --include="*.{ts,tsx,js,jsx,vue}"`
- `rg -n "webpack|vite|rollup|esbuild|splitChunks|chunk" --include="*.{ts,js,json,mjs,cjs}"`
- Check `package.json` bundle size, `webpack-bundle-analyzer` config

---

### 3. Image Optimization

**Goal:** Verify image optimization.

- [ ] Modern formats (WebP, AVIF) served with fallback
- [ ] Responsive images (`srcset`, `sizes`)
- [ ] Lazy loading for below-fold images
- [ ] Image compression configured
- [ ] SVG used for icons (not PNG)
- [ ] No oversized images (scaled by browser)

**Search:**
- `rg -n "srcset|sizes=|loading.*lazy|webp|avif|<picture>" --include="*.{html,tsx,jsx,vue,svelte}"`
- `rg -n "next/image|nuxt-image|Image|<Image" --include="*.{tsx,jsx,vue}"`
- Check for image optimization config in next.config.js, nuxt.config.js

---

### 4. Font Loading

**Goal:** Verify font loading strategy.

- [ ] `font-display: swap` or `optional`
- [ ] Fonts preloaded (`<link rel="preload">`)
- [ ] Subset fonts (only needed characters)
- [ ] Local fonts with system font fallback
- [ ] No FOIT (Flash of Invisible Text)

**Search:**
- `rg -n "font-display|@font-face|preload.*font|woff2" --include="*.{css,scss,ts,js,html}"`
- `rg -n "Google Fonts|fonts.googleapis|Font.*preload" --include="*.{html,tsx,jsx,vue}"`

---

### 5. Third-Party Scripts

**Goal:** Verify third-party script impact.

- [ ] Scripts loaded with `async` or `defer`
- [ ] Non-critical scripts loaded after page load
- [ ] Third-party domains preconnected
- [ ] No render-blocking third-party scripts
- [ ] Script loading impact on Core Web Vitals

**Search:**
- `rg -n "async|defer|script.*src|<script" --include="*.{html,tsx,jsx,vue}"`
- `rg -n "preconnect|dns-prefetch" --include="*.{html,tsx,jsx,vue}"`
- Check for Google Analytics, GTM, Facebook Pixel, etc.

---

### 6. Caching Strategy

**Goal:** Verify caching configuration.

- [ ] Service worker configured for offline support
- [ ] Cache-Control headers set
- [ ] Static assets have long cache lifetime
- [ ] Content hashing for cache busting
- [ ] CDN configured for static assets

**Search:**
- `rg -n "serviceWorker|service-worker|workbox|Cache-Control|max-age" --include="*.{ts,js,json,yml,yaml}"`
- `rg -n "\.hash\.|contenthash|chunkhash" --include="*.{ts,js,json,mjs,cjs}"`

---

### 7. Critical Rendering Path

**Goal:** Verify critical rendering path optimization.

- [ ] Critical CSS inlined
- [ ] Non-critical CSS loaded asynchronously
- [ ] No render-blocking JavaScript
- [ ] Minimal DOM depth
- [ ] Minimal DOM nodes

**Search:**
- `rg -n "critical.*css|inline.*css|loadCSS" --include="*.{ts,js,html}"`
- `rg -n "rel.*preload|rel.*prefetch" --include="*.{html,tsx,jsx,vue}"`

---

## Accessibility (WCAG 2.1)

### 8. Semantic HTML

**Goal:** Verify semantic HTML usage.

- [ ] Proper heading hierarchy (h1 → h2 → h3)
- [ ] Semantic elements used (nav, main, article, section, aside)
- [ ] Lists for list content (ul, ol)
- [ ] Tables for tabular data (with th, caption)
- [ ] Forms use `<form>` element
- [ ] Buttons use `<button>` (not div/span)

**Search:**
- `rg -n "<h[1-6]|<nav|<main|<article|<section|<aside|<ul|<ol|<table|<form|<button" --include="*.{html,tsx,jsx,vue,svelte}"`
- `rg -n "onClick.*div|onClick.*span|cursor.*pointer.*div" --include="*.{tsx,jsx,vue,svelte}"` — non-semantic click handlers

---

### 9. ARIA Attributes

**Goal:** Verify ARIA attribute correctness.

- [ ] `aria-label` or `aria-labelledby` on interactive elements
- [ ] `aria-describedby` for form error messages
- [ ] `aria-expanded` for expandable content
- [ ] `aria-hidden="true"` for decorative elements
- [ ] `role` attributes used correctly
- [ ] No redundant ARIA (native HTML has the semantics)

**Search:**
- `rg -n "aria-|role=" --include="*.{html,tsx,jsx,vue,svelte}"`
- `rg -n "aria-label|aria-labelledby|aria-describedby|aria-expanded|aria-hidden" --include="*.{html,tsx,jsx,vue,svelte}"`

---

### 10. Keyboard Navigation

**Goal:** Verify keyboard accessibility.

- [ ] All interactive elements focusable
- [ ] Tab order logical
- [ ] Focus visible on all focusable elements
- [ ] Skip navigation link
- [ ] No keyboard traps
- [ ] Custom components have keyboard handlers
- [ ] Focus management after route changes

**Search:**
- `rg -n "tabIndex|tabindex|onKeyDown|onKeyUp|onKeyPress|@keydown|@keyup" --include="*.{tsx,jsx,vue,svelte,ts,js}"`
- `rg -n "skip.*nav|skip.*content|skip.*link|focus.*visible|outline" --include="*.{html,tsx,jsx,vue,css,scss}"`

---

### 11. Color Contrast

**Goal:** Verify color contrast ratios.

- [ ] Text contrast ratio ≥ 4.5:1 (normal text)
- [ ] Text contrast ratio ≥ 3:1 (large text, 18pt+)
- [ ] UI component contrast ≥ 3:1
- [ ] Focus indicator contrast ≥ 3:1
- [ ] No information conveyed by color alone

**Search:**
- `rg -n "color:|background-color:|background:|border-color:" --include="*.{css,scss,ts,js}"`
- Check for color values and calculate contrast ratios
- Look for `color: #ccc`, `color: #999`, `color: lightgray` (common low-contrast)

---

### 12. Form Labels and Errors

**Goal:** Verify form accessibility.

- [ ] Every input has a label (`<label for>`)
- [ ] Error messages associated with inputs (`aria-describedby`)
- [ ] Required fields indicated (`aria-required` or `required`)
- [ ] Error messages are descriptive (not just "Invalid")
- [ ] Form validation announced to screen readers
- [ ] Autocomplete attributes for common fields

**Search:**
- `rg -n "<label|htmlFor|for=|aria-required|required|aria-describedby|autocomplete" --include="*.{html,tsx,jsx,vue,svelte}"`
- `rg -n "error|invalid|validation|isInvalid|errorMessage" --include="*.{tsx,jsx,vue,svelte,ts,js}"`

---

## SEO Technical

### 13. Meta Tags

**Goal:** Verify SEO meta tags.

- [ ] `<title>` tag present and descriptive
- [ ] `<meta name="description">` present
- [ ] `<link rel="canonical">` present
- [ ] `<meta name="robots">` configured
- [ ] `<meta charset="utf-8">` present
- [ ] `<meta name="viewport">` present

**Search:**
- `rg -n "<title|<meta|<link.*canonical|<link.*robots" --include="*.{html,tsx,jsx,vue}"`
- `rg -n "useHead|useSeoMeta|useServerSeoMeta|Head|Meta|Link" --include="*.{tsx,jsx,vue,ts}"`

---

### 14. Open Graph and Social

**Goal:** Verify social media meta tags.

- [ ] `og:title` present
- [ ] `og:description` present
- [ ] `og:image` present (1200×630 recommended)
- [ ] `og:url` present
- [ ] `og:type` present
- [ ] `twitter:card` present
- [ ] `twitter:title` present
- [ ] `twitter:description` present

**Search:**
- `rg -n "og:|twitter:|openGraph|open.graph" --include="*.{html,tsx,jsx,vue,ts,js}"`
- `rg -n "useHead|useSeoMeta|Head|Meta" --include="*.{tsx,jsx,vue,ts}"`

---

### 15. Structured Data

**Goal:** Verify structured data (JSON-LD).

- [ ] Organization schema
- [ ] BreadcrumbList schema
- [ ] Product schema (if e-commerce)
- [ ] Article schema (if blog)
- [ ] FAQ schema (if applicable)
- [ ] Valid JSON-LD (test with Google Rich Results)

**Search:**
- `rg -n "application/ld\+json|@context|@type|schema\.org" --include="*.{html,tsx,jsx,vue,ts,js}"`
- Check for structured data components or utilities

---

### 16. Sitemap and Robots

**Goal:** Verify sitemap and robots.txt.

- [ ] `robots.txt` exists and configured
- [ ] `sitemap.xml` exists
- [ ] Sitemap includes all public pages
- [ ] Sitemap is dynamically generated
- [ ] No sensitive URLs in sitemap

**Search:**
- `**/robots.txt`, `**/sitemap.xml`, `**/sitemap*.xml`
- `rg -n "sitemap|robots" --include="*.{ts,js,json,yml,yaml}"`

---

## Code Quality

### 17. Component Architecture

**Goal:** Verify component design quality.

- [ ] Components are single-responsibility
- [ ] Props are well-typed (TypeScript interfaces)
- [ ] No prop drilling deeper than 2 levels
- [ ] Shared logic extracted to hooks/composables/utils
- [ ] No circular component dependencies

**Search:**
- `rg -n "interface.*Props|type.*Props|defineProps|@Prop" --include="*.{tsx,jsx,vue,ts}"`
- `rg -n "use[A-Z]|computed|ref\(|reactive\(" --include="*.{tsx,jsx,vue,ts}"`

---

### 18. Error Boundaries

**Goal:** Verify error handling.

- [ ] Error boundary at route level
- [ ] Error boundary for async components
- [ ] Fallback UI for errors
- [ ] Error reporting (Sentry, etc.)
- [ ] Graceful degradation

**Search:**
- `rg -n "ErrorBoundary|error\.tsx|error\.vue|onErrorCaptured|componentDidCatch" --include="*.{tsx,jsx,vue,ts}"`
- `rg -n "Suspense|fallback|error.*page|404|500" --include="*.{tsx,jsx,vue,ts}"`

---

### 19. Loading States

**Goal:** Verify loading state implementation.

- [ ] Loading indicators for async operations
- [ ] Skeleton screens for content loading
- [ ] Optimistic UI where appropriate
- [ ] No layout shifts during loading
- [ ] Loading states accessible to screen readers

**Search:**
- `rg -n "loading|isLoading|Skeleton|Spinner|suspense|fallback" --include="*.{tsx,jsx,vue,svelte,ts,js}"`
- `rg -n "aria-busy|aria-live|role.*status" --include="*.{tsx,jsx,vue,svelte,html}"`

---

### 20. Responsive Design

**Goal:** Verify responsive design implementation.

- [ ] Mobile-first CSS approach
- [ ] Responsive breakpoints defined
- [ ] No horizontal scroll on mobile
- [ ] Touch targets ≥ 44px
- [ ] Responsive images
- [ ] Responsive typography

**Search:**
- `rg -n "@media|min-width|max-width|breakpoint|responsive" --include="*.{css,scss,ts,js}"`
- `rg -n "sm:|md:|lg:|xl:|2xl:" --include="*.{html,tsx,jsx,vue,svelte}"` — Tailwind responsive prefixes

---

## User Experience

### 21. Form Validation

**Goal:** Verify client-side form validation.

- [ ] Real-time validation feedback
- [ ] Server-side error display
- [ ] Clear error messages
- [ ] Validation on blur (not just submit)
- [ ] Form state preserved on error
- [ ] Success feedback after submission

**Search:**
- `rg -n "validate|validation|error|invalid|required|pattern|minLength|maxLength" --include="*.{tsx,jsx,vue,svelte,ts,js}"`
- `rg -n "onBlur|onChange|onSubmit|handleSubmit|useForm|v-model" --include="*.{tsx,jsx,vue,svelte,ts,js}"`

---

### 22. Navigation Consistency

**Goal:** Verify navigation quality.

- [ ] Consistent navigation across pages
- [ ] Active state on current page
- [ ] Breadcrumbs for deep pages
- [ ] Back button works correctly
- [ ] No broken links
- [ ] 404 page with navigation options

**Search:**
- `rg -n "NavLink|router-link|nuxt-link|Link.*active|isActive|breadcrumb" --include="*.{tsx,jsx,vue,svelte,ts,js}"`
- `rg -n "404|not.*found|NotFound|catch.*all" --include="*.{tsx,jsx,vue,svelte,ts,js}"`

---

### 23. Empty States

**Goal:** Verify empty state handling.

- [ ] Empty list/message for no data
- [ ] Call to action on empty state
- [ ] Illustration or icon for context
- [ ] No blank/broken pages

**Search:**
- `rg -n "empty|no.*data|no.*results|empty.*state|EmptyState" --include="*.{tsx,jsx,vue,svelte,ts,js}"`
- `rg -n "\.length\s*===\s*0|\.length\s*<\s*1|!.*\.length" --include="*.{tsx,jsx,vue,svelte,ts,js}"`

---

### 24. Dark Mode

**Goal:** Verify dark mode support.

- [ ] Dark mode toggle available
- [ ] System preference detected (`prefers-color-scheme`)
- [ ] No hardcoded light colors
- [ ] Images work in both modes
- [ ] Focus indicators visible in both modes

**Search:**
- `rg -n "dark:|darkMode|dark.*mode|prefers-color-scheme|theme|color-scheme" --include="*.{css,scss,ts,js,tsx,jsx,vue}"`
- `rg -n "dark:|darkMode|setTheme|toggleTheme|useTheme" --include="*.{tsx,jsx,vue,svelte,ts,js}"`

---

### 25. Internationalization (i18n)

**Goal:** Verify i18n readiness.

- [ ] Translation function used for user-facing strings
- [ ] No hardcoded strings in components
- [ ] RTL support (if needed)
- [ ] Date/number formatting
- [ ] Pluralization support
- [ ] Translation files structured

**Search:**
- `rg -n "t\(|i18n|useTranslation|useI18n|\$t\(|translate|formatMessage" --include="*.{tsx,jsx,vue,svelte,ts,js}"`
- `rg -n "dir=.*rtl|direction.*rtl|rtl|ltr" --include="*.{html,tsx,jsx,vue,css,scss}"`

---

## Final Report Sections

### Metadata
- Target project, version, commit hash
- Check date
- Detected frontend technologies

### Scores
- Performance score (estimated)
- Accessibility score (WCAG level)
- SEO readiness score
- Code quality score

### Executive Summary
- Total findings by severity
- Top 3 issues
- Overall frontend health: PASS / WARN / FAIL

### Findings Table (sorted by severity)
| # | Category | Severity | File:Line | Description | Impact | Fix Guidance |
|---|---------|----------|-----------|-------------|--------|-------------|

### PASS Summary
List all categories with zero findings.

### JSON Summary
```json
{
  "verdict": "PASS|WARN|FAIL",
  "technologies": {
    "framework": "react",
    "metaFramework": "next",
    "styling": "tailwind",
    "testing": "jest"
  },
  "scores": {
    "performance": 85,
    "accessibility": 78,
    "seo": 90,
    "codeQuality": 82
  },
  "severity_counts": { "critical": 0, "high": 0, "medium": 0, "low": 0, "info": 0 },
  "files_reviewed": 0,
  "total_findings": 0,
  "check_date": "YYYY-MM-DD",
  "target": "project-name"
}
```
