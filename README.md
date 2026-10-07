<!--
  SPDX-FileCopyrightText: 2026 Kubuno contributors
  SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Kubuno — Mobile (moved)

**This repository is no longer developed.** In October 2026 its content moved, with its history, to the
repositories it belongs to, following the platform layout of every Kubuno repository (`common/`, `android/`, `ios/`):

| What | New home |
|---|---|
| Shared libraries (`core-api`, `core-account`, `core-ui`, `core-viewer`, `core-vectors`) | [`kubuno/core`](https://github.com/kubuno/core), folder `mobile/` |
| Kubuno Drive (`app-drive`) and its sync engine (`core-sync`) | [`kubuno/drive`](https://github.com/kubuno/drive), folder `mobile/` |
| Kubuno Mail (`app-mail`) | [`kubuno/mail`](https://github.com/kubuno/mail), folder `mobile/` |
| Kubuno Maps (`app-maps`) | [`kubuno/maps`](https://github.com/kubuno/maps), folder `mobile/` |
| Kubuno Photos (`app-photos`) | [`kubuno/photos`](https://github.com/kubuno/photos), folder `mobile/` |
| Kubuno Messages (`app-chat`) | [`kubuno/chat`](https://github.com/kubuno/chat), folder `mobile/` |
| Kubuno Documents (`app-docs`) | [`kubuno/office`](https://github.com/kubuno/office), folder `mobile/` |

The apps now consume the shared libraries as published Maven artifacts (`com.kubuno.mobile:*`), and each app is
released from its module's repository with a `mobile-v<version>` tag (formerly `<app>-v<version>` here). The full
history stays available in this repository's past commits.

## License

[AGPL-3.0-or-later](LICENSE) © Kubuno contributors.
