# Rancher Config Reloader Fork

SUSE-based fork of `kube-logging/config-reloader` with automated rebuilds and security workflows.

## Overview

This fork maintains a **frozen code version** for stability while providing security
updates through rebuilds. We replace upstream mirrored images with Rancher-built SUSE
images that automatically rebuild when:

- New Go compiler releases are published (fixes stdlib CVEs)
- New SUSE BCI base images are available (fixes OS-level CVEs)
- Go module dependencies have security updates (fixes dependency CVEs)

**Security Model:**
- ✅ Security through fresh builds with updated dependencies
- ❌ We do NOT cherry-pick code changes from newer upstream versions
- ❌ Application code stays frozen for compatibility with the rancher-logging 4.10 chart

## Build pipeline

| Layer | Mechanism | What it owns |
|---|---|---|
| Daily cron | `.github/workflows/auto-update-go.yaml` (~10:07 UTC) | Bumps Go stdlib |
| Daily cron | `.github/workflows/auto-update-bci.yaml` (~12:13 UTC) | Bumps SUSE BCI digest |
| Continuous | `renovate.json5` | Go module updates (auto-merge on vuln + patch) |
| Triggered | `.github/workflows/cve-response.md` (agentic) | Long-tail CVE fixes (majors, transitives, multi-module) |
| Weekly | `.github/workflows/weekly-health-check.md` (agentic) | Meta-monitor — catches when automation stalls |
| Push/Tag/PR | `.github/workflows/build.yaml` | Multi-arch SUSE image build (goreleaser) |

## Coexistence with upstream

Upstream's distroless build (`Dockerfile` + `.github/workflows/artifacts.yaml`) is
left in place. Our SUSE pipeline (`Dockerfile.suse` + `build.yaml`) runs in
parallel. Consumers can pick either image set.

## Local build

```bash
# Install goreleaser
brew install goreleaser  # macOS

# Build snapshot for local testing
./scripts/build.sh

# Custom repository name
GITHUB_REPOSITORY=myorg/config-reloader ./scripts/build.sh
```

## Images

Built images push to GHCR: `ghcr.io/manno/config-reloader`

- `latest` — Latest build from `rancher-main`
- `<version>-suse1` — Release tags (e.g., `0.0.7-suse1`)
- `dev-<commit>` — Development builds

## Chart consumption

The rancher-logging 4.10 chart references this image as `images.config_reloader`
in `packages/rancher-logging/4.10/generated-changes/patch/values.yaml.patch`
(ob-team-charts repo). Once images are stable, that patch will be updated to
point at `ghcr.io/manno/config-reloader` instead of the upstream mirror.

## Upstream

- **Upstream**: https://github.com/kube-logging/config-reloader
- **Fork point**: `v0.0.7` (code frozen at this version)
- **Sync strategy**: None — we do NOT sync code changes from upstream
- **Security strategy**: Rebuild with fresh dependencies, not upstream patches

## References

- POC state: `docs/logging/fork/STATE.md` in `ob-team-charts`
- Sibling forks: `manno/logging-operator`, `manno/fluent-bit`, `manno/fluentd`
