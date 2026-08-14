# Plan: GitHub workflow to build package (issue #7)

## Approach

Add a single GitHub Actions workflow file under `.github/workflows/` that
mirrors the manual build steps documented in `README.md`, but automated:

1. Resolve the **latest stable OpenWrt release** version at runtime by
   querying the `openwrt/openwrt` GitHub releases API and filtering out
   pre-release / release-candidate tags (`*-rc*`).
2. Provision a prebuilt OpenWrt SDK for the **mediatek / filogic** target
   using the community-maintained [`openwrt/gh-action-sdk`](https://github.com/openwrt/gh-action-sdk)
   action, passing the resolved version plus target/subtarget.
3. Clone this repository into the SDK's `package/` feed.
4. Build only this package with
   `make package/luci-app-dnsmasq-ipset/compile V=s -j$(nproc)`.
5. Collect the resulting `*.ipk` (and `*.ipk` from `bin/`) and upload them as
   workflow artifacts.

The job is triggered **only manually** via `workflow_dispatch`, with input
overrides for `version`, `target` and `subtarget` defaulting to the behaviour
described above (latest release, mediatek, filogic).

## Affected files / modules

- **New:** `.github/workflows/build.yml` — the workflow.
- **New:** `specs/issue-7/spec.md`, `specs/issue-7/plan.md` — this spec/plan.

No changes to the package itself (`Makefile`, `files/`) are required; the
package already builds with `make package/<name>/compile` from a stock SDK.

## Key decisions

- **Use `openwrt/gh-action-sdk`** instead of hand-rolling SDK download/extraction.
  It handles tarball download, extraction, feeds setup and configuration caching,
  which keeps the workflow small and matches what the broader OpenWrt package
  ecosystem uses.
- **Resolve "latest release" at run time** via the GitHub API rather than
  hard-coding a version. This keeps the workflow correct over time without
  maintenance. Pre-releases are filtered so we always use a stable SDK.
- **Default target = mediatek/filogic** per the issue, but exposed as
  `workflow_dispatch` inputs so it can be rebuilt for other targets without
  editing the file.
- **Manual trigger only** (`workflow_dispatch`); no push/PR triggers, to keep
  CI cost minimal as the issue requested.
- **Artifacts only** — the workflow does not create a GitHub Release or push
  packages anywhere. That is out of scope for this issue.

## Risks / trade-offs

- The `openwrt/gh-action-sdk` action is a third-party action under the
  `openwrt` org. Pinning it to a version tag (rather than `@master`) reduces
  supply-chain risk; we pin to a specific tag.
- SDK download is large (~hundreds of MB); first run is slow, but cached by
  the action on subsequent runs for the same version.
- The package is architecture-independent (`PKGARCH:=all`), so the produced
  `.ipk` is effectively target-agnostic — the mediatek/filogic SDK is just a
  convenient, current SDK per the issue's default.
- "Latest release" detection depends on openwrt/openwrt releases being tagged
  consistently. The filter excludes `-rc*` and pre-release flags; if the
  OpenWrt project changes its tagging scheme the resolver may need updating.
