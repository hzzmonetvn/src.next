# Kiwi Chromium 157 port

## Accepted scope

- Pin Android Stable Chromium 157.0.8103.50, observed on 2026-11-11.
- Preserve Chrome extensions, ad blocking, and dark mode.
- First APK architecture: arm64-v8a.
- Compile only on GitHub Actions runners, never on the development machine.
- Start with a standard GitHub-hosted runner and measure its resources first.
- Work on port/kiwi-chromium-157; preserve the kiwi branch.

## Decisions

### Measure before downloading the full source

Chromium's Android build instructions at the pinned tag require at least 100 GB free disk and 8 GB RAM, recommending more than 16 GB RAM. The first CI job records actual resources and fails explicitly below the documented minimum. It does not delete runner software or start an expensive checkout on an undersized runner.

Considered: immediately invoking the old APK workflows. Rejected because they call external Kiwi services or retrieve prebuilt objects rather than compiling source on the GitHub runner.

### Do not treat a sparse source overlay as a completed port

PR #1 is an earlier partial Chromium import, not an integrated Kiwi build. A complete checkout at the pinned upstream revision, dependency sync, reviewed Kiwi patches, and actual compilation are required. No APK or feature validation has been completed at this stage.

### Pre-build disk cleanup (2026-11-14)

Keep the initial disposable-runner SDK cleanup and add APT download/list cleanup after dependency installation and hooks, before build-cache restoration. Log available space before and after. Installed packages, Chromium source, pinned toolchains, and incremental output remain intact. APT lists can be restored with apt-get update; a fresh job restores the runner image.

Rejected: deleting checkout metadata, output caches, or disabling swap to gain space. Those changes would interfere with incremental build diagnosis or reduce memory headroom. This cleanup does not fix the previously observed rebuild/cache invalidation issue.

## Initial evidence

- First runner: 4 vCPU, 15 GiB RAM, 86 GiB free disk; failed the preflight disk threshold.
- Diagnostic run measured optional toolchains: dotnet 5.8 GiB, Android SDK 12 GiB, ghcup 3.7 GiB, hostedtoolcache 5.3 GiB. Remove only those disposable runner directories before remeasurement. Recovery is a fresh job runner; no local machine cleanup occurs.
- Original Kiwi base is b2a61e552c940b89d2ae504de720832780f4e7ea (105.0.5195.24). Its endpoint delta to origin/kiwi has 577 files, 13,810 insertions, 834 deletions. The similarly labelled 7011172b snapshot is not the correct ancestry baseline.
- Adblock menu/settings wiring is present, but complete filter-engine/list-update behavior remains unverified. Chromium 157 extension build flags differ from 105; setting enable_extensions alone is not a port.

## Verification still required

- Runner resource preflight.
- Determine Kiwi patch baseline and conflicts.
- Full upstream source and matching Android dependencies on CI.
- GN generation and arm64 APK compilation on CI.
- Installation and extension, ad-blocking, and dark-mode runtime checks.
