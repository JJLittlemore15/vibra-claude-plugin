---
name: setup
description: First-time setup for the Vibra plugin - sign in to Vibra, check the Vibra tools respond, and check ffmpeg is installed. Use when the user installs the plugin, asks how to connect or sign in to Vibra, or the Vibra tools are not working.
allowed-tools: mcp__plugin_vibra_vibra__list_my_labels mcp__plugin_vibra_vibra__reel_playbook Bash(command -v ffmpeg) Bash(command -v ffprobe) Bash(ffmpeg -version) Bash(ffprobe -version) Bash(command -v brew) Bash(uname)
---

# Set up Vibra in Claude Code

Walk the user through these checks in order, briefly. Report each as done / needs action.

## 1. Sign in to Vibra

The plugin adds a remote MCP server called `vibra` (shown in `/mcp` as `plugin:vibra:vibra`).
Try calling `list_my_labels` (full name `mcp__plugin_vibra_vibra__list_my_labels`).

- **Works** - the user is signed in. Say how many labels they have (names are fine to show).
- **Tool not available, or an auth / "needs authentication" error** - tell the user:
  1. Type `/mcp` and press Enter.
  2. Choose **plugin:vibra:vibra** and pick **Authenticate**.
  3. A browser page from Vibra opens. Tap **Continue with Apple** (an account made with Sign in
     with Apple, including Hide My Email), or enter the email you use in the Vibra app and the 6-digit
     code Vibra emails. Then press **Allow**.
  4. Come back to Claude Code and say "done".
  Then call `list_my_labels` again to confirm. If the server shows as failed rather than needing
  authentication, suggest `/reload-plugins` or restarting Claude Code, then trying again.
- **Signed in but no labels** - that is fine; explain that labelling clips in the Vibra app (on
  the phone) makes them much easier to find here.

Access is read-only: Claude can see the user's own clips and labels and download copies to this
computer; it cannot post, delete or change anything in Vibra. The user can sign out any time from
`/mcp`.

## 2. Optional: playbook

If `reel_playbook` exists, call it once to confirm it responds (do not dump its contents). If it
does not exist yet, skip silently.

## 3. ffmpeg

Reels are cut locally with ffmpeg. Run `ffmpeg -version` and `ffprobe -version` (not just
`command -v` - a broken install can be found on the PATH and still fail to start).

- **Both run** - done.
- **Not found** - suggest an install and **ask before running anything**:
  - macOS: `brew install ffmpeg` (if `command -v brew` finds nothing, point to https://brew.sh first)
  - Linux: the distro package, e.g. `sudo apt install ffmpeg`
  - Windows: `winget install ffmpeg`
- **Found but fails to start** (e.g. a "Library not loaded" error on macOS) - suggest
  `brew update && brew upgrade ffmpeg` (or `brew reinstall ffmpeg`), again only with the user's yes.

## 4. Finish

Summarise what is ready and suggest a first ask, for example:
- "Make me a 20-second reel of my best points this week" (`/vibra:make-reel`)
- "Brainstorm reels from what's trending on Vibra Radar" (`/vibra:brainstorm-reels`)
