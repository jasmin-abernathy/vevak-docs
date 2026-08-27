# VeVak documentation

Central documentation repository for **VeVak**, an open-source Android project built around privacy-first, on-demand location sharing by SMS.

## Canonical product definition

VeVak currently focuses on a deliberately small core:

1. a trusted contact is explicitly configured on the phone;
2. that contact sends the exact configured SMS request phrase;
3. VeVak validates the sender and request locally;
4. VeVak first checks a recent cached position, then may perform one bounded location lookup;
5. VeVak sends a response by SMS when the platform, network and permissions allow it.

The core must remain usable without a VeVak account, mandatory cloud service or permanent Internet connection. SMS itself still requires a working mobile network/service.

## Permanent principles

- FOSS-first and open source;
- local processing by default;
- no advertising, trackers or telemetry;
- no mandatory VeVak server for the core feature;
- explicit consent and authorised contacts;
- no covert-use promise;
- minimal permissions and data retention;
- bounded background work;
- no continuous location tracking in the current core;
- accessibility and ecodesign considered product requirements, not afterthoughts;
- limitations and failure modes documented honestly.

## Current state — August 2026

Implemented/reference behaviour in `jasmin-abernathy/vevak` includes the SMS engine, authorised contact and phrase validation, rate limiting, recent-cache-first location lookup, a bounded fresh location attempt, optional older-position fallback, privacy-safe diagnostics, FOSS/Play separation principles and F-Droid-oriented metadata.

The project remains a prototype under active testing. A stable release still requires reproducible builds, real-device and multi-operator testing, dual-SIM validation, background/screen-off testing, accessibility review, security/privacy review and documented failure modes.

## Priority order

### P0 — stabilise before expanding

- full installable Android build;
- instrumented SMS receiver tests;
- real SMS tests with screen on/off and app closed;
- robust dual-SIM/eSIM behaviour;
- multi-manufacturer/background restriction tests;
- guided end-to-end readiness test;
- actionable manufacturer/battery diagnostics.

### P1 — after core reliability

- explicit outgoing SOS to the trusted contact, with strong anti-accidental-trigger UX;
- stronger request authentication compatible with SMS/offline use;
- several authorised contacts with explicit rights;
- accessibility hardening and published testing evidence.

### Exploratory / gated

A local-first device-recovery feature set inspired by useful ideas from Android Find Hub/Find Device may be explored inside VeVak rather than as a separate application. Candidate capabilities include ringing the device, reporting battery/state, showing a lock-screen message, exposing last known position and local/network readiness information, and an optional encrypted relay where a direct SMS path is insufficient.

Any remote-control capability must be separately threat-modelled, explicit, revocable and clearly consent-based. A mandatory central VeVak tracking server is out of scope for the core design.

Temporary periodic tracking, automatic emergency-service contact, covert tracking, sensor-heavy triggers and media capture are not part of the current core and must not be presented as committed features without a new explicit product decision.

## Repository topology

Current accessible public repositories:

- `jasmin-abernathy/vevak` — Android reference implementation and roadmap;
- `jasmin-abernathy/Vevak-website` — public website;
- `jasmin-abernathy/vevak-docs` — this documentation repository;
- `jasmin-abernathy/vevak-brand` — brand and design guidance.

Planned Android split discussed in August 2026:

- `VeVaK-android-FOSS` — canonical FOSS build using platform/open Android APIs;
- `VeVaK-android-PlayStore` — Play-specific build, including proprietary location components only when justified;
- `VeVaK-android-Custom` — separated custom/integration work so client-specific requirements do not contaminate the canonical build.

## Safety wording

VeVak is not an emergency service and must never be the user's only safety mechanism. It does not guarantee SMS delivery, location availability or background execution.

## Licence

See `LICENSE`.
