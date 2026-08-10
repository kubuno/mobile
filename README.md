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
- **`:app-drive`**, **`:app-mail`** — the Compose (Material 3) apps, Hilt-wired, each with
  its own UnifiedPush receiver and notifications.

De-googled by design: push uses UnifiedPush, tokens are stored encrypted with an Android
Keystore AES-GCM key, and user-installed CAs are trusted so self-hosted instances with a
private CA work out of the box.

---

## Building

Requirements: JDK 17+ (Android Studio's JBR works), Android SDK (compileSdk 36).

```bash
./gradlew :app-mail:assembleDebug     # debug APK — Kubuno Mail
./gradlew :app-drive:assembleDebug    # debug APK — Kubuno Drive
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
