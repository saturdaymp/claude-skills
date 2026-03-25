---
name: smp-generate-gh-changelog-script
description: This skill should be used when the user wants to generate or update a CHANGELOG.md file from GitHub releases. Common triggers include "generate changelog", "create changelog", "update changelog from releases", "build changelog", or "add changelog script". It creates a standalone bash script that fetches GitHub releases and generates a formatted CHANGELOG.md, so changelog generation works without Claude.
argument-hint: "[output-path]"
---

# Generate Changelog Script

Create a standalone bash script that generates a CHANGELOG.md from GitHub releases.

## Input

The user optionally provided an output path for the changelog: $ARGUMENTS

If no output path was provided, the script will default to `CHANGELOG.md` in the repository root.

## Phase 1 - Check Prerequisites

1. Confirm we are inside a git repository:
   ```
   git rev-parse --show-toplevel
   ```

2. Confirm `gh` CLI is available:
   ```
   gh --version
   ```

3. If either check fails, report the error and stop.

## Phase 2 - Create the Script

1. Create a `scripts/` directory in the repository root if it doesn't already exist.

2. If `scripts/generate-changelog.sh` already exists, warn the user and ask for confirmation before overwriting.

3. Write the following script to `scripts/generate-changelog.sh`:

```bash
#!/usr/bin/env bash
#
# generate-changelog.sh
#
# Generates a CHANGELOG.md from GitHub releases using the GitHub CLI (gh).
# The release body content is preserved as-is from each GitHub release.
#
# Usage: ./scripts/generate-changelog.sh [output-path]
#
# Arguments:
#   output-path  Path to the output file (default: CHANGELOG.md in the repo root)
#
# Requirements:
#   - git
#   - GitHub CLI (gh) installed and authenticated
#

set -euo pipefail

# --- Configuration ---
OUTPUT_FILE="${1:-CHANGELOG.md}"

# --- Preflight checks ---
if ! command -v gh &> /dev/null; then
  echo "Error: GitHub CLI (gh) is not installed. Install it from https://cli.github.com/" >&2
  exit 1
fi

if ! gh auth status &> /dev/null; then
  echo "Error: GitHub CLI is not authenticated. Run 'gh auth login' first." >&2
  exit 1
fi

if ! git rev-parse --is-inside-work-tree &> /dev/null; then
  echo "Error: Not inside a git repository." >&2
  exit 1
fi

# --- Detect repository ---
REPO=$(gh repo view --json nameWithOwner --jq '.nameWithOwner')
if [[ -z "$REPO" ]]; then
  echo "Error: Could not determine GitHub repository." >&2
  exit 1
fi

# --- Fetch releases ---
RELEASES=$(gh release list --repo "$REPO" --limit 1000 --json tagName,publishedAt,isDraft,isPrerelease --jq '[.[] | select(.isDraft == false)] | sort_by(.publishedAt) | reverse')

RELEASE_COUNT=$(echo "$RELEASES" | jq 'length')
if [[ "$RELEASE_COUNT" -eq 0 ]]; then
  echo "No published releases found for $REPO."
  exit 0
fi

echo "Found $RELEASE_COUNT release(s) for $REPO"

# --- Build changelog ---
{
  echo "# Changelog"
  echo ""
  echo "All notable changes to this project will be documented in this file."
  echo "This file is auto-generated from [GitHub releases](https://github.com/$REPO/releases)."
  echo ""

  # Collect link references
  LINK_REFS=""

  for i in $(seq 0 $((RELEASE_COUNT - 1))); do
    TAG=$(echo "$RELEASES" | jq -r ".[$i].tagName")
    IS_PRERELEASE=$(echo "$RELEASES" | jq -r ".[$i].isPrerelease")

    # Fetch full release details
    RELEASE_DETAIL=$(gh release view "$TAG" --repo "$REPO" --json tagName,name,body,publishedAt,isPrerelease)

    NAME=$(echo "$RELEASE_DETAIL" | jq -r '.name // empty')
    BODY=$(echo "$RELEASE_DETAIL" | jq -r '.body // empty')
    PUBLISHED=$(echo "$RELEASE_DETAIL" | jq -r '.publishedAt')

    # Format the date as YYYY-MM-DD
    DATE=$(date -jf "%Y-%m-%dT%H:%M:%SZ" "$PUBLISHED" "+%Y-%m-%d" 2>/dev/null || echo "$PUBLISHED" | cut -c1-10)

    # Strip leading 'v' for the display version
    VERSION="${TAG#v}"

    # Build the heading
    HEADING="## [$VERSION] - $DATE"
    if [[ "$IS_PRERELEASE" == "true" ]]; then
      HEADING="$HEADING (Pre-release)"
    fi

    echo "$HEADING"
    echo ""

    if [[ -n "$BODY" ]]; then
      echo "$BODY"
      echo ""
    fi

    # Collect link reference
    LINK_REFS="${LINK_REFS}[$VERSION]: https://github.com/$REPO/releases/tag/$TAG
"
  done

  # Write link references
  echo "$LINK_REFS"
} > "$OUTPUT_FILE"

echo "Changelog written to $OUTPUT_FILE"
```

4. Make the script executable:
   ```
   chmod +x scripts/generate-changelog.sh
   ```

5. Show the user the created script and confirm it was written successfully.

## Phase 3 - Run the Script (Optional)

1. Ask the user: "Would you like me to run the script now to generate the initial CHANGELOG.md?"

2. If yes, run:
   ```
   ./scripts/generate-changelog.sh
   ```

3. Show the user the generated CHANGELOG.md content for review.

## Phase 4 - Update README (Optional)

1. Ask the user: "Would you like me to update the README with documentation on how to use the changelog script?"

2. If yes, add a section to the README. Suggested content:

   ```markdown
   ### smp-generate-changelog

   Creates a standalone bash script (`scripts/generate-changelog.sh`) that generates a `CHANGELOG.md` from GitHub releases. The script can be run independently without Claude.

   **Usage:**

   ```
   /smp-generate-changelog
   ```

   Once the script is created, you can run it directly:

   ```bash
   ./scripts/generate-changelog.sh              # Writes to CHANGELOG.md
   ./scripts/generate-changelog.sh docs/CHANGES.md  # Custom output path
   ```

   **Script requirements:** [GitHub CLI (`gh`)](https://cli.github.com/) installed and authenticated.
   ```

3. **STOP and show the draft README changes to the user for confirmation before writing.**

## Phase 5 - Commit (Optional)

1. Ask the user if they would like to commit the new files.

2. If yes:
   - Stage the relevant files (`scripts/generate-changelog.sh`, and optionally `CHANGELOG.md`, `README.md`)
   - Draft a commit message
   - **STOP and show the draft commit message. Wait for user confirmation before committing.**
   - Commit with the confirmed message.

## Error Handling

- If `gh` is not installed or not authenticated, report the error clearly and stop.
- If the repository has no GitHub remote, report and stop.
- If `scripts/generate-changelog.sh` already exists, warn and ask before overwriting.
- Never retry failed commands silently. Always report errors and ask the user for guidance.

## Important Notes

- At every **STOP** point, wait for the user to respond before continuing. Do not skip confirmation steps.
- Use `gh` for all GitHub API interactions, never raw HTTP requests.
- The generated script works with any GitHub repository, not just this one.
