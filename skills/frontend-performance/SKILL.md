---
name: frontend-performance
description: Optimize web frontend performance including Core Web Vitals, bundle size, rendering, and loading. Use when pages are slow, LCP/CLS/FID are poor, JavaScript bundles are too large, or when implementing lazy loading, code splitting, caching. Trigger on "performance", "slow website", "LCP", "CLS", "bundle size", "lazy loading", "code splitting", "Core Web Vitals", or "page speed".
---

# Frontend Performance

Performance is a feature. Measure before optimizing; measure again after.

## Core Web Vitals

| Metric | What | Target |
|--------|------|--------|
| LCP | Largest Contentful Paint | < 2.5s |
| INP | Interaction to Next Paint | < 200ms |
| CLS | Cumulative Layout Shift | < 0.1 |

## Diagnostic Tools

- Lighthouse (Chrome DevTools)
- PageSpeed Insights (field data)
- WebPageTest (filmstrip, waterfall)
- Chrome DevTools Performance tab
- Bundle analyzer (`webpack-bundle-analyzer` / `rollup-plugin-visualizer`)

## LCP Optimizations (Loading)

1. **Preload the LCP image**: `<link rel="preload" as="image" href="hero.jpg">`
2. **Serve images in modern format**: WebP/AVIF with fallbacks
3. **Eliminate render-blocking resources**: `defer` scripts, inline critical CSS
4. **Use CDN** for static assets
5. **Server-side render** or prerender key content
6. **Reduce TTFB**: cache API responses, optimize server queries

## CLS Optimizations (Layout Stability)

1. **Always set width/height on images and videos** (or use `aspect-ratio`)
2. **Reserve space for ads/embeds** with min-height
3. **Don't insert content above existing content** (use skeletons below)
4. **Use `font-display: swap`** with preloaded fallback metrics
5. **Avoid dynamically injected banners** without reserved space

## INP Optimizations (Responsiveness)

1. **Break up long tasks**: `scheduler.yield()` or `setTimeout(0)` between chunks
2. **Debounce non-critical event handlers** (scroll, resize)
3. **Use `requestIdleCallback`** for non-urgent work
4. **Web Workers** for heavy computation
5. **Avoid layout thrashing**: batch DOM reads, then writes

## Bundle Size

```bash
# Analyze what's in your bundle
npx webpack-bundle-analyzer stats.json

# Check your budget
npx bundlesize
```

Techniques:
- **Code splitting**: dynamic `import()` for routes/features
- **Tree shaking**: ensure ESM imports, side-effect-free packages
- **Replace heavy deps**: lodash → lodash-es, moment → date-fns/Temporal
- **External scripts** for rarely used features (analytics, chat widgets)
- **gzip/brotli** on the server

```javascript
// Route-based code splitting
const Dashboard = lazy(() => import('./Dashboard'));
const Settings = lazy(() => import('./Settings'));
```

## Image Optimization

```html
<picture>
  <source srcset="hero.avif" type="image/avif">
  <source srcset="hero.webp" type="image/webp">
  <img src="hero.jpg" alt="..." width="800" height="600" loading="lazy">
</picture>
```

- Use `loading="lazy"` for below-fold images
- Use `loading="eager"` + `fetchpriority="high"` for LCP image
- Responsive images with `srcset` and `sizes`
- Target: < 100KB for LCP image, < 50KB for others

## Caching Strategy

| Asset | Cache-Control |
|-------|--------------|
| HTML | `no-cache` or short max-age |
| Hashed assets (`.abc123.js`) | `max-age=31536000, immutable` |
| API responses | `s-maxage=60, stale-while-revalidate=300` |
| Images | `max-age=86400` |

## Checklist

- [ ] LCP < 2.5s on mobile (field data, not just Lighthouse)
- [ ] CLS < 0.1 (images have dimensions, no layout shifts)
- [ ] INP < 200ms (no long tasks blocking interaction)
- [ ] JS bundle < 200KB gzipped (application code)
- [ ] Images in modern formats with lazy loading
- [ ] Code splitting on routes
- [ ] Render-blocking resources eliminated
- [ ] Performance budget enforced in CI
