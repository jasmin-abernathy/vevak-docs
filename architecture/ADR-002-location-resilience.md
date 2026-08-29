# ADR-002 — Location resilience without bypassing Android

- **Status:** Accepted
- **Date:** 2026-08-29
- **Applies to:** `jasmin-abernathy/vevak` 0.3.1+

## Context

Real-device testing exposed a failure mode in the first trusted-place implementation:

1. VeVak could register the user's home Wi-Fi while Android Location was enabled;
2. the user could later switch Android's global Location control off;
3. Android then redacted the SSID, causing the trusted-place comparison to fail;
4. Android location providers were also unavailable and their last-location cache could be empty;
5. VeVak therefore answered `Position indisponible` even though useful local context had existed moments earlier.

Android's documented behaviour is intentional. Location-sensitive Wi-Fi identifiers and scan results are gated by location permissions/settings, and disabling Location can clear the platform fused-location cache. A normal third-party application must not claim it can silently turn these services back on.

A product requirement added during beta validation is that **users must not be forced to activate Android Location merely to register the Wi-Fi connection they are currently using as Maison**. Location should instead be presented as an optional reliability improvement.

## Decision

VeVak uses graceful degradation instead of trying to bypass Android.

### 1. Trusted Wi-Fi: two levels of identification

#### Session-only mode — no Location required

When the user chooses the current Wi-Fi as Maison while Android does not expose the SSID, VeVak does not fail and does not ask the user to enable Location first.

It stores, in app-private runtime state only:

- a hash of Android's opaque `Network.networkHandle` for the current active Wi-Fi session;
- Android's read-only `Settings.Global.BOOT_COUNT`.

The persistent settings store only a non-sensitive marker saying that Maison is configured in session-only mode. The actual network-session identifier is not included in `.vvk` backups.

This mode recognises Maison only while both conditions remain true:

- the exact Android Wi-Fi network session is unchanged;
- the phone remains in the same boot.

A disconnect/reconnect, network-session replacement or reboot invalidates the shortcut. VeVak fails closed rather than guessing from IP addresses, gateway addresses or other common network properties.

#### Durable mode — Location optional but useful

When Android exposes the SSID, VeVak hashes it with SHA-256 and stores only that hash. The SSID itself is never stored in clear text.

Android treats SSID access as location-sensitive. Therefore enabling precise Location permission and the global Android Location switch can allow VeVak to make Maison durable across Wi-Fi reconnections and device reboots. The UI presents this as an **optional improvement**, not a prerequisite for registering the current connection.

At the same time, VeVak stores an app-private hash of Android's opaque `Network.networkHandle` for that already verified network session, together with Android's boot count. If Android later redacts the SSID because Location was switched off, VeVak may continue to recognise the trusted place while that same network session remains active.

A visible SSID non-match always wins for durable configurations. A session marker is never allowed to override evidence that the phone is connected to a different readable Wi-Fi network.

This requires `ACCESS_NETWORK_STATE`, a normal read-only Android permission. Reading the global boot count also requires no write access. The canonical FOSS variant still does not request `INTERNET`.

### 2. VeVak-owned last-location memory

VeVak stores one last successfully acquired, non-mocked position in its private app storage:

- latitude;
- longitude;
- optional accuracy;
- acquisition timestamp.

The memory has a hard maximum retention of **24 hours**. It is not exported in `.vvk` backups, not placed in diagnostics, not written to logs and not sent anywhere except when it is actually selected for an authorised SMS/manual response.

The age of a remembered position is preserved and shown in the response. Reading the remembered point must never refresh its timestamp.

### 3. Resolution order

For normal requests the implementation should prefer:

1. trusted-place label when positively recognised;
2. freshest acceptable cache from Android or VeVak's bounded local memory;
3. bounded current-location lookup when providers are enabled;
4. freshest stale fallback when the user has allowed stale fallback;
5. explicit unavailable result when none of the above is safe to use.

The duress/safety path remains separate and must not inspect trusted Wi-Fi, the real location providers or VeVak's remembered real position.

## Why not implement a radio-fingerprint engine now?

Projects such as Déjà Vu / Local NLP and NeoStumbler demonstrate useful local learning from Wi-Fi/cellular observations. This remains a valid P1 research direction, but it does **not** solve the immediate Android Location-off regression by itself: current Android versions still gate Wi-Fi scans and many radio identifiers behind location-related permissions/settings.

Adding more radio collection now would increase permission surface, sensitive local state and test complexity without guaranteeing that the relevant observations remain readable when Location is disabled.

Therefore radio-environment learning is deferred until real-device evidence shows a concrete reliability benefit that justifies those costs.

## Consequences

### Positive

- users can register the current Wi-Fi connection as Maison without enabling Android Location;
- Location is now an optional reliability enhancement rather than a setup prerequisite for the current session;
- switching Location off no longer necessarily destroys every useful fallback;
- the trusted-place shortcut survives the same Wi-Fi session within the same boot;
- durable SSID hashing remains available when the user chooses to permit it;
- VeVak is independent from Android's volatile location cache for up to 24 hours;
- no Internet permission or central location service is added;
- stale data is explicitly aged instead of masquerading as current.

### Negative / limits

- session-only Maison does not survive a Wi-Fi reconnect or reboot;
- VeVak still cannot obtain a genuinely new GPS/network location while Android Location is disabled;
- durable trusted-place recognition across reconnect/reboot still depends on Android exposing the SSID at least once;
- the app now deliberately retains one precise coordinate locally for up to 24 hours;
- real-device validation remains required across Android versions and manufacturer network/background implementations.

## Required regression tests

- with Location disabled, register the currently active Wi-Fi as Maison: configuration must succeed in session-only mode;
- while that exact Wi-Fi session remains active, a normal request should return the Maison response;
- disconnect/reconnect while Location stays off: session-only Maison must fail closed;
- reboot while Location stays off: the pre-reboot session-only Maison must fail closed;
- enable Location and re-register Maison: the configuration should upgrade to durable SSID-hash mode when Android exposes the SSID;
- register trusted Wi-Fi with Location on, then switch Location off without disconnecting Wi-Fi: durable trusted-place continuity should continue for the same session;
- switch to another readable Wi-Fi: the old trusted network must not match;
- obtain a real location, switch Location off, then request again: an explicitly aged VeVak remembered point may be used;
- wait beyond the 24-hour retention window: remembered point must no longer be returned;
- mocked positions must never enter persistent memory;
- duress request must never use the trusted-network or remembered-real-location paths.
