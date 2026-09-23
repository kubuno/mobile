<!--
  SPDX-FileCopyrightText: 2026 Kubuno contributors
  SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Kubuno — iOS

> Part of the [Kubuno mobile](../README.md) repository.

**Status: planned.** No iOS app has been started yet; this folder reserves its place beside
[`android/`](../android/README.md) so both platforms share one repository, one release flow
and one set of conventions.

## What already exists to build on

- **The API client.** The [api-spec](https://github.com/kubuno/api-spec) repository publishes
  the OpenAPI specification of the platform and a generated **Swift package**
  (`clients/swift`), the starting point for the network layer.
- **The Android apps as the reference.** Their behaviour — shared accounts, offline-first
  sync with a delta feed and an outbox, token refresh, UnifiedPush-style notifications — is
  described in [`android/README.md`](../android/README.md) and is what the iOS apps are meant
  to match, with the platform's own conventions (Keychain for credentials, APNs for push).

## Contributing

Interested in building the iOS apps? Open an issue in this repository to discuss the plan
before starting.

## License

[AGPL-3.0-or-later](../LICENSE) © Kubuno contributors.
