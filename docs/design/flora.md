# Flora

How a plant comes to look like it grows here. Written after the rework that made the trees match
the world they stand in, because three of the four things wrong with them were the same mistake in
different places and none of it is obvious from the code.

Flora is not a hull. A building's shape is a heightmap on a plan and its sides are a mesh —
[`hulls.md`](hulls.md) is the authority on that and none of it applies here. A plant is a picture,
stood upright and turned to face the camera, and it stays one: a wood is thousands of trees and no
number of them should cost what a building costs. What follows is about the picture.

---

## A cell is not a size

The sheet is cells of `FloraCell` pixels, and a plant is drawn at `PixelsPerUnit` — the same
resolution as the ground, the character and every machine. How much of its cell a plant fills is
how big that plant is, and `FloraDef.Height` in tiles is the instruction the drawing obeys.

It was the other way round for a long time: a small cell, scaled at placement by `FloraDef.Size`.
Two things came out of that, and they are the same thing:

- **A tree was drawn at half the resolution of the dirt under it.** Sixty-four pixels stretched
  over two tiles is sixteen pixels to a tile, beside a world drawn at thirty-two. Chunky foliage on
  crisp ground reads as a sticker, and repainting the sticker does not fix it.
- **A tree came out shorter than the player**, while its own definition said "twice a person". The
  size lived in the scale, so nothing checked it against the number that claimed to say it.

The scale at placement is now variation and nothing else — no two plants of a kind quite the same
size. If a plant is the wrong size, the fix is `FloraDef.Height`, and the drawing follows.

## Foliage separates by light, not by outline

The obvious way to build a crown is a handful of leafy clumps, each with a dark rim so they read
separately. **That draws a cabbage.** Six rimmed discs in a ring, every seam picked out in black,
and no arrangement of them escapes it — the eye reads the rims, and the rims are circles.

What works is the other order:

1. **One silhouette.** Four overlapping patches, flat filled, no rims, merging into a single
   ragged outline. Nothing here is meant to read as a patch.
2. **The underside**, down and to the right of the sun. A canopy is thickest where it is deepest,
   and the bottom of it never sees the sky.
3. **The lit clumps**, up and to the left, five of them, deliberately unequal and deliberately not
   touching. Light through a canopy is dappled. One lit patch the size of the crown is not a lit
   tree, it is a splash of paint on a dark one.
4. **A couple of chips** of the brightest value where the light lands squarest.
5. **A broken edge**: sprays off it and bites out of it.

Three values and a broken edge is all foliage is at this size.

**A crown spreads.** It is drawn as an oval wider than it is tall — a tree stops racing upward for
the light once it has won and starts going out sideways for more of it. Built out of circles it
comes out a ball however roughly the circles are cut.

### The bites have to actually cut

`Pixels.Blend` returns early on a fully transparent colour, which is right for painting and wrong
for taking a bite out of a silhouette. Every crown in the game had been calling `Disc` with a
transparent colour for the bites, and every one of those calls did nothing — so foliage had been
added to and never cut, which is the one construction guaranteed to come out convex.
`Pixels.Erase` is the one that cuts.

## Values, against the planet rather than against the palette

Every ground on this world sits between about 0.13 and 0.39 in value, and all of them are cold.
The old foliage was painted at two thirds brightness in near-pure mint: three times the value of
the dirt it stood on, and the most saturated thing on the screen.

So **the mass of a plant is darker than the ground it grows out of**, and only its lit face comes
up past the ground's own value. That is what foliage does — a canopy is a hole in the sky with a
bright edge, not a lamp. The one plant allowed to be pale is rime bristle, which takes the ice
nothing else will and is named for frost, and even that sits under the ice rather than over it.

## Timber

- **The light is up and to the left, so the left of a trunk is the lit edge.** It was the other way
  round for the whole life of the old drawing, and a trunk lit from the wrong side reads as off
  without anybody being able to say why.
- **Roots are separate things, not a swell in the column.** Swelling the trunk toward the ground
  draws a cone, and a cone under a pole is a lamp post on a base plate. Two short uneven wedges
  spreading out of the base read as roots; two matched ones read as the foot of a wine glass.
- **Boughs have to reach past the crown's own outline.** A bough entirely inside the leaves is a
  bough nobody will ever see, and the thing it is there to say — that the mass on top is carried
  rather than balanced — is only said by the part that shows.

## Shadows

Every plant with a shape worth laying down throws a real one, laid over from its roots the way a
character's is, following the sun through the day. It used to throw a contact shadow painted into
its own cell, which was the cheap answer to the wrong half of the problem.

The expensive part of a shadow here is not drawing it. It is **joining `ShadowField`** — a list
walked in full by every sprite that asks whether it is in shade, which a couple of hundred plants
would turn into tens of thousands of tests a frame. So flora casts and does not answer: walking
under a tree does not darken you. That is a small loss against a wood that sits on the ground.

Two knobs exist for this, both on `SpriteShadow`:

- `field: false` on `Attach`, which is what keeps them out of the list.
- `Reach`, the correction for a caster standing on its own shadow. A figure is a column and covers
  the near half of theirs with their own boots; a tree is a crown three tiles up on a stick and
  covers almost none of it, so it wants less.

A tuft throws nothing at all. Bristle is the thickest thing on the planet by four to one, its
shadow is eight pixels nobody will pick out of the speckle on the dirt, and it would be the
majority of every shadow in front of the camera.

---

## Checking one

The same rules as everything else — [`README.md`](../../README.md#checking-a-change):

1. `tools/compile-check.sh`.
2. One build, then `-factoryScenes terrain`, which is the pass that walks to a tree and stands
   either side of it (`80-behind-a-tree`, `81-in-front-of-a-tree`) and then pulls out to
   `77-terraced-country` for the wood at distance. **Both zooms matter and they disagree**: the
   first tree that read well up close was a row of cabbages at the far one.
3. Read the `FAILURES` line, then look at the pictures.
