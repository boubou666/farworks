# Fauna

What lives here, why the first of it stands on ore, and what has to exist before anything can be
hit — health, a stick, and a way to die that does not end the run.

**Built in 1.65.0: all six steps of the order below.** The lodeback and its guarded seams, health
and the one door damage goes through, the Scrap Maul, dying and the three pack rules, the three
hostility modes with the world-creation screen they are chosen on, and seams that fill back up.

What is *not* built is everything under **What is deliberately not built**, and that list is the
more important half of this document. This was written before a line of it existed, as the argument
for which parts of a very large idea are worth having at all and in what order -- and the parts left
out were left out on purpose.

---

## The rule everything below is measured against

Hostile fauna is the standard way a factory game turns into a different game. The path is well
worn: something attacks the base, so the base needs walls; walls need turrets; turrets need ammo;
ammo needs a production line; and within two versions the interesting question is damage per belt
and the factory is the thing you build in order to fight. The combat game wins that argument every
time, because a threat with a clock on it out-shouts a puzzle without one.

Farworks gets its pressure from scarcity and distance. Nothing generates within 48 tiles of the
pad. The pod hands over enough metal for exactly the station, the bench and two tools, and not one
plate more. A tool takes fifteen seconds against a machine's two. That pressure is the game, and a
wave timer does not add to it — it replaces it.

So, the test every idea below has to pass:

> **Is the answer to this a factory answer or a shooter answer?**

Factory answers are walls that are also fences, routes that go round, ground you take and hold by
building on it, light, and a machine that keeps them off. Shooter answers are damage per second,
ammo counts, kiting and cooldown management. Take the first kind. Leave the second.

---

## A guard is a place, not a clock

**The first creature — and for a good while the only one — stands on ore and does not leave it.**

It does not path to your base. It does not come at night. It is where the copper is, and getting
the copper means dealing with it. That single decision is what keeps the whole feature inside the
game that already exists: it makes fauna another statement about *where you may expand*, which is
the exact pressure the planet is already built around, only sharper. A surveyed deposit is no
longer a walk; some of them are a walk and a decision.

It is also, by a wide margin, the cheapest thing to build. A creature tied to a site needs local
steering and a leash. A creature that hunts you needs pathfinding across terraces, ramps, storeys,
belts and buildings — and that is not `MoveTowards`, it is a real navigation problem on a world
that has been getting harder to navigate every version since the 3D conversion. Guards give us
fauna without solving it, and if it is ever solved, guards are still right.

### Which deposits are guarded

Rolled from the seed with the deposits themselves, so a planet is the same planet every load and a
guarded seam is guarded for whoever surveys it.

- **Never the starter node.** `EnsureStarterDeposit` puts one iron node inside the clear radius, and
  it is the only ore a new pilot can reach. It stays free.
- **Nothing inside the exclusion radius** — 48 tiles, the ring the opening happens in.
- **Further out, and richer, means more likely and more of them.** Not every distant seam: a
  planet where all the good ore is occupied is a planet with one mechanic, and the pleasure of
  finding a fat unguarded seam depends on guarded ones existing.
- **Titanium and quartz lean guarded.** They arrive late, they matter, and they are the right place
  for the difficulty to sit.

A guarded site should read as guarded from outside its reach: bones, a trampled ring, a different
scatter on the ground. Walking into it should never be the way you find out.

---

## Hostility is an option, not a design

Three settings, chosen when the world is made and changeable at any time afterwards:

| Mode | What it means |
| --- | --- |
| **None** | No fauna at all. The game as it is today. |
| **Provoked** | They hold their ground and warn you off. They never strike first. |
| **Aggressive** | They come for anyone who gets close, and go home again. |

**Provoked is the default** for a new world. It is the mode this design is actually for: the
creature is an obstacle you choose to take on, at a moment you choose, and a player who wants no
part of it can survey elsewhere and never fight anything. Aggressive is for people who want the
site itself to be dangerous ground.

**The mode is never baked into the planet.** This matters more than it sounds. The seed places
guard sites and their populations under every setting, including *None* — what the setting changes
is whether they are brought to life. Turn fauna on halfway through a run and the same sites
populate, in the same places, at the same strengths; turn it off and they are gone without the ore
moving an inch. The alternative is a world whose *shape* depends on a switch, and then the switch
can only be thrown once, at creation, which is exactly the thing being asked for and not.

Under **Aggressive**, two numbers make it fair rather than miserable: an **aggro radius** — how
close is too close, measured from the site rather than from the creature — and a **leash**, how far
one will chase before it turns round and goes home. The leash is not a detail. Without it, a chase
that starts at a copper seam ends at your smelter, and the game has quietly become base defence
through the back door.

Where these live: new fields on `SaveGame`, ordered so that **0 is the value that makes an old save
correct** — `None` first, because a save written before any of this had no fauna in it and must
come back with none. A new world sets `Provoked` explicitly rather than leaning on the default. In
co-op the host owns both settings and a guest is told what they are; a guest changing the world's
difficulty from their own pause menu is a moderation question, not an option.

### The screen this is chosen on

There was no world-creation screen at all: `NEW GAME` in both menus booted straight onto a planet,
which is the right answer for a game with nothing to decide. Building one was most of the cost of
these two options, and it carries a seed field as well, because a screen that exists is the only
place a seed could ever go.

Both settings are also on the pause menu under **THIS WORLD**, apart from `OPTIONS` because that
page is about the machine — the window, the speakers, the keyboard — and these are about the
planet. They travel in the save, and in co-op they belong to the host.

---

## Health

**One number, on `Actor`.** Not on `PlayerController`, not on a Unity component. The simulation
being Unity-free is the most valuable property this codebase has for the co-op that is coming, and
a health pool is state that has to agree between machines. It goes in `ActorSave` beside the pack,
and a save without it reads as full health.

**All damage goes through one door.** Not `actor.Health -= n` scattered across the callers, but a
single `Damage(amount, source)` that owns the whole question of what absorbs it. That is the entire
preparation for shields, and it is worth doing on the first day:

- **Shields later** are a second pool that takes the hit before health does, recharges on its own
  after a few seconds without one, and hangs off the `Body` apparel slot — which already exists and
  has nothing in it. Nothing else needs to know they were added: `Damage` walks the pools in order,
  and there is one more pool.
- **The readout has room for two bars from the start**, even while there is only one to draw. A
  health bar that has to be moved and rebuilt the day a shield arrives is a small job done twice.

**What brings health back.** Slowly, on its own, once nothing has hurt you for six seconds: two and
a half a second, all the way to full.

The cap was 60% in the first draft of this, on the argument that a bad fight should still be costing
you ten minutes later. It was wrong, and playing it is what settled it: what a fight actually costs
is the walk, the swings and the half minute of standing about afterwards, and a permanent ceiling
mostly leaves somebody at their base wondering what they are supposed to do about the missing 40%.
The knob is still there — `Actor.RegenCap` — because the food chain below wants exactly it back the
day a cooked meal is what takes you the last stretch.

---

## The stick

**A melee weapon first, and for the first release, only.**

The reason is the ammo chain. A gun that eats a manufactured round is a new production tree, a new
belt run and a new thing your factory exists to feed — the single biggest step towards the game we
decided not to make, and it can be taken later from a standing start. A stick costs a recipe at the
bench and nothing else.

It also makes distance the resource. To hit something you have to close with it, which means
backing off is always available and always works, which is what lets a player decline a fight they
have wandered into. A ranged weapon quietly takes that away.

Mechanically it is the smallest thing that can work:

- **In the equipment wheel**, alongside the pistol and the scanner, on the same wheel and in the
  same hand. What you are holding already matters; this makes it matter more.
- **A swing, not a hit at range.** Left mouse, a short cooldown, and a cone in front of the
  character about a tile and a half deep — the reach the world already uses for touching things
  (`FloraHarvest.Reach`, `SleeperPack.Reach`). Everything in the cone is hit, so a swing at two
  crowded animals hits two.
- **It is a poor weapon and reads as one.** Scrap and a rod. The pistol stays a mining tool and
  never becomes a gun by accident.

Whether it also cuts scrub faster is a nice question and a later one.

---

## Dying

**You come to at the Main Station**, the one building the game guarantees exists — the opening is
built so that it must be standing before anything else can happen. If it has been taken up, the pad
is the fallback.

What happens to what you were carrying is the second world option, changeable in play like the
first:

| Mode | What happens |
| --- | --- |
| **Keep everything** | You wake at the station with your pack intact. |
| **Leave a cache** *(default)* | Everything you carried goes into a container on the ground where you fell. |
| **Lose everything** | The pack is gone. |

**The cache is an ordinary container**, holding exactly what you had, standing where you died, and
marked on the map for as long as it exists. It does not tick away — a timer on it turns a bad
afternoon into a lost one, and there is nothing to be gained by it. Die on the way back and you
have two caches; that is the punishment, and it is enough. In co-op anyone can open one: a mate
fetching your pack is one of the better reasons to have a mate.

**One hard rule under every mode: the tools are never lost.** The pistol and the scanner stay on
the character. This is not generosity, it is the rule the incinerator already follows — the opening
has no slack in it at all, and losing the pistol means you cannot mine, which means you cannot make
a pistol, which means the run is over and the game never says so. A cache you can walk back to is a
setback. A pistol at the bottom of the canyon you died in is an unwinnable save.

---

## Remains

**Alien Meat.** What a dead creature leaves, and the first thing on this planet that comes from
something that was alive.

- **`AlienMeat = 30`**, numbered after `Rivet` for the reason every id here is numbered in the
  order it arrived: an id is what a save writes down, and renumbering the list turns every stack of
  rivets in an existing hold into meat.
- **A range, not a figure**, and **rolled by whoever is holding the world** — the rule mining and
  flora drops already follow, for the same reason. Two machines rolling their own yields are two
  different factories by teatime.
- **The range is per creature kind**, and a bigger animal is worth the walk.

What it is *for* wants deciding before it ships, because an item with no use is a stack that sits in
the hold looking like an oversight. Two candidates:

- **Fuel.** The Biomass Burner already takes fuel by quality, and meat is a fuel value in a table.
  It costs nothing to ship and it is honest — this is a planet where the first fuel was leaves.
- **Food** *(later)*, which is what it actually wants to be: cooked at a machine that does not exist
  yet, and the thing that puts health back to full. This is the better answer and it is a whole
  system away.

Recommendation: ship it as fuel, design the food chain against it, and let the fuel use quietly
become the poor option once cooking exists — exactly what happened to leaves.

---

## Coming back

**The site repopulates. The creature does not respawn.**

The model already exists, and it is `FloraRegrowth`: what gets written down is not the population
but the short list of places the player has interfered with, each with a clock. A cleared guard site
holds a count and a timer, refills to its cap over minutes rather than seconds, and the untouched
sites on the rest of the planet stay free — costing nothing, stored nowhere, and rolled from the
seed when somebody finally walks there.

Three rules make the difference between a system that respects your afternoon and one that does
not:

- **Never within sight.** Nothing appears on a tile anyone can see. This is what stops repopulation
  from reading as cheating, and it is most of the whole of it.
- **Never in a base.** A tile that is built on, lit, or walked daily is not somewhere anything comes
  back to.
- **Clearing a site buys real time.** Four minutes an animal. If you clear a seam and it is occupied
  again by the time the miner is placed, the fight was a toll rather than a decision, and a toll is
  the thing this document exists to avoid.

And one rule that is about a fight rather than about a site: **an animal that loses its quarry walks
home and mends there**, three health a second, about twenty seconds for a whole one. Without it, the
cheapest way to clear any seam is a war of attrition — out, two swings, away, back in a minute, two
more — which is a chore with a timer rather than a fight anybody decided to have. It only mends on
its own ground, so chasing one off its ground is still worth something.

And the part that satisfies the founding rule: **build on it and it stops coming back.** A guard
site under a miner, a foundation or a lit yard is cleared for good, the same way a bush cannot grow
back through a floor. Taking ground by putting your factory on it is a factory answer to a fauna
problem. That is the shape every later addition here should have.

---

## Drawing one

A creature is a picture stood up and turned to face the camera, exactly like the characters and the
flora, through `Billboards`. `hulls.md` is about buildings and none of it applies. `flora.md`
mostly does:

- **Painted at the world's resolution**, and how much of its cell it fills is how big it is. A
  creature scaled up from a small cell is a sticker on crisp ground, and this was learned once
  already.
- **Lit from a painted relief, never from an outline.** The standing rule for anything that stands
  up.
- **Values against the planet, not against the palette.** The ground sits between 0.13 and 0.39 in
  value and all of it is cold. A creature is a dark mass with a lit face, and the one thing on it
  allowed to be bright is whatever says *this one is looking at you*.

Movement is where a picture earns its keep: a walk cycle, a warning posture that is unmistakable at
a glance under **Provoked**, and a wind-up before a strike long enough to back out of.

---

## Co-op

Creatures are **host-authoritative** and ride the snapshot the way actors do. Everything in the
world today is either derived from the seed or the direct result of a player action, and these are
the first things that are neither — they move on their own, on the host, and guests are shown the
result. Nothing here should be simulated on a guest, including the swing that kills one.

`coop.md` is not built yet, which is an argument for landing fauna before it rather than after: a
snapshot designed while moving entities exist is a snapshot that has them in it.

---

## What is deliberately not built

Written down so that finding one of these missing is not mistaken for an oversight:

- **Waves, raids, and anything that comes to the base.** The whole argument at the top.
- **Turrets and ammunition.** No defence buildings, no ammo item, no ammo recipe.
- **Building damage and repair.** Nothing can hurt a machine, so nothing needs mending. This is a
  large system in disguise: every building would need a health field in the save, and every machine
  screen a state for it.
- **Long-range creature pathfinding.** Guards steer inside their site and leash home. Nothing
  navigates the planet.
- **Bosses, nests that grow, spreading territory.** Each of them turns fauna back into a clock.

All of these become possible later, and none is a prerequisite for anything above.

---

## The order to build it in

1. **A creature that is alive and cannot be touched.** Placement from the seed on guard sites,
   drawing, walk, idle, save and load, host authority. No damage in either direction. This was the
   whole of the hard part — spawning, movement over terraced ground, the draw and tick budget, the
   snapshot — proved at no design risk. *Done.*
2. **Health, and the door all damage goes through.** With the readout built for two bars. *Done.*
3. **The stick, and hitting things.** Remains drop, rolled by the host. *Done.*
4. **Dying.** The station respawn and the three pack rules, and the world-creation screen they
   needed. *Done.*
5. **The hostility modes**, and the runtime switch for both options. *Done.*
6. **Repopulation**, with the three rules and the build-over-it clause. *Done.*

Nothing after step 6 is decided. The open questions below are where it would start.

---

## Open questions

- **Does a guard drop anything besides meat?** A material only they carry would make fighting one a
  supply line rather than a toll, and that is a real fork in the road — it is the difference between
  fauna being an obstacle and fauna being a resource. Leaning: not at first.
- **Are they on the map?** A surveyed deposit tells you it is there; should it tell you it is
  occupied? Leaning: yes, once surveyed, because a survey that hides the one thing that matters
  makes the scanner feel like a liar. `FaunaHerds.Guarded` already answers the question — nothing
  asks it yet.
- **A guarded seam should read as guarded from outside its reach**: bones, a trampled ring, a
  different scatter on the ground. Not built, and the one piece of the original design that is
  missing rather than deferred — walking into it should never be the way you find out.
- **Night.** The obvious move is that they are worse after dark, and the game already has a day. It
  is also the fastest way to turn a guard back into a clock. Leaning: no — and if ever, only under
  Aggressive.
