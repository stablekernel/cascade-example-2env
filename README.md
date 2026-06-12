# cascade-example-2env

A reference implementation of a two-environment release pipeline using Cascade. This repository demonstrates how to orchestrate builds across multiple architectures, gate deployments by environment, and publish artifacts at a defined boundary.

## What This Repository Shows

This scaffold implements a realistic multi-environment CI/CD workflow:

- **Matrix Builds**: A container image build matrix that targets both `linux/amd64` and `linux/arm64`, with artifact passing to dependent builds
- **Environment Progression**: Automated promotion from staging to production with release gates
- **Publish Boundary**: The prod environment marks the release boundary; prerelease tags in staging become published releases when promoted to prod
- **State Tracking**: Cascade's manifest-based state tracking ensures auditability and enables manual intervention

The pipeline is triggered by commits to `src/` and includes two key workflows:
- **Orchestrate**: Builds all artifacts and deploys to staging
- **Promote**: Manually gates and promotes staging artifacts to production

## Getting Started

### Prerequisites
- A GitHub repository with Actions enabled
- Cascade CLI v0.1.0 or later (see [stablekernel/cascade](https://github.com/stablekernel/cascade))
- For state persistence: a GitHub token with `contents:write` scope (passed via `secrets.CASCADE_STATE_TEST_TOKEN`)

### Setup

The scaffold is ready to use after cloning:

1. **Review the manifest**: `.github/manifest.yaml` defines environments, builds, and state tracking
2. **Customize workflows**: Modify build steps in `.github/workflows/build-*.yaml` to match your application
3. **Set the state token**: Add a personal access token or fine-grained token to `Settings → Secrets and variables → Actions → New repository secret` as `CASCADE_STATE_TEST_TOKEN` (if you want state commits to appear under a specific user, else leave empty for `github-actions[bot]`)

### Triggering Pipelines

**Automatic (Orchestrate on commit):**
Push a change to any file matching `src/**`:
```bash
echo "v1.0.0" > src/version.txt
git add src/version.txt
git commit -m "bump: v1.0.0"
git push origin main
```

The `Orchestrate` workflow will start automatically, build the matrix, and deploy to staging.

**Manual (Promote to production):**
After staging is verified, promote to production:
1. Go to `Actions → Promote`
2. Click "Run workflow"
3. Select `mode: staging-to-prod`
4. Review the preflight and click through

The promotion will gate on the prod environment, create a published release (removing the RC tag), and trigger any publish callbacks.

## What the Test Driver Validates

`.github/workflows/scenario-suite.yaml` is a weekly test suite that validates the complete pipeline:

| Job | Validates |
|-----|-----------|
| `reset` | Manifest state resets cleanly |
| `commit-and-build` | Orchestrate triggers, matrix both legs run, staging state populated, release created as prerelease |
| `promote-staging` | Promote runs, prod state populated, release published (RC tag deleted) |
| `dispatch-inputs-check` | Manual dispatch inputs (reason, force) are passed through |
| `pr-preview-check` | PR preview comments work (non-blocking) |

The test suite runs every Monday at 06:00 UTC and can be triggered manually from the Actions tab.

## Documentation

For detailed architecture and workflow design:
- [stablekernel/cascade](https://github.com/stablekernel/cascade) - Main project repository
- `.github/manifest.yaml` - Pipeline configuration
- `.github/workflows/` - Auto-generated and custom workflow definitions

## License

This scaffold is part of the Cascade project and follows the same license.
