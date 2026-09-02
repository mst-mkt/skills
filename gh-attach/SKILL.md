---
name: gh-attach
description: Attach local images or videos to GitHub issues, pull requests, and comments using the `--attach` flag (gh CLI v2.99.0+). Use when the user wants to embed a screenshot, image, or video file in an issue, PR, or comment, or when a workflow needs to upload a local media file to GitHub.
allowed-tools:
  - Bash(gh --version:*)
  - Bash(gh issue:*)
  - Bash(gh pr:*)
---

# gh attach

Attach local image and video files to GitHub issues, pull requests, and comments using the `--attach` flag added in gh CLI v2.99.0.

Installed version: !`gh --version | head -1`

## Version Check

Compare the installed version above with 2.99.0.

- If 2.99.0 or later, proceed with the `--attach` flag.
- If older, inform the user that `--attach` requires gh CLI 2.99.0+ and suggest upgrading via the missing-tools skill or attaching the file manually through the GitHub web UI.

## Usage

The `--attach` flag is available on `gh issue create`, `gh issue edit`, `gh issue comment`, `gh pr create`, `gh pr edit`, and `gh pr comment`.

```sh
gh issue create --attach './screenshot.png#Caption text'
gh issue edit 123 --attach ./image.png
gh issue comment 123 --attach ./repro.png

gh pr create --attach ./before.png
gh pr edit 456 --attach ./after.png
gh pr comment 456 --attach ./result.mp4
```

Repeat the flag to attach multiple files in a single invocation. If the body text already references the local path, gh replaces it with the uploaded URL. Otherwise it appends the attachment.

## Notes

- Supported image formats: png, jpg, jpeg, gif, webp, svg (max 10 MB). Supported video formats: mp4, mov, webm (max 100 MB).
- The value format is `<path>` or `<path>#<caption>`. Captions are supported for images only, not for videos.
- Available on GitHub.com and GitHub Enterprise Cloud. Not supported on GitHub Enterprise Server.
