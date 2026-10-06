# Darkelf Shadow Browser — Security Architecture & Protective Features

**Project:** Darkelf Shadow Community Edition  
**Repository:** Darkelf-Labs/Darkelf-Shadow-CE  
**Architecture reviewed:** October 2026  
**Runtime:** Python 3.11+ · PySide6 · Qt WebEngine / Chromium  
**License:** LGPL-3.0-or-later

---

## 1. Overview

Darkelf Shadow is a privacy-focused browser built on PySide6 and Qt WebEngine. It combines Chromium-based rendering with a Darkelf application layer that adds request interception, content filtering, fingerprinting defenses, session-aware privacy controls, local threat analysis, compatibility handling, and runtime diagnostics.

Shadow is distributed in two materially different forms:

- **Python / PyPI builds** use the platform PySide6 / Qt WebEngine runtime and an off-the-record `QWebEngineProfile`.
- **Native macOS ARM64 builds** use Darkelf's custom Qt WebEngine 6.11.2 build and a named `Darkelf` profile to support native authentication behavior. The profile uses memory HTTP caching and nonpersistent cookies, with selected website storage removed during normal shutdown.

The native macOS profile is therefore not identical to the PyPI privacy model. Shadow's documentation treats these modes separately rather than claiming that both are off the record.

---

## 2. Core Design Principles

Shadow's current architecture is organized around several defensive principles:

1. **Reduce persistent browsing state where practical.**
2. **Intercept and classify network requests before loading.**
3. **Block supported advertising, tracking, telemetry, and nuisance rules.**
4. **Reduce fingerprinting surface through browser-side API modification.**
5. **Adapt privacy protections when compatibility-sensitive websites require native behavior.**
6. **Keep local threat analysis inside the browser process.**
7. **Separate privacy telemetry from claims of verified malware detection or system security.**
8. **Fail conservatively when a requested filter syntax is unsupported rather than misinterpreting it as a blocking rule.**
9. **Preserve compatibility through narrow, explicit exceptions rather than broad global allowlisting.**

---

## 3. Runtime and Browser Profile Architecture

Shadow creates its browsing profile during startup in `shadow/boot.py`.

### Python / PyPI and Source Launches

Standard installations create an unnamed:

```python
QWebEngineProfile(app)
```

This profile is off the record.

### Native macOS Application

The signed Darkelf macOS application creates:

```python
QWebEngineProfile("Darkelf", app)
```

when the runtime is positively identified as the expected macOS ARM64 app bundle.

The profile is configured with:

```python
profile.setHttpCacheType(QWebEngineProfile.MemoryHttpCache)
profile.setPersistentCookiesPolicy(QWebEngineProfile.NoPersistentCookies)
profile.setHttpAcceptLanguage("en-US,en;q=0.9")
```

Additional browser settings disable or restrict several capabilities, including:

- browser plugins;
- hyperlink auditing;
- JavaScript-created windows;
- JavaScript clipboard access;
- local-content access to remote URLs;
- local-content access to local files.

Local storage remains enabled because many modern sites depend on it.

### Shutdown Cleanup

For the native named profile, Shadow attempts to destroy browser pages and profile objects before removing selected website state during normal shutdown.

Documented cleanup targets include:

- localStorage;
- IndexedDB;
- service-worker state;
- browsing history;
- caches.

This is a best-effort cleanup model rather than a zero-trace guarantee. Crashes, forced termination, locked files, operating-system behavior, authentication state, explicitly retained files, and older profiles can leave data behind.

---

## 4. Request Interception Layer

Shadow installs `StealthInterceptor`, a `QWebEngineUrlRequestInterceptor`, on the shared profile.

This layer evaluates requests before the filtering engine is applied.

Its responsibilities include:

- request-type classification;
- first-party and requested-host comparison;
- MiniAI monitoring;
- Panic and Lockdown enforcement;
- URL tracking-parameter cleanup;
- narrow compatibility-resource exceptions;
- human-verification resource exceptions;
- filter-engine evaluation;
- selected request-header adjustments;
- Darkelf Quantum request-state updates.

### Tracking-Parameter Cleanup

For top-level documents, Shadow removes known tracking parameters including:

- `utm_source`
- `utm_medium`
- `utm_campaign`
- `utm_term`
- `utm_content`
- `fbclid`
- `gclid`
- `mc_eid`

The cleaned URL is then redirected before page loading continues.

### Local and Non-Web Schemes

The interceptor distinguishes safe internal schemes from network requests and explicitly blocks `file:` requests at the interceptor level.

Loopback and selected private IPv4 addresses are recognized separately from public hostnames.

---

## 5. Darkelf Standard Protection Filter Engine

Shadow implements its own EasyList-style parser and request matcher in `shadow/filters.py`.

The engine supports both:

- network blocking rules;
- cosmetic filtering rules.

Configured upstream subscriptions are fetched, cached, merged, compiled, parsed, and indexed into Darkelf's local **Darkelf Standard Protection** ruleset.

### Filter Compilation

The compiler:

- deduplicates repeated rules;
- preserves supported exception semantics;
- processes `$badfilter` cancellation directives;
- skips unsupported rule actions instead of treating them as URL blockers;
- preserves cosmetic rules separately from network rules;
- creates a locally compiled cache;
- verifies cached merged content using SHA-256 metadata before reuse.

### Rule Indexing

Network rules are indexed using:

- resource type;
- conservative literal tokens;
- fallback buckets for rules that cannot safely be indexed.

This reduces per-request scanning while preserving matching semantics.

### Unsupported Rule Syntax

Shadow deliberately skips unsupported actions such as selected:

- CSP modification;
- redirects;
- header modification;
- cookie modification;
- URL transforms;
- remove-parameter operations;
- unsupported scriptlet/page actions.

This prevents unsupported syntax from accidentally becoming an over-broad network block.

### Filter-List Retrieval Safeguards

Filter-list downloads include safeguards such as:

- allowed HTTP/HTTPS schemes only;
- private/internal-address rejection;
- response size caps;
- atomic cache replacement;
- preservation of the last usable cache when refresh fails.

---

## 6. Cosmetic Filtering

Darkelf Standard Protection also stores CSS cosmetic rules and exceptions.

After a page successfully loads, Shadow determines the active host and injects the applicable stylesheet into the document.

Cosmetic injection is skipped for compatibility-mode sites when Darkelf intentionally allows a more native Chromium environment.

This separates visual nuisance filtering from request blocking and reduces unnecessary CSS processing on sensitive authentication or compatibility sites.

---

## 7. Smart Canvas Architecture

Shadow implements a site-aware Canvas privacy policy in `HardenedWebPage`.

Four modes are defined:

| Mode | Behavior |
|---|---|
| **BLOCKED** | Canvas readback is disabled. |
| **PROTECTED** | Canvas readback remains available but Darkelf's deterministic protection is scheduled. |
| **COMPATIBLE** | Native readback is allowed because compatibility mode bypasses injected Canvas defenses. |
| **TRUSTED** | Native Canvas behavior is allowed for trusted or temporarily trusted hosts. |

Unknown hosts default to **BLOCKED**.

### Temporary Verification Trust

Shadow includes human-verification detection for common challenge providers.

When a recognized challenge is detected, the exact first-party host can receive temporary session Canvas trust to reduce repeated CAPTCHA or bot-verification loops.

This trust:

- applies only to the current Shadow session;
- is limited to the exact challenged host;
- is cleared when Shadow exits.

The detection signal comes from page behavior and is not cryptographic proof that a challenge is genuine.

---

## 8. Fingerprinting Defenses

`HardenedWebPage` installs privacy scripts at `QWebEngineScript.DocumentCreation` and runs them in subframes as well as the main document.

Current application-level defenses include protections or normalization for:

- Canvas;
- WebGL-facing JavaScript behavior;
- Audio fingerprinting;
- battery information;
- geolocation;
- WebRTC-facing APIs;
- media-device exposure;
- fonts;
- `hardwareConcurrency`;
- `deviceMemory`;
- screen dimensions;
- navigator/platform values;
- plugin and MIME-type exposure;
- iframe environment consistency.

### Screen Normalization

Shadow reports a normalized screen environment while allowing Chromium to retain the real viewport dimensions.

The current JavaScript policy uses a normalized:

```text
1440 × 900
```

screen value.

### Hardware Characteristics

The browser modifies or limits selected high-entropy values such as:

- `navigator.deviceMemory`;
- `navigator.hardwareConcurrency`;
- `navigator.platform`;
- `navigator.languages`;
- `navigator.maxTouchPoints`.

Some values are domain-sensitive or randomized in order to reduce a single globally stable fingerprint.

---

## 9. WebRTC Protection

Shadow applies application-level WebRTC defenses during document creation.

The JavaScript layer attempts to neutralize or remove access to:

- `RTCPeerConnection`;
- `webkitRTCPeerConnection`;
- `mozRTCPeerConnection`;
- `RTCDataChannel`;
- `navigator.mediaDevices`.

A secondary WebRTC routine also attempts to sanitize SDP output if an `RTCPeerConnection` implementation remains available.

The custom macOS DMG engine additionally includes native WebRTC changes according to the project documentation.

Standard PyPI installations should therefore not be assumed to have the same native engine protections as the custom DMG.

---

## 10. Geolocation and Battery Defenses

Shadow removes normal JavaScript access to `navigator.geolocation` and modifies geolocation permission queries to report denial.

Battery access is also modified through a dedicated injected defense when `navigator.getBattery` is available.

These measures reduce direct exposure of location and power-state characteristics to page scripts.

They do not prevent a website from estimating location by other means such as:

- IP address;
- account information;
- language;
- time zone;
- behavioral data.

---

## 11. WebGL and Graphics Fingerprinting

Shadow includes JavaScript-level WebGL fingerprint defenses in `browser_page.py`.

The project also documents native WebGL modifications in the custom macOS engine.

These are separate layers:

- **application-level JavaScript defenses** are part of the Shadow Python source;
- **native WebGL engine modifications** apply only where the custom engine is actually present.

Shadow's own privacy-status reporting intentionally avoids claiming that native WebGL or WebRTC protections have been verified merely because application-level code is scheduled.

---

## 12. Compatibility Mode

A major part of Shadow's privacy architecture is controlled compatibility bypass.

Some authentication-heavy, messaging, media, and challenge-sensitive domains are allowed to use a more native Chromium environment.

Compatibility mode can bypass injected fingerprint modifications for those documents and their related challenge/authentication frames.

Current categories include selected:

- Google / Gmail / Workspace infrastructure;
- GitHub;
- Apple authentication;
- Microsoft / Outlook authentication;
- TikTok;
- Zalo;
- WhatsApp;
- Discord;
- Spotify;
- SourceForge;
- Darkelf's own website;
- other explicitly configured compatibility hosts.

Network exceptions are also narrowly scoped to defined first-party/resource-host relationships.

This design intentionally trades some fingerprint resistance for functionality on those sites rather than globally weakening protections.

---

## 13. Human-Verification Compatibility

Shadow recognizes resources associated with common verification systems such as:

- hCaptcha;
- reCAPTCHA;
- Cloudflare challenges;
- Arkose / FunCaptcha;
- AWS WAF token infrastructure.

Those challenge resources can bypass normal filter evaluation where necessary.

The goal is to prevent login, signup, checkout, and other user flows from entering verification loops while avoiding broad allowlisting of entire provider ecosystems.

---

## 14. Popup and Secondary-Navigation Control

`SecondaryNavigationPage` distinguishes between:

- genuine user-clicked links;
- script-generated secondary navigation.

User-clicked secondary links are redirected into the current Shadow tab.

Script-created popups and popunders are blocked.

This provides a browser-level popup policy without requiring site-specific popup rules.

---

## 15. Local MiniAI Sentinel

Shadow initializes `DarkelfMiniAISentinel` during browser startup.

MiniAI operates locally and maintains bounded session telemetry for browser activity.

It monitors characteristics such as:

- tracker-related URLs;
- suspicious URL patterns;
- fingerprinting indicators;
- request bursts;
- redirect anomalies;
- selected malware/exploit/intrusion keywords;
- HTTP-upgrade activity.

The system tracks statistics including:

- unique domains;
- tracker events;
- suspicious events;
- fingerprint attempts;
- intrusion indicators;
- malware/exploit indicators;
- request anomalies;
- threat score;
- current Lockdown state;
- current Panic state.

MiniAI is a heuristic browser-defense system. Its labels are indicators generated from Darkelf's rules and should not be interpreted as proof that a site or resource is malicious.

---

## 16. Lockdown and Panic Modes

MiniAI can escalate from passive monitoring into two request-blocking states.

### Lockdown

Lockdown is triggered only after multiple recent critical events meet the configured threshold.

When Lockdown is active, the request interceptor blocks outgoing requests.

### Panic

Panic is a higher escalation state triggered after a larger number of qualifying critical events within the configured time window.

When Panic is active, the interceptor blocks requests before ordinary compatibility or filtering exemptions are considered.

The Inspector distinguishes inactive and triggered states using:

- **STANDBY**
- **ACTIVE**

These states describe Darkelf's own defensive logic, not an operating-system security state.

---

## 17. Darkelf Quantum Session State

Shadow includes `DarkelfPQ`, also surfaced as **Darkelf Quantum** in the user interface.

Despite the filename and historical terminology, the current implementation is primarily a session-integrity and state-chaining mechanism using standard Python cryptographic primitives such as:

- `secrets`;
- SHA3-512;
- HMAC;
- a SHA3-based HKDF-style derivation routine.

The system maintains:

- per-tab random seeds;
- per-tab counters;
- Canvas seeds;
- a rolling SHA3 state chain;
- bounded observed-request fingerprints;
- generation and rekey counters;
- watchdog state.

Only selected request categories are observed to avoid hashing every static resource.

### Lifecycle

Quantum state is session-oriented.

On browser shutdown, Shadow calls the Quantum destruction path, which clears references to:

- tab seeds;
- Canvas seeds;
- counters;
- observed fingerprints;
- the active chain.

This makes the state eligible for ordinary Python memory reclamation.

It does **not** guarantee physical RAM zeroization.

### Terminology Limitation

The current `darkelf_pq.py` implementation should not be described as a post-quantum key-exchange or post-quantum transport protocol.

The current code uses SHA3/HMAC/session-state techniques; it does not itself implement ML-KEM/Kyber or another post-quantum public-key exchange.

---

## 18. WebAuthn and Passkeys

Shadow includes Qt WebEngine WebAuthn handling for passkeys and hardware authenticators.

The browser can respond to authentication states involving:

- account selection;
- PIN collection;
- authenticator interaction;
- completion;
- cancellation;
- retry/failure states.

The native macOS distribution uses the named profile partly to support platform authentication behavior.

Actual Touch ID or passkey availability still depends on:

- Qt/Chromium support;
- application signing;
- keychain access groups and entitlements;
- macOS provisioning;
- authenticator support;
- website implementation.

WebAuthn support should therefore not be treated as a guarantee that every provider will authenticate successfully.

---

## 19. Media and Codec Scope

The custom macOS Shadow distribution includes H.264/AVC support.

H.264 support is separate from DRM.

Darkelf Shadow does **not** bundle Widevine and should not be described as providing Widevine DRM merely because H.264 playback works.

Media compatibility can also vary between the custom native engine and a normal PyPI installation using the platform Qt WebEngine package.

---

## 20. Browser UI Isolation and View Source

Shadow implements browser-owned interfaces using Qt widgets rather than rendering all privileged controls inside ordinary web content.

Examples include:

- toolbar;
- tab bar;
- download shelf;
- find bar;
- Inspector;
- settings interfaces;
- authentication prompts.

### View Source

Shadow's View Source implementation independently fetches HTTP/HTTPS source with explicit controls.

It:

- accepts only HTTP and HTTPS URLs;
- rejects embedded credentials;
- limits redirects;
- limits source size to 8 MiB;
- rejects unsupported schemes such as `file:`;
- displays fetched HTML as literal text rather than executing it.

This keeps View Source behavior separate from normal page rendering.

---

## 21. Darkelf Inspector

The current Inspector exposes:

- **Console**
- **Quantum**
- **MiniAI**
- **Shortcuts**
- **Help**

The Inspector is intended to expose Darkelf's own runtime information and diagnostics.

Quantum status and MiniAI statistics are operational telemetry. They are not equivalent to an external audit, malware verdict, or cryptographic attestation.

---

## 22. Downloads and Session Artifacts

Shadow creates its download directory lazily when a download is initiated.

The browser also contains cleanup logic for selected download traces and WebEngine state.

However, downloaded files are intentionally persistent user artifacts unless explicitly removed.

Likewise, the following can persist independently of ephemeral browsing state:

- downloaded files;
- filter caches;
- compiled filter data;
- saved snapshots;
- application settings;
- authentication material;
- operating-system logs or caches.

Shadow therefore does not claim zero-disk-trace operation.

---

## 23. Native macOS Engine vs Standard PyPI Engine

A central architectural distinction is that the open Python application layer and the native custom engine are not the same thing.

### Standard PyPI Build

Provides:

- PySide6 / Qt WebEngine supplied by the platform package;
- off-the-record profile;
- application-level request interception;
- application-level filtering;
- JavaScript-level privacy defenses;
- MiniAI;
- Darkelf Quantum.

### Native macOS ARM64 Build

Adds the custom Darkelf Qt WebEngine 6.11.2 distribution and platform integration, including project-documented native modifications such as:

- WebGL-related engine changes;
- WebRTC changes;
- H.264 support;
- native WebAuthn/platform authentication integration.

The custom engine is not distributed through the ordinary PyPI package.

---

## 24. Security Boundaries and Limitations

Shadow is designed to reduce common browser privacy and tracking exposures, but it is not an anonymity system and does not eliminate all fingerprinting or forensic traces.

Important limitations include:

- compatibility mode intentionally exposes a more native environment on selected sites;
- JavaScript API modifications can themselves be fingerprintable;
- native custom-engine protections do not apply automatically to standard PyPI installations;
- IP addresses remain visible to remote network peers unless the user employs a separate network-privacy layer;
- downloaded files and deliberately retained data remain persistent;
- operating-system telemetry and caches are outside Shadow's complete control;
- crash cleanup is inherently less reliable than normal shutdown;
- filter syntax support is deliberately incomplete;
- unsupported shared-hosting/site-boundary cases can remain;
- heuristic MiniAI detections can produce false positives or miss threats;
- browser defenses cannot replace operating-system patching, endpoint security, account security, or safe browsing practices.

---

## 25. Layered Architecture Summary

Shadow's current defensive flow can be summarized as:

```text
User Navigation
      │
      ▼
Darkelf Browser UI
      │
      ▼
HardenedWebPage
  ├─ Canvas policy
  ├─ fingerprint defenses
  ├─ WebRTC / geolocation / battery defenses
  ├─ compatibility mode
  └─ WebAuthn handling
      │
      ▼
Qt WebEngine Profile
      │
      ▼
StealthInterceptor
  ├─ MiniAI monitoring
  ├─ Lockdown / Panic enforcement
  ├─ tracking-parameter cleanup
  ├─ challenge compatibility
  ├─ site compatibility rules
  ├─ Darkelf Quantum updates
  └─ Darkelf Standard Protection
      │
      ▼
Qt WebEngine / Chromium Network Stack
      │
      ▼
Remote Site
```

The major subsystems are intentionally layered rather than treated as one universal security mechanism.

---

## 26. Source Modules

Key implementation files in the current Shadow repository include:

```text
shadow/
├── boot.py
├── browser.py
├── browser_downloads.py
├── browser_features.py
├── browser_homepage.py
├── browser_icons.py
├── browser_page.py
├── browser_ui.py
├── cli.py
├── darkelf_context_menu.py
├── darkelf_inspector.py
├── darkelf_pq.py
├── filters.py
├── interceptor.py
├── miniai.py
├── settings_dialog.py
├── settings_pages.py
├── splash.py
└── utils.py
```

The most security-relevant application-layer modules are:

- `boot.py` — profile and browser startup configuration;
- `browser_page.py` — WebEngine page policy and fingerprinting defenses;
- `interceptor.py` — request interception and enforcement;
- `filters.py` — network and cosmetic filtering;
- `miniai.py` — local heuristic threat monitoring;
- `darkelf_pq.py` — session-state chaining and runtime state;
- `browser.py` — tab lifecycle, shutdown, source viewing, and browser integration.

---

## 27. Security Claim Scope

This document describes the architecture visible in the current open-source Shadow repository and the behavior explicitly documented by that project.

It does not constitute:

- an independent penetration test;
- a formal cryptographic audit;
- proof that every native engine patch is active in every build;
- a guarantee of anonymity;
- a guarantee that browser state is physically erased;
- a guarantee that all tracking, fingerprinting, malicious content, or exploits will be detected or blocked.

Runtime behavior can also differ according to Qt version, operating system, build configuration, website behavior, and whether the native Darkelf engine is present.

---

## 28. Project Information

**Darkelf Shadow Community Edition**  
**Darkelf Labs**

Website: https://darkelfbrowser.com  
Repository: https://github.com/Darkelf-Labs/Darkelf-Shadow-CE

For current behavior, the repository source and latest release documentation take precedence over historical Darkelf documents.

**Copyright © 2024–2026 Dr. Kevin Moore / Darkelf Labs**
