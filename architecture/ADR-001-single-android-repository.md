# ADR-001 — Keep VeVak Android variants in one repository

- **Status:** Accepted
- **Date:** 2026-08-27

## Context

VeVak needs a canonical FOSS build while still allowing a Play-distributed variant to use platform-specific components when they provide a justified benefit.

The Android project already implements this separation with Gradle product flavors:

- `foss` uses Android platform/open APIs and does not depend on Google Play Services;
- `play` isolates its Google location dependency with `playImplementation` and its own build configuration.

The SMS engine, validation rules, privacy constraints, UI, tests and most location logic remain shared.

## Decision

Keep the Android application in the single public repository `jasmin-abernathy/vevak`.

Use Gradle flavors, source sets and dependency scopes to isolate distribution-specific components.

Do not create separate `VeVaK-android-FOSS` and `VeVaK-android-PlayStore` repositories.

Do not create a generic `VeVaK-android-Custom` repository in advance. A separate integration repository may be created later only when a concrete, durable requirement cannot be isolated cleanly within the existing project.

## Why

A single repository:

- keeps fixes shared;
- prevents FOSS and Play implementations from drifting;
- reduces duplicated CI, documentation and dependency maintenance;
- makes comparative tests between variants easier;
- keeps issue tracking and release criteria coherent;
- better matches VeVak's functional-sobriety and maintenance goals.

## Constraints

The monorepo is appropriate while:

- the FOSS variant can be built without proprietary runtime dependencies;
- Play-only dependencies stay strictly scoped to the Play variant;
- core behaviour remains consistent between variants;
- tests cover the shared core and variant-specific boundaries.

## When to reconsider

Revisit this decision if a real integration requires a fundamentally different licence or architecture, a distribution requires incompatible release histories that cannot be handled cleanly in one repository, or proprietary components can no longer be isolated from the canonical FOSS build.

Until one of those conditions is met, creating more Android repositories is intentionally avoided.
