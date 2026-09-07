# Known issues

Things found and deliberately **not** chased, so that finding one does not derail the job in hand.
An entry here is a promise to come back to it, not a shrug: each carries what was seen, what the
evidence says, and where to start — enough to pick up cold, months later, without rediscovering it.

Add to this rather than investigating, unless the thing is actually blocking the change being made.

---

## 1. Interface clicks are dropped when the capture is launched from a script

**Severity:** high — it is the reason a capture run cannot be trusted without reading its
`FAILURES` line, and the reason release screenshots sometimes come out showing an idle machine.

**What happens.** Scenes that click on an interface *panel* — a recipe tile, an inventory slot, the
stack splitter, an upgrade box — fail far more often when the player is started by
`tools/release-shots.sh` than when the identical command is typed at a terminal. Clicks at the
**world** (placing a building, the belt tool) land either way.

Observed in one full pass:

```
Factory: shift click opened the splitter = False
Factory: dragged 30 from the hold ... chose none, input = empty, status = NoRecipe
Factory: the smelter cast nothing - NoRecipe.
Factory: picked up empty of 20 held
Factory: gave 20 Iron Bar -- box 0, hand 0, hold 20
```

The same scenes, run straight from a terminal, pass: `shift click opened the splitter = True`,
`smelter output = 3x Iron Bar, status = Running`.

**It is the foreground. Proved.** The capture sets
`InputSystem.settings.backgroundBehavior = IgnoreFocus` and `Application.runInBackground = true`,
which is why *world* input keeps working unfocused — those reads go through `GameInput` straight to
the device. A **UI Toolkit runtime panel** is told no such thing: it takes pointer events through
the panel's own event system, which is a separate path and is not covered by that setting.

That was a hypothesis until the window was deliberately pushed to the back of the z-order
(`WindowPlacement.SendToBack`, on by default so a run does not cover the desktop). The same
`smelter` scene, same build, same seed, back to back:

```
window in front:  shift click opened the splitter = False   FAILURES = 1
window at back:   dragged 0 from the hold (0.00, 900.00)    FAILURES = 2
```

So the foreground is the variable. Note the second symptom as well as the count: with the window
behind, `worldBound.center` on a cargo slot comes back at the origin, so the panel is not even
laying its slots out — the clicks are not merely dropped, there is nothing at the coordinate to
click. Rendering is unaffected either way; the shots come out identically.

**What follows from that.** `-factoryFront` keeps the window forward and `tools/release-shots.sh`
passes it, because release shots are the one run that has to be trusted. Any check that does not
click on a panel should be run without it and stay out of the way.

**It is the event path, not the layout. Also proved.** The capture now logs what the interface
would hand a click at that coordinate to (`UIRoot.DescribePick`, called in the smelter scene). At
the moment a recipe click fails, the pick comes back as the recipe tile itself:

```
Factory: under the recipe tile = recipe-cell.recipe-cell--compact < station__grid <
         station__screen.machine__screen < machine__body < machine.ui-blocker < layer.scrim < ...
```

So the coordinate is right, the element is on top, and it is pickable. Nothing is covering it. The
injected pointer event simply is not acted on. That rules out every layout explanation and leaves
the panel's own event path, which is what the focus theory above says.

**It is currently failing every time, not occasionally.** As of v0.32.0 the `smelter` scene fails
its splitter and recipe clicks on four runs out of four, window in front, same seed and build —
where the same scene passed earlier in the same session. Whether that is the same intermittent
fault having a bad day or something newer sitting on top of it has not been established. What has
been ruled out: the wrapping cargo grid (v0.31.0) — reverting it to fixed rows changes nothing —
and the layout, per the pick above.

**Where to start.**
- `CaptureDirector.Run` — where focus behaviour is set up; whatever fixes this belongs beside it.
- Try forcing the window to the foreground at capture start, or check whether a
  `PanelSettings`/`EventSystem` option exists to make a runtime panel ignore focus too.
- Reproduce: `tools/release-shots.sh <version> smelter` a few times and watch the `FAILURES` line;
  then the same `-factoryScenes smelter` typed at a terminal.

**Workaround in the meantime.** Read the `FAILURES` line. If a scene failed, re-run that scene
straight from a terminal. Noted in the header of `tools/release-shots.sh`.

---

## 2. Editor-only assertions are caught by nothing automatic

**Severity:** medium — it shipped a broken main menu once already.

Unity compiles assertions into the editor that a built player does not have, and the capture pass
runs a **player**. Code can pass every scene and throw every frame the moment Play is pressed. The
one that bit: UI Toolkit refuses to create a `VisualElement` from a MonoBehaviour constructor or
field initializer, which is an editor-only check — the player ran the same code perfectly.

There is no automated cover for this. After touching interface code, either press Play or read the
tail of `%LOCALAPPDATA%/Unity/Editor/Editor.log`.

**Idea worth trying.** An editor batchmode method that enters play mode for a few seconds and
reports the console would close the gap. See the dead end below before attempting it.

---

## 3. Dead ends — do not spend time here again

- **`EditorApplication.EnterPlaymode()` driven from `-executeMethod` does not work**, in batchmode
  or with a real window. Batchmode never enters play mode at all; without it the editor runs the
  method and quits before play mode starts. Two attempts, both wasted.
- **Force-killing the editor leaves an `Assets/_Recovery/` folder** that must be deleted, and Unity
  will offer to recover from it on the next open.
- **`Debug.LogError` in a player log is the bare message** with no marker and no stack. Grepping a
  log for "error" finds nothing however badly the run went. This is closed — the capture counts
  failures off `Application.logMessageReceived` and prints `FAILURES = n` — but it is recorded here
  so nobody reintroduces a check that looks like one and is not.

---

## 4. Small cleanups, whenever the file is open anyway

- `UIRoot._podRecycled` is assigned and never read (CS0414). The `PodRecycled` event it listens to
  may want removing with it.
- `ControlsPanel` walks `Key.IMESelected`, which is obsolete (CS0618). It is skipped explicitly, so
  the warning is the only cost; the enum walk exists because `allKeys` throws on entries with no
  control behind them.
- `SaveGame.PowerOnline` and `BuildingSave.Rotated` are both vestigial, kept only so an older save
  still parses. Both can go at the next `SaveGame.CurrentFormat` bump.
- `docs/screenshots/v0.22.0` and earlier are full seventy-image sets from when a release carried the
  previous folder forward. They could be trimmed to the pictures their changelog entries actually
  reference; the entries are the only thing that links to them.
- `tools/release-shots.sh` hardcodes `-factorySeed 1337`. Right for changelog images, wrong the day
  a shot is wanted of a different planet.

---

## 5. Terrain does not get out of the way — answered a different way

The cutaway fades buildings standing between the camera and the character. It cannot do the same
for terrain, and the reason is not that terrain is hard to reach: it is that there is nothing
behind the ground. A faded cliff is a hole onto the sky.

So the ground stays and the character is drawn through it — `SeenThrough` hangs a second copy of
their silhouette on the body, drawn only where the depth buffer says something nearer got there
first. In the open it draws nothing. Behind a rock it draws a ghost of whoever is behind the rock,
and it answers every occluder there is rather than the ones somebody remembered to make fadeable.

Still open, and smaller than it was: a character standing in a hollow with the camera on the high
side is a ghost rather than a character, which reads as being underground rather than as being
behind something. If that turns out to matter, the answer is probably for the terrain shader to
dissolve the band of ground in front of them after all, accepting the hole, because at that angle
the hole is filled by the ground behind it.

---

## 6. The intro scene fails on how close the player gets to the recycled pod

`-factoryScenes intro` reports `FAILURES = 1`:

```
Factory: pod recycled, its ground still blocked = False, walked to within 0,22 of where it stood
Factory: the recycled pod left a collider behind.
```

The message is misleading. `blocked = False` says the collider *was* taken away properly; what trips the
assertion is the second half of it, `strayed > 0.2f` — the player walks to 0.22 of the wreck and the
check wants 0.20.

Deterministic at seed 1337, and **not** caused by the 3D work: it reproduces identically at `cf74106`
with the pod changes stashed. It was simply never run during that work, because the scene only
started producing shots again once `station` was found to be the one that frames the mining pass.

Worth someone deciding which of the two is wrong. Either the walk genuinely stops two hundredths of
a tile too early — in which case the interesting question is what stops it, since the collider is
gone — or the tolerance was written tighter than the walk it is measuring and wants to be 0.25.
Do not "fix" it by widening the tolerance without looking, because the first reading is a real bug
about movement and the second is a typo, and they are not the same thing.

---

## 7. A sprite hung under the placement ghost does not rasterise

**Severity:** low — it cost one feature that was dropped rather than shipped, and nothing that
exists today depends on it. It is here because whatever is going on is not obvious and the next
person to try the same thing will spend the same three builds.

**What was being built.** A mark on the ground under the placement ghost, for the storey aiming
added in 1.38.0: a foundation can be laid in mid air now, and a ghost three floors up is drawn in
very nearly the same place as a ghost a dozen tiles further north with nothing underneath either
of them to give the difference away.

**What happened.** A second `SpriteRenderer` was added to `WireframeVisual` beside the cell grid
and the hologram, carrying `ProceduralArt.WireCellSprite`, scaled to the footprint, with a local Z
that cancels the ghost root's storey so the world Z lands on the planet. The capture pass asserted
it directly and every number came back right:

```
Factory: the ground mark - drawn = True, has a picture = True, alpha 0,51,
         at z -0,090 against a floor at -0,010, the ghost at -4,010
```

Enabled, a live sprite, a visible alpha, the correct world position — and no pixels. The whole
frame was scanned for the ghost's own outline colour and the only rectangle in it was the ghost's:
64 by 36 at the ghost, and nothing at all 207 pixels below it where the projection puts the mark.

**What was ruled out.** Depth against the terrain (the mark sits nearer the camera than the ground
it covers, and `SpriteShadow` puts its silhouette on the planet at *less* clearance and draws
fine); a stale `HideAndDontSave` sprite reference (re-fetched per frame, no change); alpha; sorting
order; the mark being hidden behind the player (its edges fall clear of them); a footprint of zero
size.

**Where to start.** The cell grid works and the mark does not, and the differences between them are
few enough to bisect: the cells hang off `_cellRoot` rather than off the ghost root directly, they
are created during play rather than in the constructor, and their local Z is a hair rather than a
whole storey's worth. Try a mark parented to the world instead of to the ghost — which is what
`SpriteShadow.EnsureGround` does, and it is the one thing in the game already doing this job
successfully.

**Meanwhile** the storey is said in words: the placement prompt reads `LEVEL 2` while the ghost is
being aimed by hand, which is the information the mark was for.
