# Kubuno for Android

The official native Android client for [Kubuno](https://github.com/kubuno/core), the
self-hosted, libre (AGPLv3) cloud platform — a sovereign, open-source alternative to
proprietary productivity suites.

This app is a **native Kotlin client for the Kubuno drive module**, in the spirit of the
Nextcloud mobile client: browse and manage your files, upload and download with resumable
transfers, keep files available offline, auto-upload your camera roll, and receive push
notifications — all against your own server.

## Status

Early development. Current milestone: **M5 — push notifications & release**.

| Milestone | Scope | Status |
|---|---|---|
| M1 | Project scaffold, native auth (login / TOTP / token rotation), onboarding UI | ✅ done |
| M2 | Offline-first file browser (Room + delta sync + thumbnails + WebSocket) | ✅ done |
| M3 | File actions, uploads (simple + chunked resumable), downloads, transfer queue | ✅ done |
| M4 | Camera-roll auto-upload, offline pins, settings | ✅ done |
| M5 | UnifiedPush notifications, deep links, i18n polish, release build | ⏳ |

## Architecture

Three Gradle modules:

- **`:core-api`** — pure-JVM HTTP client (OkHttp 5 + Retrofit 3 + kotlinx.serialization).
  Owns the `TokenManager`, a careful port of the battle-tested refresh state machine from
  the desktop sync engine (`kubuno/desktop`): single-flight rotation, fresh-token adoption,
  transient-failure cooldown, and atomic persistence of rotated refresh tokens (the server
  rotates the refresh token on every native refresh and detects token reuse).
- **`:core-sync`** — Android library: Room database (source of truth for the UI), delta
  sync engine, offline outbox, WorkManager workers, chunked uploader. Lands in M2/M3.
- **`:app`** — Jetpack Compose (Material 3) UI, Hilt wiring, notifications, UnifiedPush
  receiver.

Design principles:

- **Offline-first**: the UI reads Room only; the server is reconciled through the drive
  module's cursor-based delta endpoint (`GET /api/v1/drive/sync/delta`).
- **Conflict-safe writes**: content updates send `If-Match` etags; a 412 produces a
  Nextcloud-style conflict copy instead of overwriting anyone's data.
- **Idempotent mutations**: every write carries an `Idempotency-Key`, so the offline
  outbox can replay safely.
- **De-googled**: push notifications use [UnifiedPush](https://unifiedpush.org/) (e.g. via
  ntfy), matching the server's push worker. No proprietary push services.
- Tokens are stored encrypted with an Android Keystore AES-GCM key; user-installed CAs are
  trusted so self-hosted instances with private CAs work out of the box.

## Building

Requirements: JDK 17+ (Android Studio's JBR works), Android SDK (compileSdk 36).

```bash
./gradlew :app:assembleDebug     # debug APK
./gradlew :core-api:test         # JVM unit tests (auth state machine)
```

The debug APK lands in `app/build/outputs/apk/debug/`.

## License

GNU Affero General Public License v3.0 — see [LICENSE](LICENSE).
