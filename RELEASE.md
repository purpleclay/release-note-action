# Releasing

This is a composite GitHub Action, so there's nothing to build. Consumers use the source directly at a tagged git ref, and releasing is: tag, verify, publish.

- [Versioning](#versioning)
- [Cutting a release](#cutting-a-release)
- [Moving the major version tag](#moving-the-major-version-tag)
- [Fixing a bad release](#fixing-a-bad-release)

## Versioning

Tags are `vX.Y.Z`. Consumers pin either a full commit SHA or the floating major tag (e.g. `v0`), which is the convention used throughout this action's README (`purpleclay/release-note-action@v0`). Semver tracks the caller interface:

- Major: renamed or removed inputs or outputs, or any increase in the permissions callers must grant.
- Minor: new inputs with defaults, new outputs.
- Patch: dependency bumps (including the default release-note version), internal hardening, documentation.

## Cutting a release

Tagging is a manual step today. This repo doesn't yet use [`nsv`](https://github.com/purpleclay/nsv) to automate it, though that may change.

1. Confirm `main` is green (ci) and every merged but unreleased change is accounted for:

   ```sh
   last_tag=$(git describe --tags --abbrev=0 --match 'v*.*.*' 2>/dev/null || true)
   git log "${last_tag:+$last_tag..}main" --oneline
   ```

2. Create a signed, annotated tag locally and push it:

   ```sh
   git tag -s vX.Y.Z -m "chore: release for vX.Y.Z"
   git push origin vX.Y.Z
   ```

   The tag must be signed by an authorized signer (`purpleclay`). The `release` workflow verifies this before publishing anything.

3. The `release` workflow triggers on the tag push and waits for approval in the `release` environment. It then validates the tag format, verifies the signature, and runs this action from the tagged commit (`uses: $/`) to generate the release note and publish the GitHub release. No build assets.

4. [Move the major version tag](#moving-the-major-version-tag).

5. Verify that the release exists, the notes render correctly, and `purpleclay/release-note-action@vX` resolves to the new commit.

If the action is broken at the tagged commit, the release fails at the notes step and nothing is published. Fix it on `main` and tag a new patch release.

## Moving the major version tag

The floating major tag (`vX`) isn't moved automatically. Move it straight after the release above publishes:

```sh
git tag -sf vX -m "chore: release for vX.Y.Z"
git push origin vX --force
```

## Fixing a bad release

Releases are immutable, so a bad release is never edited or deleted. Follow it with a fixed patch release and leave the bad tag in place. Once the patch is out, move the major tag on to it as above.
