# Vibra for Claude Code

Make padel reels from your Vibra Clips with Claude: find your best moments, learn from what's
trending on Vibra Radar, and cut 9:16 reels ready for Instagram.

You film and mark highlights in the Vibra app on your phone. This plugin lets Claude Code on your
computer (Mac or Windows PC) find those clips, suggest reel ideas, and cut a vertical 1080x1920 reel you can post
yourself.

## What you get

| | |
|---|---|
| `/vibra:setup` | Sign in to Vibra, check everything works, check ffmpeg |
| `/vibra:make-reel` | Pick clips, download them, cut a 9:16 H.264/AAC mp4 |
| `/vibra:brainstorm-reels` | Five reel ideas built from your clips and what's trending on Vibra Radar |

You don't have to type the commands - just ask in plain words and Claude picks the right one.

## What you need

- A Mac or Windows PC. Making reels needs Claude Code or Codex running on your computer - the
  Claude and ChatGPT websites and phone apps can't download and cut video
- A Claude plan that includes Claude Code
- [Claude Code](https://code.claude.com) in a terminal, or the Code tab of the Claude desktop app
  (a local session on your computer - plugins don't run in cloud sessions)
- A Vibra account with some clips
- [ffmpeg](https://ffmpeg.org) - on a Mac `brew install ffmpeg`, on Windows `winget install ffmpeg`. `/vibra:setup` checks for it and
  asks before installing anything.

## Install

In Claude Code, run:

```
/plugin marketplace add JJLittlemore15/vibra-claude-plugin
/plugin install vibra@vibra
```

Or from a terminal:

```bash
claude plugin marketplace add JJLittlemore15/vibra-claude-plugin
claude plugin install vibra@vibra
```

Then run `/reload-plugins` (or restart Claude Code).

**Claude desktop app (Code tab):** run the two terminal commands above once, then open a local
session in the Code tab - the plugin appears there too, because the desktop app and the terminal
share the same settings. You can also manage it from the `+` button next to the prompt box,
under **Plugins**.

## Sign in

Run `/vibra:setup`, or do it by hand:

1. Type `/mcp`, choose **plugin:vibra:vibra**, then **Authenticate**.
2. A Vibra page opens in your browser. Tap **Continue with Apple** (or enter the email you use in
   the app, then the 6-digit code Vibra emails you), and press **Allow**.
3. Back in Claude Code you're connected. Sign out any time from `/mcp`.

## Using Codex

The Codex version lives in its own repo: [vibra-codex-plugin](https://github.com/JJLittlemore15/vibra-codex-plugin).

## Things to ask

- "Make me a 20-second reel of my best points this week"
- "Brainstorm reels from what's trending on Vibra Radar"
- "Cut a 15-second reel from my clips labelled *bandeja*, captions on"
- "Which of my clips haven't I used yet?"

Finished reels are saved under `./vibra-reels/<date>-<name>/` in the folder you ran Claude in.

## Privacy

- **Read-only.** Claude can see your own clips, labels and notes, and Vibra Radar's public top
  reels. It cannot post, delete or change anything in Vibra, and it never posts to Instagram -
  you post the reel yourself.
- **Your files stay on your computer.** Clips are downloaded through short-lived links (valid for
  10 minutes), cut locally with ffmpeg, and the downloaded originals are deleted afterwards unless
  you ask to keep them.
- Your sign-in is handled by Claude Code (OAuth); this plugin stores no keys or passwords.

## Troubleshooting

- **Vibra tools missing or "needs authentication"** - run `/mcp` and authenticate
  `plugin:vibra:vibra`.
- **Server shows as failed** - run `/reload-plugins` or restart Claude Code, then `/mcp` again.
- **ffmpeg "Library not loaded" on a Mac** - `brew update && brew upgrade ffmpeg`.
- **Update the plugin** - `claude plugin marketplace update vibra` then
  `claude plugin update vibra@vibra`.

## License

[MIT](LICENSE)
