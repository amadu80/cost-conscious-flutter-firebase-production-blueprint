# Accessibility, localization, and SEO

## Problem

A web application can be technically operational while excluding keyboard, screen-reader, low-vision, low-bandwidth, or non-default-language users. Flutter's canvas-oriented rendering and SPA routing also require deliberate semantics, document metadata, and crawl strategy.

## Options considered

1. Treat accessibility, localization, and discovery as post-launch polish.
2. Replace the client architecture solely for theoretical SEO.
3. Define requirements by route and user need, test them continuously, and introduce server-rendered surfaces only where evidence requires them.

## Decision and why

Build accessibility and localization into shared components and release gates. Define which routes are public and discoverable, which are private or application-only, and which need server-rendered metadata or separate content pages.

## Accessibility controls

Use semantic labels and roles, logical focus order, visible focus, keyboard operation, sufficient contrast, scalable text, reduced-motion consideration, descriptive errors, announced loading changes, and meaningful alternatives for images. Automated scans catch only part of the problem; include keyboard and screen-reader testing in critical flows.

## Localization controls

Keep user-visible strings in localization resources, test every supported locale, handle text expansion and pluralization, and localize validation and backend error mapping. Format currency, dates, numbers, geography, and terminology according to the target community rather than translating words in isolation.

## SEO and public web strategy

Provide unique titles, descriptions, canonical URLs, robots policy, sitemap where useful, structured data only when truthful, and stable public links. Do not expose private or low-quality dynamic pages merely to increase indexed volume. If crawler rendering or social previews remain inadequate, evaluate pre-rendered or server-rendered public landing/detail pages while retaining Flutter for authenticated application flows.

## Consequences and validation

Semantics and localization increase test scope; reusable components reduce repetition. SEO work can conflict with privacy and stale content, so define indexing policy explicitly. Validate with keyboard navigation, screen readers, zoom, contrast, multiple locales, constrained layouts, crawler tools, link previews, and refreshed deep links.
