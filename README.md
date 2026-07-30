# Release Note Action

[![GitHub Action](https://img.shields.io/badge/GitHub_Action-purple?logo=github&logoColor=white)](https://github.com/purpleclay/gpg-import-action)
[![MIT](https://img.shields.io/badge/MIT-gray?logo=github&logoColor=white)](LICENSE)

A GitHub Action for [release-note](https://github.com/purpleclay/release-note): generates a release note for your project from conventional commits, with contributor attribution.

## Usage

Generate and publish the generated release note however you choose:

```yaml
- uses: actions/checkout@v7
  with:
    # release-note requires the full commit history to generate the note.
    fetch-depth: 0

- uses: purpleclay/release-note-action@v0
  id: release_note

# Requires `permissions: contents: write` on this job — the default
# github.token is read-only unless the job/workflow grants it.
- run: gh release create "${GITHUB_REF_NAME}" --notes-file "${NOTES_FILE}"
  env:
    GH_TOKEN: ${{ github.token }}
    NOTES_FILE: ${{ steps.release_note.outputs.release-note-path }}
```

> [!NOTE]
> Examples in this README use convenience major-version tags (e.g. `@v0`, `@v7`) for readability. For production workflows, prefer pinning to a full commit SHA.

## Inputs

See [action.yml](action.yml)

```yaml
- uses: purpleclay/release-note-action@v0
  with:
    # Directory containing the git repository to generate a release note for.
    # Optional. Default is github.workspace
    working-directory:

    # Version of release-note to download.
    # Optional. Defaults to the version this action release was tested
    # against, so pinning the action pins a known-good pairing.
    version:

    # Token for the release download and GitHub API requests (contributor
    # attribution, avatars).
    # Optional. Default is github.token
    token:

    # Verify the downloaded release-note checksum and attestation before
    # execution. Requires release-note >= 0.10.1.
    # Optional. Default is true
    verify-attestation:

    # Path the generated release note is written to.
    # Optional. Defaults to a file under the runner's temp directory —
    # outside the workspace, so the working tree stays clean; set explicitly
    # to write in-workspace (e.g. a committed CHANGELOG).
    output:
```

## Outputs

| Name                   | Description                                                                     |
| ---------------------- | ------------------------------------------------------------------------------- |
| `release-note-path`    | Path to the generated release note, ready for `gh release create --notes-file`. |
| `release-note-version` | The release-note version that was executed.                                     |

> [!IMPORTANT]
> The note's content is exposed only as a file, never as a step output. Release notes embed commit messages, which can be attacker-influenced in any repo accepting outside contributions — interpolating that into `${{ }}` in a shell step is a classic Actions script-injection vector. Always consume the path.
