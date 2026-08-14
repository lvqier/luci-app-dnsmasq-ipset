# Issue #7: Add GitHub workflow config to build package

## Goal

Add a GitHub Actions workflow that builds the `luci-app-dnsmasq-ipset`
package using the OpenWrt SDK, so contributors can produce a ready-to-install
`.ipk` without setting up a local toolchain.

## Requirements (from issue)

- Use the **latest released** OpenWrt SDK (not snapshot, not a pinned old release).
- Default target: **mediatek / filogic**.
- Trigger **manually** (`workflow_dispatch`).
- Follow the build instructions described in `README.md` (build the package
  from the OpenWrt SDK).
