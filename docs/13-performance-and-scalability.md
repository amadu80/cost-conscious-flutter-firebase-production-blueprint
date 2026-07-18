# Performance and scalability

## Problem

Flutter web startup, large JavaScript bundles, images, Firestore reads, function cold starts, and edge behavior affect both experience and cost. Optimizing without measurements wastes effort; waiting until scale can lock expensive access patterns into the product.

## Options considered

1. Optimize prematurely for hypothetical traffic.
2. Ignore performance until users complain.
3. Establish budgets and representative measurements, then optimize the dominant constraint.

## Decision and why

Use measured performance budgets for startup, core interaction latency, image weight, backend p95/p99 latency, and billed operations per journey. Test on representative low-end devices and constrained networks, not only developer hardware.

## Client decisions

Compress uploaded media to a modern format, generate appropriate dimensions, lazy-load off-screen content, and use placeholders that preserve layout. Track bundle growth and defer non-critical initialization. Cache immutable assets at the edge while revalidating mutable bootstrap files.

## Data and backend decisions

Use cursor pagination, bounded page sizes, indexed queries, local persistence with explicit freshness, and denormalized list summaries where justified. Avoid N+1 reads, unbounded listeners, fan-out without limits, and retry loops. Make functions idempotent, cap concurrency and batches, and measure cold and warm latency separately.

## Scaling options

Firestore indexes and aggregation may be sufficient initially. Consider dedicated search, queues, scheduled aggregation, CDN transformation, or a custom service only when measurements show that query capability, latency, throughput, or cost cannot meet objectives. Record migration thresholds before adopting another vendor.

## Consequences and validation

Caching and denormalization improve speed but introduce freshness and repair complexity. Performance tests should cover cold load, warm load, previous-release upgrade, slow network, large collection, image-heavy page, and backend failure. Revisit budgets when device mix, traffic, feature scope, or provider pricing changes.
