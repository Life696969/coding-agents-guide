# Track 6 — Editing video with an agent

**Prerequisites:** `ffmpeg` on PATH, and one video file of their own (30 seconds is
plenty). Check with `ffmpeg -version` **before starting the track.**
**Time:** about 40 minutes.
**They end up with:** a repeatable edit — trimmed, captioned, correctly encoded — that
the agent produces from a spec rather than by guessing, and that they can re-run
after changing one number.

---

## What they must understand by the end

**The agent cannot see or hear your video. It can only measure it.**

This is the whole track. Everything that goes wrong in agent-driven editing comes
from the agent guessing at something it could have measured: where the speech
starts, how long the clip is, whether the frame count is a whole number, whether the
caption is off the bottom of the screen.

The workflow that works is always: **measure → decide → render → verify.**

---

## Beat 1 — measure before anything

Have them run this on their own clip:

```bash
ffprobe -v error -show_entries stream=codec_name,width,height,r_frame_rate,nb_frames \
  -show_entries format=duration -of default=nw=1 input.mp4
```

Have them read it out: dimensions, frame rate, duration.

**TRAP — the one that ruins phone footage:** phone video is often recorded landscape
with a rotation flag. `ffprobe` reports `1920x1080` while the picture is portrait.
Check for it:

```bash
ffprobe -v error -select_streams v:0 -show_entries stream_side_data=rotation \
  -of default=nw=1 input.mp4
```

If that prints `rotation=-90`, the real frame is the other way round. Modern ffmpeg
applies it automatically on decode, but any arithmetic they do on width and height
must use the rotated values or every crop will be wrong.

---

## Beat 2 — find the edit points by measurement

Nobody should be typing timestamps they eyeballed. Have them find where the speech
actually starts:

```bash
ffmpeg -hide_banner -i input.mp4 -af "silencedetect=noise=-35dB:d=0.25" -f null - 2>&1 \
  | grep silence_
```

That prints every silence boundary. The first `silence_end` is roughly where speech
begins.

Have them compare it to where they *thought* it started. It is usually 0.3–0.8s off,
and that gap is exactly the dead air that makes an edit feel slow.

**CHECK:** have them state their trim points as numbers with a reason attached —
"start 0.72 because speech begins at 0.80 and I want 80ms of head" — not "start at
about a second".

---

## Beat 3 — cut on frame boundaries

Have the agent make the trim:

```bash
ffmpeg -y -i input.mp4 -ss 0.72 -t 6.40 -c:v libx264 -crf 18 -preset medium \
  -c:a aac -b:a 192k -movflags +faststart out.mp4
```

Then the part people skip — **check what actually came out**:

```bash
ffprobe -v error -select_streams v:0 -count_frames \
  -show_entries stream=nb_read_frames -of csv=p=0 out.mp4
```

**TRAP:** a duration that is not a whole number of frames gives a partial frame at
the end, which shows up as a stutter when clips are joined. At 30fps, pick durations
that are multiples of 1/30. If it might also be played at 60, use multiples of 1/30
and the count works at both. `6.40 x 30 = 192` — whole. `6.35 x 30 = 190.5` — not.

Have them deliberately try a bad duration and look at the frame count. Feeling this
once is worth more than being told.

---

## Beat 4 — captions without wrecking the picture

The common way to make captions readable is a dark band behind them. It is also the
fastest way to make good footage look badly lit, because the darkened area sits
right next to the undarkened area and the seam is obvious on skin and on bright
walls.

Do it with the type instead — a heavy stroke and a tight shadow:

```bash
ffmpeg -y -i out.mp4 -vf "drawtext=\
text='what he actually said':\
fontfile='C\:/Windows/Fonts/arialbd.ttf':\
fontsize=54:fontcolor=white:borderw=6:bordercolor=black@0.9:\
x=(w-text_w)/2:y=h*0.72" \
-c:a copy captioned.mp4
```

**TRAP — Windows paths in filter graphs:** the filter syntax uses `:` as a separator,
so `C:/Windows/...` breaks it. Escape the drive colon as `C\:/Windows/...` exactly as
above. This costs people an hour.

**CHECK:** have them pause on a frame, and check the picture *outside* the text is
as bright as the source. If the whole lower third got darker, they used a box; go
back to stroke and shadow.

---

## Beat 5 — verify, do not trust

Have them add a verification step and treat it as part of the edit, not an extra:

```bash
# frame count is what we asked for
ffprobe -v error -select_streams v:0 -count_frames \
  -show_entries stream=nb_read_frames -of csv=p=0 captioned.mp4

# audio has not drifted from the picture
ffprobe -v error -show_entries stream=duration -of csv=p=0 captioned.mp4

# loudness is sane for social (-14 LUFS is the usual target)
ffmpeg -hide_banner -i captioned.mp4 -af loudnorm=print_format=summary -f null - 2>&1 \
  | grep -E "Input (Integrated|True Peak)"
```

**This is the beat that separates people who ship from people who re-render all
night.** An agent will happily report success on a file with 190 frames when you
asked for 192. Only a check catches it.

---

## The project — an edit script that re-runs

> Write a script that takes a video and produces a finished vertical clip: measured
> trim points, a whole number of frames, captions with no dark band, audio
> normalised for social, and a printed verification report at the end.
>
> Then change one number — the trim point — and re-run it.

Requirements:

- Every timestamp is **derived from a measurement**, not typed by hand.
- The script prints what it measured before it renders.
- It fails loudly if the frame count is not what was requested.
- Re-running with a different trim needs one edit in one place.

**CHECK:** have them run it twice with different trims and confirm both outputs are
frame-exact. Then have them break it deliberately — request a duration that is not a
whole frame count — and confirm the script *complains* instead of quietly shipping.

---

## Beat 6 — what this does not do

- **The agent still cannot judge taste.** It can cut where speech starts; it cannot
  tell you the take is flat. Watch the output.
- **Re-encoding costs quality.** Every pass through x264 loses a little. Do the whole
  edit in one filter chain rather than five sequential renders, and keep the master.
- **Platforms re-encode again.** Ship a true peak below about −1 dBTP or loud
  consonants crunch after upload.
- **This scales to a pipeline.** The same measure → decide → render → verify shape is
  what production video tooling does; you have just built the small version.

**Predict-then-run:** have them change the caption `y` from `h*0.72` to `h*0.95`,
predict where it lands, then render and look. On most phones that is under the UI.

---

## What next

- **Build your own skill** (track 4) — make this whole edit one command
- **Deploying subagents** (track 1) — measure, render and verify as parallel agents
