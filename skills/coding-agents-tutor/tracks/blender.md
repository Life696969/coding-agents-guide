# Track 7 — 3D with Blender from an agent

**Prerequisites:** Blender installed and callable from the command line. Check with
`blender --version` **before starting the track** — if it is missing, offer to help
install it or to switch tracks. On Windows it is usually not on PATH by default and
lives at `C:\Program Files\Blender Foundation\Blender <version>\blender.exe`.
**Time:** about 40 minutes.
**They end up with:** a scene generated entirely by script and rendered headless, that
they can change by editing one number and re-running.

---

## What they must understand by the end

**Blender is a Python program with a GUI attached.** Almost everything the interface
does, `bpy` can do without it — which means an agent can build, light, animate and
render a scene without ever seeing one.

That makes 3D one of the best things to hand an agent, and also one of the easiest to
get quietly wrong: the agent cannot look at the render. It can only check that the
file exists. **You are the eyes.** Every beat here ends in them opening the image.

---

## Beat 1 — headless, first

Before any scene work, prove the loop runs.

`hello.py`:

```python
import bpy

# A fresh file still contains the default cube, camera and light.
bpy.ops.wm.read_factory_settings(use_empty=True)
print("Blender is running the script. Objects:", len(bpy.data.objects))
```

```bash
blender --background --python hello.py
```

**CHECK:** they should see their print line in the terminal. If Blender opens a
window, `--background` is missing entirely — its position on the command line does not
matter, because Blender parses arguments in passes and handles `--background` in an
earlier pass than `--python` regardless of order.

**TRAP:** order *does* matter, just not for that. `blender --background --python x.py
scene.blend` loads the .blend **after** the script runs, wiping everything the script
built — the `.blend` goes first: `blender --background scene.blend --python x.py`.
Short flags also cannot be combined: `blender -ba file.blend` errors out.

---

## Beat 2 — the four things every scene needs

An empty scene renders black, and this is where most first attempts land. A render
needs: **geometry, a light, a camera pointed at the geometry, and an output path.**

```python
import bpy
import math

bpy.ops.wm.read_factory_settings(use_empty=True)

# geometry
bpy.ops.mesh.primitive_uv_sphere_add(radius=1.0, location=(0, 0, 1))
sphere = bpy.context.active_object
bpy.ops.object.shade_smooth()

# ground
bpy.ops.mesh.primitive_plane_add(size=20, location=(0, 0, 0))

# light
bpy.ops.object.light_add(type="AREA", location=(4, -4, 6))
light = bpy.context.active_object
light.data.energy = 800
light.data.size = 5

# camera, aimed with a track-to constraint so we never do the maths
bpy.ops.object.camera_add(location=(6, -6, 4))
cam = bpy.context.active_object
bpy.context.scene.camera = cam
track = cam.constraints.new(type="TRACK_TO")
track.target = sphere
track.track_axis = "TRACK_NEGATIVE_Z"
track.up_axis = "UP_Y"

# output
scene = bpy.context.scene
scene.render.engine = "CYCLES"
scene.cycles.samples = 64
scene.render.resolution_x = 960
scene.render.resolution_y = 960
scene.render.filepath = "//render.png"

bpy.ops.render.render(write_still=True)
```

```bash
blender --background --python scene.py
```

**CHECK — they must open the PNG.** Not confirm it exists. Open it. The agent cannot
do this step and it is the only one that matters.

**TRAP:** `//` at the start of a Blender path means "relative to the .blend file". In
a background script with no saved file it resolves to the working directory, which
is usually what you want — but if the render vanishes, that is where it went.

---

## Beat 3 — the aiming problem

Point out what Beat 2 quietly avoided: nobody computed a camera rotation.

Positioning a camera by Euler angles is where agent-written Blender scripts fall
apart — the agent has no spatial feedback, so it guesses, renders black, guesses
again. A `TRACK_TO` constraint removes the problem: say what to look at, move the
camera anywhere, it stays aimed.

Have them change only the camera `location` — to `(0, -12, 2)`, then `(10, 0, 10)` —
and re-render each time. The subject stays framed.

> **The general lesson, and it is the point of the track:** when an agent cannot see
> the result, prefer constructs that are *correct by construction* over ones that
> need visual verification. Constraints over computed angles.

---

## Beat 4 — parameters, not edits

Have them lift the numbers to the top:

```python
PARAMS = {
    "subject": "sphere",     # sphere | cube | torus
    "cam": (6, -6, 4),
    "light_energy": 800,
    "samples": 64,
    "res": 960,
    "out": "//render.png",
}
```

Now a variation is a value change, not a rewrite — and an agent can sweep it.

Have them render three versions at `light_energy` 200, 800 and 3000, then look at
all three. Feeling how far off a guessed light value can be is the point.

**CHECK:** the script must run unchanged for all three. If they edited anything but
`PARAMS`, they have not finished the refactor.

---

## Beat 5 — render time is real

Cycles is a path tracer and cost scales with samples and resolution.

Have them time it:

```bash
# 64 samples — bash / macOS / Linux
time blender --background --python scene.py
```

```powershell
# PowerShell. Out-Default matters: Measure-Command swallows the command's
# output otherwise, and you would lose every print the script makes.
Measure-Command { blender --background --python scene.py | Out-Default }
```

Then 512 samples. It will be slower — but well short of eight times, and how far short
depends entirely on the scene. Cycles has **adaptive sampling on by default**, so
`samples` is a *ceiling*, not a count: easy pixels converge and stop early. Two other
things compress the ratio — denoising is on by default, and a fixed cost for process
startup, Python init and the file write sits inside both numbers regardless of
samples. Measuring it for their own scene is the point; putting a number on it in
advance is the habit this beat exists to break, so do not give them one.

**The working rule:** iterate at 32–64 samples and small resolution, raise it only
for the final. An agent left to its own devices will happily queue a 1024-sample 4K
render for a composition test and burn twenty minutes proving the light is in the
wrong place.

**TRAP:** EEVEE renders in a fraction of the time and is right for previews, but it is
a rasteriser: it handles glass, caustics and indirect light differently, so a preview
can look wrong in ways the final will not, and vice versa. Use it to check
*composition*, not lighting.

The engine string is version-dependent and it has never been plain `"EEVEE"`:

| Blender | `scene.render.engine` |
|---|---|
| 2.80 – 4.1 | `"BLENDER_EEVEE"` |
| 4.2 – 4.5 | `"BLENDER_EEVEE_NEXT"` |
| 5.0+ | `"BLENDER_EEVEE"` again |

So the two maintained LTS lines disagree: 5.2 LTS wants `"BLENDER_EEVEE"`, 4.5 LTS
wants `"BLENDER_EEVEE_NEXT"`, and the wrong one is an enum `TypeError`, not a silent
fallback. Check `bpy.app.version` rather than hard-coding it.

---

## The project — a parameterised scene generator

> Write a script that generates a small scene from a parameter block and renders it
> headless: a subject the parameters choose between, a ground plane, a light whose
> strength is a parameter, a constraint-aimed camera, and a chosen output path.
>
> Then have your agent sweep one parameter across four values and produce four
> renders in one run.

Requirements:

- Every number that matters is in `PARAMS`.
- The camera is aimed by constraint, never by hand-computed rotation.
- The script prints, before rendering, what it is about to render and where the file
  will land.
- The sweep writes four differently named files, not four overwrites of one.

**CHECK:** open all four. Then have them ask the agent *which one looks best* — and
notice it cannot answer, because it never saw them. That is not a failure of the
agent; it is the boundary of the technique, and knowing exactly where that boundary
sits is what this track is for.

---

## Beat 6 — what this is genuinely good for

- **Variation at volume.** Forty product renders on forty backgrounds is a loop.
  This is the real win.
- **Data-driven scenes.** Geometry from a CSV, a chart in 3D, a floor plan from
  measurements.
- **Repeatability.** A scene in a script is diffable and re-renderable a year later.
  A `.blend` you nudged by hand is not.

And what it is not: **taste**. The agent will not tell them the composition is dull.
Sweep the parameters, then use your own eyes.

**Predict-then-run:** have them remove the light entirely, predict the render, then
run it. Black, not dark — and it is worth seeing once, because "no light" and "bad
light" look nothing alike and agents confuse them constantly.

---

## What next

- **Build your own skill** (track 4) — turn the generator into one command
- **Editing video with an agent** (track 6) — same measure/verify discipline, moving pictures
