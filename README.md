<!--
  SPDX-FileCopyrightText: 2026 Kubuno contributors
  SPDX-License-Identifier: AGPL-3.0-or-later
-->

<div align="center">

<img src="https://raw.githubusercontent.com/kubuno/core/main/.github/logo.png" alt="Kubuno logo" width="120">

# Kubuno — Mobile

[![License: AGPL v3](https://img.shields.io/badge/License-AGPL_v3-blue.svg)](LICENSE)
![Android](https://img.shields.io/badge/Android-Kotlin_·_Compose-3DDC84.svg)
![iOS](https://img.shields.io/badge/iOS-planned-lightgrey.svg)
![Status](https://img.shields.io/badge/status-alpha-yellow.svg)

**The native mobile apps of [Kubuno](https://github.com/kubuno/core) — the self-hosted, libre (AGPLv3) cloud platform, a sovereign alternative to Google Workspace and Microsoft 365.**

</div>

---

This repository gathers Kubuno's native phone and tablet apps, one folder per operating
system — the same layout as the [desktop](https://github.com/kubuno/desktop) repository.
Each app talks to a Kubuno instance over the platform's public API
([OpenAPI specification](https://github.com/kubuno/api-spec)) and keeps working offline.

| Folder | Platform | Status | Details |
|---|---|---|---|
| [`android/`](android/README.md) | Android (Kotlin, Jetpack Compose) | Alpha — six apps | [android/README.md](android/README.md) |
| [`ios/`](ios/README.md) | iOS / iPadOS | Planned | [ios/README.md](ios/README.md) |

## The apps

| App | Android | iOS |
|---|---|---|
| **Kubuno Drive** — files, offline copies, uploads | Yes | Planned |
| **Kubuno Photos** — photo library and backup | Yes | Planned |
| **Kubuno Documents** — office documents | Yes | Planned |
| **Kubuno Mail** — offline-first mail | Yes | Planned |
| **Kubuno Maps** — maps and saved places | Yes | Planned |
| **Kubuno Messages** — chat, calls and meetings | Yes | Planned |

On Android the apps share **one set of device accounts**: sign in once in any Kubuno app and
the others reuse the account, borrowing short-lived access tokens without ever holding your
refresh token. Push notifications use UnifiedPush — no proprietary push service.

## Repository layout

```
mobile/
├── android/   Gradle project: shared core modules (:core-*) and the apps (:app-*)
├── ios/       iOS apps (planned)
└── .github/   CI — builds and releases the Android apps
```

## Releases

Each app is released on its own: a tag `<app>-v<version>` (for example `drive-v1.2.0`)
builds that app and attaches it to a GitHub Release. Notes are generated from the commits.

## Security

Please report vulnerabilities privately — see [`SECURITY.md`](SECURITY.md).

## Contributing

Issues and pull requests are welcome. For any significant change, please open an issue first.

## License

[AGPL-3.0-or-later](LICENSE) © Kubuno contributors.
