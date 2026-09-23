<!--
  SPDX-FileCopyrightText: 2026 Kubuno contributors
  SPDX-License-Identifier: AGPL-3.0-or-later
-->

<div align="center">

<img src="https://raw.githubusercontent.com/kubuno/core/main/.github/logo.png" alt="Kubuno logo" width="120">

# Kubuno — Android

[![License: AGPL v3](https://img.shields.io/badge/License-AGPL_v3-blue.svg)](../LICENSE)
![Kotlin](https://img.shields.io/badge/Kotlin-Jetpack_Compose-7F52FF.svg)
![Android](https://img.shields.io/badge/Android-compileSdk_36-3DDC84.svg)
![Push](https://img.shields.io/badge/push-UnifiedPush-4D38DB.svg)
![Status](https://img.shields.io/badge/status-alpha-yellow.svg)

**The native Android clients of [Kubuno](https://github.com/kubuno/core) — the self-hosted, libre (AGPLv3) cloud platform, a sovereign alternative to Google Workspace and Microsoft 365.**

</div>

---

> Part of the [Kubuno mobile](../README.md) repository — this folder is the Android
> project; the iOS apps will sit beside it in [`ios/`](../ios/README.md).

This folder hosts a suite of native Kotlin apps that share a common core and,
crucially, a single set of **device accounts**: sign in once in any Kubuno app and the
siblings reuse the same account through the Android `AccountManager`, borrowing
short-lived access tokens without ever holding your refresh token.

| App | Module | applicationId |
|---|---|---|
| **Kubuno Drive** | `:app-drive` | `com.kubuno.android` |
| **Kubuno Photos** | `:app-photos` | `com.kubuno.photos.android` |
| **Kubuno Documents** | `:app-docs` | `com.kubuno.docs.android` |
| **Kubuno Mail** | `:app-mail` | `com.kubuno.mail.android` |
| **Kubuno Maps** | `:app-maps` | `com.kubuno.maps.android` |
| **Kubuno Messages** | `:app-chat` | `com.kubuno.chat.android` |

---

## Kubuno Messages

A native client for the Kubuno chat module: conversations, groups and channels, with the
anatomy people already know from mainstream messengers and the Kubuno design system's own
skin.

- **Live by default** — one WebSocket to the chat module (`/api/v1/chat/ws`) carries every
  event about your account, so the list and the open conversation update together without
  polling and without per-conversation subscriptions.
- **Idempotent sending** — each message carries a client nonce that the module treats as an
  idempotency key, so a send interrupted mid-flight is retried without ever duplicating.
- **Notifications** — over [UnifiedPush](https://unifiedpush.org/), like every other Kubuno
  app. The payload is content-free by design: it names the sender or the group, never the
  message.

> **On encryption.** The chat module advertises Signal-style end-to-end encryption, but it
> does not implement it yet: message bodies travel as base64-encoded JSON, and the prekeys
> the module publishes are never consumed. This app therefore makes **no** encryption claim
> in its interface, and deliberately neither publishes nor fetches keys — registering a
> second key set would overwrite the one the web client uses. When the module ships a real
> protocol, the envelope encode/decode pair (`ChatEnvelope`) is the only place that changes.

### Milestones

| Milestone | Scope | Status |
|---|---|---|
| M1 | Conversation list (filters, archive, pin, unread), conversation reader, live socket, optimistic send | Done |
| M2 | Reply, edit, delete, forward, reactions, receipts, typing, in-chat search | Done |
| M3 | Media: gallery, camera, documents, voice messages | Done |
| M4 | New conversations and groups, mentions, polls, pinning, ephemeral messages, UnifiedPush | Done |
| M5a | Audio and video calls (WebRTC, mesh, interoperable with the web client) | Done |
| M5b | Offline Bluetooth relay between nearby devices | not started — see below |

**Calls** need the instance to publish ICE servers (`GET /chat/config`), and a
TURN relay for anything behind a real NAT. This client uses what the instance
configures and nothing else: with an empty list a call still connects between
directly reachable peers and fails visibly otherwise. It never falls back to a
third party's public STUN, which would hand the participants' addresses to a
stranger the instance did not choose.

> **A current browser cannot yet call an Android client.** Chrome 152 negotiates
> DTLS 1.3, and no published build of the Android WebRTC library completes that
> handshake with it: M125, M137 and M144 fail with `UNSUPPORTED_PROTOCOL`, and
> M150 — the only one that speaks DTLS 1.3 — gets through HelloRetryRequest and
> then fails inside BoringSSL with `WRONG_CURVE`. With DTLS 1.3 disabled in the
> browser the same call connects immediately, so everything above the handshake
> is proven: ICE pairs, the signalling is single-copy and correctly ordered, and
> the phone takes the answerer role. This is not specific to Kubuno — mobile
> WebRTC libraries trail Chrome — but until it clears upstream, calls between
> the web client and a phone should not be advertised as working. The library is
> pinned to M150 for that reason, and must be kept close to current: the peer at
> the other end of a call is a browser that updates itself.

**The Bluetooth relay is not started, deliberately.** Carrying messages through
a stranger's phone is only defensible once the module encrypts them, and today
it does not (see the encryption note above). The transport design is settled —
BLE dual-role advertising and GATT, controlled flooding with a TTL, a
store-and-forward queue — but it waits on a real protocol rather than shipping a
relay for plaintext.

---

## Kubuno Maps

A native maps client built on [MapLibre](https://maplibre.org/) — a libre renderer, not a
proprietary maps SDK.

- **Search and places** — geocoding and nearby points of interest through the maps module.
- **Directions** — walking, cycling, driving and transit routes, with a turn-by-turn
  navigation view.
- **GPX** — import, browse and follow recorded tracks, with an elevation profile.
- **Layers** — plan, satellite and a relief view built from terrain tiles shaded beneath
  the labels, so the map stays readable.

---

## Kubuno Mail

A native mail client for the Kubuno mail module, built for privacy and offline use.

- **Shared accounts** — consumes the accounts a sibling app (Drive) signed in, via the
  `com.kubuno` system authenticator. Mail never stores a refresh token; it borrows
  15-minute access tokens on demand.
- **Offline-first** — the inbox reads a local Room cache reconciled through the mail
  module's cursor-based delta endpoint (`/changes`). Threads, folders (Inbox, Starred,
  Sent, Drafts), the reader, and compose/send all work against the cache.
- **Compose & attachments** — write, reply, and send messages; download and open
  attachments through a `FileProvider`.
- **Push notifications** — new-mail alerts over [UnifiedPush](https://unifiedpush.org/)
  (e.g. via [ntfy](https://ntfy.sh/) or NextPush). No FCM, no proprietary push services.
  When a message arrives, the server's push worker delivers a compact payload to your
  distributor, and the app posts a notification that **deep-links straight into the
  thread** (`kubuno-mail://thread/<id>`).

Push is entirely optional: with no UnifiedPush distributor installed, the app registers
nothing and stays fully functional — everything except live notifications keeps working.

### Milestones

| Milestone | Scope | Status |
|---|---|---|
| M1 | Scaffold, shared-account consumption, inbox shell | Done |
| M2 | Offline-first thread list, reader, folders (Room + delta) | Done |
| M3 | Compose / reply / send, attachments | Done |
| M5 | UnifiedPush push, notifications, deep links, R8 release build | Done |

---

## Kubuno Photos

A native gallery for the Kubuno photos module.

- **Gallery and backup** — browse the library on the server, and back up the device's
  camera roll; a first-run welcome screen asks whether to switch backup on.
- **Capture and import** — take a photo or a video and upload it, or import pictures
  already on the device.
- **Info sheet** — swipe up on a photo for its details, with its location on a map.
- **Search** — free-text queries and category shortcuts.
- **Albums** — browse and open albums, add a selection, rename or delete an album,
  pick its cover (or let it choose one) and remove photos from it.
- **Sharing** — share one or several photos to other apps.

---

## Kubuno Documents

A native reader and light editor for the documents of the Kubuno office module.

- **Three entry points**, as on the web — recent documents, browse, templates.
- **Faithful rendering** — headings, inline marks, lists, tables, images, links, page and
  section breaks.
- **Light editing** — text, basic marks, lists and heading levels, then export to
  `.docx` or `.odt`.
- **Honest limits** — live collaboration and the web's full ribbon are out of scope; a
  block whose content the editor cannot place back exactly (footnotes, endnotes, inline
  images) is shown read-only rather than risk deleting what the user never saw.

---

## Kubuno Drive

A native client for the Kubuno drive module, in the spirit of the Nextcloud mobile app:
browse and manage files, upload and download with resumable chunked transfers, keep files
available offline, and auto-upload your camera roll — all against your own server. It is
offline-first (Room as the UI source of truth), conflict-safe (`If-Match` etags produce
Nextcloud-style conflict copies), and uses idempotent, replayable writes.

---

## Architecture

Gradle modules:

- **`:core-api`** — pure-JVM HTTP client (OkHttp 5 + Retrofit 3 + kotlinx.serialization).
  Owns the `TokenManager`: a battle-tested refresh state machine (single-flight rotation,
  fresh-token adoption, transient-failure cooldown, atomic persistence of rotated tokens).
- **`:core-account`** — the shared-account layer: the `com.kubuno` `AccountManager`
  authenticator, `SharedAccounts` discovery, and `BrokeredClients` (authenticated OkHttp
  clients that borrow access tokens for accounts owned by a sibling app).
- **`:core-ui`** — the shared Kubuno UI component library for Compose (design tokens,
  badges, callouts, selection controls, tabs, shell components), drawn to the web
  design system's own geometry.
- **`:core-sync`** — Room, delta sync engine, offline outbox, WorkManager workers, chunked
  uploader (drive).
- **`:core-viewer`** — the shared in-app viewers (image, PDF, text, audio/video), used by
  both drive and mail.
- **`:app-drive`**, **`:app-photos`**, **`:app-docs`**, **`:app-mail`**, **`:app-maps`**,
  **`:app-chat`** — the Compose apps, Hilt-wired, built on the shared Kubuno UI component
  library of `:core-ui`.

De-googled by design: push uses UnifiedPush, tokens are stored encrypted with an Android
Keystore AES-GCM key, and user-installed CAs are trusted so self-hosted instances with a
private CA work out of the box.

---

## Build

Requirements: a full JDK 17+ with `javac` (Android Studio's JBR works; a Java runtime alone
does not), Android SDK (compileSdk 36).

```bash
cd android                            # from the repository root
./gradlew :app-mail:assembleDebug     # debug APK — Kubuno Mail
./gradlew :app-drive:assembleDebug    # debug APK — Kubuno Drive
./gradlew :app-maps:assembleDebug     # debug APK — Kubuno Maps
./gradlew :app-chat:assembleDebug     # debug APK — Kubuno Messages
./gradlew :app-photos:assembleDebug   # debug APK — Kubuno Photos
./gradlew :app-docs:assembleDebug     # debug APK — Kubuno Documents
./gradlew :app-mail:assembleRelease   # minified (R8) release APK, unsigned
./gradlew :core-api:test              # JVM unit tests (auth state machine)
```

Each debug APK lands in `app-<name>/build/outputs/apk/debug/`. A release build is shrunk
and obfuscated by R8; see `app-mail/proguard-rules.pro` for the keep-rules audit.

## Releases

CI (`../.github/workflows/build.yml`, running in this folder) builds every app on push to `main` and pull requests.
A tag of the form `<app>-v*` (e.g. `mail-v0.1.0`, `drive-v1.2.0`) builds that app's
release APK and attaches it to a GitHub release.

## Security

Please report vulnerabilities privately — see [`SECURITY.md`](../SECURITY.md).

## Contributing

Issues and pull requests are welcome. For any significant change, please open an issue first.

## License

[AGPL-3.0-or-later](../LICENSE) © Kubuno contributors.
