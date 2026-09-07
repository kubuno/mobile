# Kubuno for Android

The official native Android clients for [Kubuno](https://github.com/kubuno/core), the
self-hosted, libre (AGPLv3) cloud platform — a sovereign, open-source alternative to
proprietary productivity suites.

This repository hosts a suite of native Kotlin apps that share a common core and,
crucially, a single set of **device accounts**: sign in once in any Kubuno app and the
siblings reuse the same account through the Android `AccountManager`, borrowing
short-lived access tokens without ever holding your refresh token.

| App | Module | applicationId |
|---|---|---|
| **Kubuno Drive** | `:app-drive` | `com.kubuno.drive.android` |
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
| M1 | Conversation list (filters, archive, pin, unread), conversation reader, live socket, optimistic send | ✅ done |
| M2 | Reply, edit, delete, forward, reactions, receipts, typing, in-chat search | ✅ done |
| M3 | Media: gallery, camera, documents, voice messages | ✅ done |
| M4 | New conversations and groups, mentions, polls, pinning, ephemeral messages, UnifiedPush | ✅ done |
| M5a | Audio and video calls (WebRTC, mesh, interoperable with the web client) | ✅ done |
| M5b | Offline Bluetooth relay between nearby devices | not started — see below |

**Calls** need the instance to publish ICE servers (`GET /chat/config`), and a
TURN relay for anything behind a real NAT. This client uses what the instance
configures and nothing else: with an empty list a call still connects between
directly reachable peers and fails visibly otherwise. It never falls back to a
third party's public STUN, which would hand the participants' addresses to a
stranger the instance did not choose.

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
| M1 | Scaffold, shared-account consumption, inbox shell | ✅ done |
| M2 | Offline-first thread list, reader, folders (Room + delta) | ✅ done |
| M3 | Compose / reply / send, attachments | ✅ done |
| M5 | UnifiedPush push, notifications, deep links, R8 release build | ✅ done |

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
- **`:core-ui`** — shared Compose design tokens and shell components.
- **`:core-sync`** — Room, delta sync engine, offline outbox, WorkManager workers, chunked
  uploader (drive).
- **`:core-viewer`** — the shared in-app viewers (image, PDF, text, audio/video), used by
  both drive and mail.
- **`:app-drive`**, **`:app-mail`**, **`:app-maps`**, **`:app-chat`** — the Compose
  (Material 3) apps, Hilt-wired, each with its own UnifiedPush receiver and notifications.

De-googled by design: push uses UnifiedPush, tokens are stored encrypted with an Android
Keystore AES-GCM key, and user-installed CAs are trusted so self-hosted instances with a
private CA work out of the box.

---

## Building

Requirements: JDK 17+ (Android Studio's JBR works), Android SDK (compileSdk 36).

```bash
./gradlew :app-mail:assembleDebug     # debug APK — Kubuno Mail
./gradlew :app-drive:assembleDebug    # debug APK — Kubuno Drive
./gradlew :app-maps:assembleDebug     # debug APK — Kubuno Maps
./gradlew :app-chat:assembleDebug     # debug APK — Kubuno Messages
./gradlew :app-mail:assembleRelease   # minified (R8) release APK, unsigned
./gradlew :core-api:test              # JVM unit tests (auth state machine)
```

Each debug APK lands in `app-<name>/build/outputs/apk/debug/`. A release build is shrunk
and obfuscated by R8; see `app-mail/proguard-rules.pro` for the keep-rules audit.

## Releases

CI (`.github/workflows/build.yml`) builds every app on push to `main` and pull requests.
A tag of the form `<app>-v*` (e.g. `mail-v0.1.0`, `drive-v1.2.0`) builds that app's
release APK and attaches it to a GitHub release.

## License

GNU Affero General Public License v3.0 — see [LICENSE](LICENSE).
