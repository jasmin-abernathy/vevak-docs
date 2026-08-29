# VeVak documentation

Central documentation repository for **VeVak**, an open-source Android project built around privacy-first, on-demand location sharing by SMS.

## Canonical product definition

VeVak currently focuses on a deliberately bounded core:

1. one or more trusted contacts are explicitly configured and authorised for a finite period on the phone;
2. a contact sends that contact's exact configured SMS request phrase;
3. VeVak validates the sender, phrase, authorisation, visibility requirements and anti-tracking limits locally;
4. VeVak resolves the best available answer using a local resilience ladder;
5. VeVak sends the result by SMS when Android and the mobile network allow it.

The core must remain usable without a VeVak account, mandatory cloud service or permanent Internet connection. SMS itself still requires a working mobile network/service.

## Location resilience ladder

As of **0.3.1**, VeVak no longer relies exclusively on Android's own last-location cache. Android can clear that cache when the user switches global Location off.

For a normal authorised request, the intended order is:

1. if the currently connected Wi-Fi was positively registered as a trusted place, return its local label without waking GPS;
2. preserve that trusted-place result while the **same verified Android Wi-Fi network session** remains active, even if Android subsequently redacts the SSID after Location is switched off;
3. compare Android's provider cache with VeVak's own app-private remembered location and use a sufficiently fresh one;
4. when Location services are available, perform one bounded current-location lookup using the available Android providers;
5. if current acquisition fails and the user allows stale fallback, use the freshest remaining last-known position;
6. VeVak's own remembered position expires after **24 hours** and its age is exposed in the SMS rather than presented as current.

The remembered position is local only, is not part of `.vvk` configuration exports and is not written to diagnostics or logs. Mocked positions are not persisted in this resilience cache.

VeVak does **not** attempt to silently re-enable Android Location. A third-party application cannot promise a new GPS/network fix while the user has disabled the platform's location services. The design therefore focuses on graceful degradation rather than bypassing the operating system.

## Permanent principles

- FOSS-first and open source;
- local processing by default;
- no advertising, trackers or telemetry;
- no mandatory VeVak server for the core feature;
- explicit consent and authorised contacts;
- no covert-use promise;
- minimal permissions and bounded data retention;
- bounded background work;
- no continuous location tracking in the current core;
- accessibility and ecodesign considered product requirements, not afterthoughts;
- limitations and failure modes documented honestly.

The canonical FOSS variant still has **no `INTERNET` permission**. `ACCESS_NETWORK_STATE` is allowed only to determine whether the already-active network session is Wi-Fi and to maintain continuity of a previously verified trusted-place session; it cannot itself transmit data.

## Current state — 29 August 2026

Implemented/reference behaviour in `jasmin-abernathy/vevak` includes:

- SMS request/reply engine;
- up to five locally configured trusted contacts with separate finite authorisations;
- per-contact phrase validation and local revoke/reactivate controls;
- global anti-tracking limits shared across contacts;
- trusted-place Wi-Fi shortcut;
- continuity of the same verified Wi-Fi session when Android later redacts the SSID;
- Android provider-cache + bounded fresh-location acquisition;
- VeVak-owned last-location memory with a 24-hour hard retention cap;
- manual outgoing position sharing;
- optional safety/duress fallback path that never inspects real location;
- encrypted `.vvk` configuration export/import;
- privacy-safe diagnostics;
- FOSS/Play separation and F-Droid-oriented boundaries.

The project remains a prototype under active real-device testing. A stable release still requires reproducible builds, real-device and multi-operator testing, dual-SIM validation, background/screen-off testing, accessibility review, security/privacy review and documented failure modes.

## Priority order

### P0 — stabilise before expanding

- real SMS tests with screen on/off and app closed;
- regression test: trusted Wi-Fi registered, then global Location switched off while the same Wi-Fi session remains connected;
- regression test: known recent position, then global Location switched off, verifying that VeVak returns the explicitly aged local memory rather than `Position indisponible`;
- reboot/reconnect behaviour when Location stays off;
- robust dual-SIM/eSIM behaviour;
- multi-manufacturer/background restriction tests;
- guided end-to-end readiness test;
- actionable manufacturer/battery diagnostics.

### P1 — after core reliability

- evaluate a **local radio-environment memory** inspired by projects such as Déjà Vu / Local NLP and NeoStumbler, but only if it materially improves reliability under Android's permission model;
- do not add Wi-Fi scanning or cellular fingerprint collection merely to appear more resilient: Android still gates much of that data behind Location and the privacy cost must be justified;
- stronger request authentication compatible with SMS/offline use;
- accessibility hardening and published testing evidence.

### Exploratory / gated

A local-first device-recovery feature set inspired by useful ideas from Android Find Hub/Find Device may be explored inside VeVak rather than as a separate application. Candidate capabilities include ringing the device, reporting battery/state, showing a lock-screen message, exposing last known position and local/network readiness information, and an optional encrypted relay where a direct SMS path is insufficient.

Any remote-control capability must be separately threat-modelled, explicit, revocable and clearly consent-based. A mandatory central VeVak tracking server is out of scope for the core design.

Temporary periodic tracking, automatic emergency-service contact, covert tracking, sensor-heavy triggers and media capture are not part of the current core and must not be presented as committed features without a new explicit product decision.

## Repository topology

Current VeVak repositories:

- `jasmin-abernathy/vevak` — Android application, shared core, `foss` and `play` Gradle flavors, roadmap and implementation documentation;
- `jasmin-abernathy/Vevak-website` — public website and private tester flow;
- `jasmin-abernathy/vevak-docs` — this documentation repository;
- `jasmin-abernathy/vevak-brand` — brand and design guidance.

The Android application deliberately remains a **single repository**. The `foss` and `play` variants share the same tested core and isolate proprietary dependencies through Gradle source sets/dependency scopes. This reduces duplication and prevents security or reliability fixes from drifting between copies.

See:

- `architecture/ADR-001-single-android-repository.md`;
- `architecture/ADR-002-location-resilience.md`.

## Support the project

VeVak is intended to remain usable without a donation. Voluntary contributions help cover development, real-device testing, documentation and maintenance time, but they do not unlock features, security capabilities or priority access.

- [Support VeVak / Soutenir VeVak](https://vevak.lepotager.org/soutenir/)

## Safety wording

VeVak is not an emergency service and must never be the user's only safety mechanism. It does not guarantee SMS delivery, a fresh location fix or background execution. When a remembered position is used, its age must remain visible to the recipient.

## Licence

See `LICENSE`.
