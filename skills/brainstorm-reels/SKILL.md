---
name: brainstorm-reels
description: Propose five concrete padel reel ideas built from the user's own Vibra Clips and what is trending on Vibra Radar. Use when the user asks for reel ideas, what to post, content ideas, or to brainstorm from Vibra Radar or their clips.
argument-hint: "[optional focus, e.g. this week's matches or funny moments]"
---

# Brainstorm reels from Vibra Clips and Vibra Radar

The Vibra tools come from this plugin's `vibra` MCP server (full names
`mcp__plugin_vibra_vibra__<tool>`) and are read-only. If they are missing or say the user is not
signed in, tell the user to run `/mcp`, choose `plugin:vibra:vibra` and sign in (or run
`/vibra:setup`), then stop.

Focus from the user (may be empty): $ARGUMENTS

## Steps

1. Call `reel_playbook` if the tool exists and follow it over anything here. If it is missing or
   errors, carry on without mentioning it.
2. Call `top_radar_reels`. Read *why* each reel scored and its `craft` breakdown (hook, structure,
   shot, edit style, length, audio). Look for patterns that repeat across the top reels.
3. Call `list_my_labels`, then `find_my_clips` (prefer labelled, unused clips; narrow by the
   user's focus or recent dates if given). Note each clip's `item_id`, labels, notes, trim window
   length and match date/venue.
4. Propose **exactly five** ideas. Every idea must be makeable from clips the user actually has -
   never invent clips. For each idea give:
   - **Title** - a few words
   - **Clips** - the specific items: label, match date (and venue if useful), `item_id`, seconds
   - **Hook** - what happens in the first 1-2 seconds
   - **Structure** - the order of clips and the payoff
   - **Length** - target seconds (sum of the chosen trim windows, roughly)
   - **Caption idea** - one line for the post (and on-screen text, if any)
   - **Why it should work** - one line linking it to a Radar pattern or the playbook
   Keep each idea compact. Use Radar reels as inspiration for craft only; never copy them.
5. If the user has too few clips for five distinct ideas, say so and propose as many as their
   clips genuinely support, plus what to film or label next time.
6. Ask which idea to make (or what to change). When they choose, hand off to the `make-reel`
   skill (`/vibra:make-reel`) with the chosen clips, order, length and caption choice already
   filled in, so it does not ask again.

Never post or share anything.
