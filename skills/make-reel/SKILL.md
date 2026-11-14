---
name: make-reel
description: Cut a 9:16 Instagram-ready padel reel from the user's Vibra Clips. Use when the user asks for a reel, edit, cut, highlight video, montage or short from their Vibra clips, labels or matches.
argument-hint: "[what you want, e.g. 20-second reel of my best points this week]"
---

# Make a reel from Vibra Clips

You are cutting a vertical (9:16, 1080x1920) reel on the user's own computer from clips they
recorded and marked in Vibra. The Vibra tools come from the `vibra` MCP server of this plugin
(full names `mcp__plugin_vibra_vibra__<tool>`). They are read-only: nothing here can change or
post anything in Vibra.

If no Vibra tool is available, or a tool says the user is not signed in, stop and tell the user
to run `/mcp`, choose `plugin:vibra:vibra` and sign in (or run `/vibra:setup`). Do not guess clips.

Request from the user (may be empty): $ARGUMENTS

## 1. Playbook first

Call `reel_playbook` if the tool exists. **Follow it over anything in this file** where they
differ (length, pacing, crop, captions, ordering, music, naming). If the tool is missing, errors
or returns nothing, say nothing about it and use the defaults below.

## 2. Ask only what is missing

From the request and the playbook, work out:
- **Length** (default 15-30 seconds)
- **Vibe** (e.g. best rallies, funny moments, one player's shots, a single match)
- **Which clips** - specific labels, a date range, a match, or "you choose"
- **Captions** on or off (default off)

Ask one short question covering only the gaps. If the user said "you choose", choose.

## 3. Find the clips

1. `list_my_labels` to see the user's label names (use their exact spelling in filters).
2. `find_my_clips` with labels / used-unused / dates as fits. Prefer **labelled** and **unused**
   clips unless the user asked otherwise. Each item has an `item_id`, a `trim_start`/`trim_end`
   window, labels, notes and the match date/venue.
3. Optional: `top_radar_reels` when a hook or pacing model would help. Use its `craft` breakdown
   (hook, structure, shot, edit style, length, audio) as inspiration for ordering and length only;
   never copy anyone's content.
4. Pick the clips and order (strongest moment first unless the playbook says otherwise). Fit the
   total of the trim windows to the target length; drop or shorten clips rather than speed up.
   Tell the user the plan in a few lines (clip, label, match date, seconds) before downloading.

## 4. Download straight away

Signed links from `get_clip_download` **expire after 10 minutes**, so fetch each one and download
it immediately - never collect links first and download later. If a download fails with 403/expired,
call `get_clip_download` again for that item.

```bash
WORK="./vibra-reels/$(date +%Y-%m-%d)-<short-slug>"   # e.g. 2026-11-13-best-points
mkdir -p "$WORK/src"
curl -fL --retry 2 -o "$WORK/src/01-<item_id>.mp4" "<signed_url>"
```

Use the trim window returned by `get_clip_download` for that item (it wins over the one from
`find_my_clips` if they differ). Never print the signed URL back to the user.


## Check it really shows the action

A label or a trim window is not proof - a clip labelled "smash" can start after the smash, so the reel
shows only the aftermath. **Unless the user says not to, check every clip with your own eyes:**

- **Before cutting:** pull frames across the whole downloaded file (one every 0.25-0.5 s, e.g.
  `ffmpeg -i in.mp4 -vf fps=3,scale=480:-1 frames/%03d.jpg`) and look at them. Find the moment the
  named shot is actually played - the swing and the ball - and cut around it, treating
  `trim_start`/`trim_end` as a starting guess. If the shot isn't in the clip, don't use it: say which
  clip and pick another.
- **After cutting:** look at frames from each segment of the finished reel and confirm each one shows
  what you say it shows. Fix it before showing the user.
- Tell the user in one line per clip what you saw ("clip 2: smash at 3.4 s, kept 2.8-5.0 s"). Never
  claim a clip shows a shot you haven't seen in its frames.

## 5. Check ffmpeg

```bash
command -v ffmpeg && command -v ffprobe
```

If missing: on macOS suggest `brew install ffmpeg` (if Homebrew itself is missing, point to
https://brew.sh); on Linux the distro package (`sudo apt install ffmpeg` etc.); on Windows
`winget install ffmpeg`. **Ask before installing anything** and wait for a yes.

## 6. Inspect each source

```bash
ffprobe -v error -select_streams v:0 \
  -show_entries stream=width,height,r_frame_rate:stream_side_data=rotation:stream_tags=rotate \
  -show_entries format=duration -of json "$WORK/src/01-<item_id>.mp4"
ffprobe -v error -select_streams a -show_entries stream=index -of csv=p=0 "$WORK/src/01-<item_id>.mp4"
```

- A rotation of 90/-90/270 means the displayed frame is height x width. ffmpeg auto-rotates on
  decode, so do **not** add a `transpose`; just use the displayed size when reasoning about crop.
- Check the trim window fits inside `duration`. If `trim_end` is far beyond the duration, the
  values may be milliseconds - divide by 1000. If there is no window, use the whole clip.
- Note which clips have no audio stream (second command prints nothing).

## 7. Cut, crop and join in ONE encode

Build a single ffmpeg command: input-seek each clip to its window, normalise every clip to
1080x1920 / 30 fps / 48 kHz stereo, concatenate, optionally burn captions, and encode once.
Do not cut to intermediate files and re-encode them again.

Per clip *i* (window `S`..`E`, duration `D = E - S`):

```
-ss S -t D -i "$WORK/src/0i-....mp4"
```

Video chain per input (centre crop - keeps the middle of the court, where the action usually is):

```
[i:v]scale=1080:1920:force_original_aspect_ratio=increase,crop=1080:1920,setsar=1,fps=30,format=yuv420p[vi]
```

- If the action sits off-centre (landscape padel footage often does), shift the crop:
  `crop=1080:1920:x=(iw-1080)*0.35:y=0` (0 = far left, 0.5 = centre, 1 = far right). Ask the user
  or use clip notes/labels when it matters; default to centre.
- If the user prefers the whole court visible, use a blurred fill instead of a crop:
  `split[a][b];[a]scale=1080:1920:force_original_aspect_ratio=increase,crop=1080:1920,boxblur=20[bg];[b]scale=1080:-2[fg];[bg][fg]overlay=(W-w)/2:(H-h)/2,setsar=1,fps=30,format=yuv420p`

Audio chain per input:

```
[i:a]aformat=sample_rates=48000:channel_layouts=stereo,asetpts=PTS-STARTPTS[ai]
```

For a clip with no audio, add a silent input instead: `-f lavfi -t D -i anullsrc=r=48000:cl=stereo`
and use that input's audio in its place.

Join:

```
[v0][a0][v1][a1]...concat=n=N:v=1:a=1[v][a]
```

Optional captions (only if the user wants them): write each caption to a text file (avoids
quote-escaping) and chain `drawtext` on `[v]` with `enable='between(t,START,END)'` using the
cumulative times in the joined video:

```
[v]drawtext=textfile='$WORK/cap1.txt':fontsize=64:fontcolor=white:borderw=4:bordercolor=black:x=(w-text_w)/2:y=h*0.12:enable='between(t,0,2.5)'[vout]
```

Check `ffmpeg -hide_banner -filters | grep drawtext` first; if missing, skip captions and say why.
On macOS a font can be named with `fontfile=/System/Library/Fonts/Supplemental/Arial Bold.ttf`.

Encode:

```bash
ffmpeg -hide_banner -y <inputs> -filter_complex "<graph>" -map "[vout]" -map "[a]" \
  -c:v libx264 -preset medium -crf 20 -profile:v high -pix_fmt yuv420p \
  -c:a aac -b:a 160k -ar 48000 -movflags +faststart "$WORK/<slug>.mp4"
```

(Map `[v]` instead of `[vout]` when there are no captions.) For longer filter graphs, write the
graph to `$WORK/graph.txt` and pass `-filter_complex_script "$WORK/graph.txt"`.

## 8. Check the result

```bash
ffprobe -v error -show_entries stream=codec_name,width,height:format=duration -of default=nw=1 "$WORK/<slug>.mp4"
```

Expect h264 1080x1920, aac, and a duration close to the plan. Optionally grab a frame to look at:
`ffmpeg -v error -ss 1 -i "$WORK/<slug>.mp4" -frames:v 1 "$WORK/preview.jpg"` and view it to
confirm the crop keeps the players in shot; adjust and re-run if not.

## 9. Hand over

- Give the full path of the mp4 and a short summary: length, clips used (label + match date),
  crop choice, captions on/off.
- Suggest a caption/hashtags for the post if useful, but **never post, upload or share anything**.
  The user posts it themselves.
- Delete `$WORK/src/` (the downloaded source clips) unless the user asked to keep them. Keep the
  finished reel.
