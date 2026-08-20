# ATLAS

A chord instrument for the Ableton Move, built as a [Schwung](https://github.com/charlesvestal/schwung)
overtake module.

Twenty pages of harmony on the pads — triads, sevenths, blues, secondary
dominants, the circle of fifths, ninths, elevenths and thirteenths, altered
dominants. Plus an arpeggiator, a strummer, two bass lanes and a chord library
you build as you play.

![The panel](/panel(1).svg)

---

## Read this first

**ATLAS sends notes over USB.** Plug the Move into a computer and it appears as
a MIDI device — DAW, softsynth, whatever you like.

**It cannot play Move's own internal synths.** Not a bug in ATLAS: the function
that would reach them runs and delivers nothing in the current Schwung release,
and other overtake modules hit the same wall. If a later Schwung fixes it,
**Shift+Mute** turns on that second route — it's already wired up and off by
default.

So: USB works, internal doesn't. Set your DAW up before you wonder why it's
quiet.

---

## Install

Drop the folder into your Schwung modules directory:

```
scp atlas-module-P8.7.tar.gz ableton@move.local:/data/UserData/
ssh ableton@move.local 'tar -xzf /data/UserData/atlas-module-P8.7.tar.gz \
  -C /data/UserData/schwung/modules/overtake/'
```

Then pick **Atlas** from Schwung's Overtake menu.

---

## The one idea that makes it click

**Left and right changes which chord. Up and down changes how thick it is.**

Every page is eight columns by four rows. Columns are the chords — I, ii, iii,
IV and so on. Rows are how much you're stacking on top: a plain triad at the
bottom, sevenths and ninths as you climb.

So sliding sideways moves through a progression, and sliding up the same column
reharmonises the chord you're already on without changing what it is. Once
that's in your fingers the whole thing opens up.

**The colours tell you the job.** Green is home, blue is the pull away from
home, amber is the tension that wants to resolve, magenta is borrowed from
somewhere else. Each page shifts the whole palette a little so you can tell at
a glance where you are.

---

## Getting a sound out

1. Plug into a computer, arm a MIDI track.
2. Press pads. You're on **TRIADS-6** by default — the full major scale.
3. Turn **K2** to change key, **K3** for major/minor.
4. Turn the **jog wheel** to browse pages; the name blinks until you **click**
   to commit.

That's most of it. Everything below is depth.

---

## The pads

| | |
|---|---|
| Press a pad | that column's chord, at that row's thickness |
| Slide up a column | same chord, richer |
| **Up / Down** | move the chords an octave |
| **Shift + Up / Down** | move the bass an octave |
| **Shift** (tap) | jump to the relative minor |
| **Undo** | panic — stop everything |

Panic matters here: some things ring until you stop them deliberately.

---

## Four ways a pad can play

The four **track buttons** across the top:

| | Mode | What a pad press does |
|---|---|---|
| 1 | **ADDED BASS** | the chord, plus its root two octaves down |
| 2 | **CHORD** | just the chord — the default |
| 3 | **ARP** | one note at a time, in time |
| 4 | **STRUM** | rolled out like a guitar |

**ARP needs the transport running** — press Play, or nothing happens. Press
track 3 again to cycle nine directions: up, down, up-down, down-up, converge,
diverge, random, the order you pressed the pads, and all-at-once.

**STRUM** cycles four directions on its button too: forward, reverse, and two
humanised versions — `RANDT` varies the timing only, `RANDX` also swaps the
odd neighbouring note. A real hand is uneven, not out of order, which is why
they're separate.

---

## Holding things

| Button | |
|---|---|
| **Loop** | fire the last chord again, in the mode it was played |
| **Copy** | LEGATO — every chord rings until the next one. Blinks while armed |
| **Undo** | stop everything |

Legato and page-change sustain have **no timeout**. That's deliberate — a
progression shouldn't gap because you paused — but it means Undo is how you
stop, not waiting.

---

## The sixteen step buttons

| Steps | |
|---|---|
| **1–8** | the bass notes of the key. 1 and 8 are the root, in red |
| **9** | silence — somewhere neutral to land |
| **10–13** | notes from *the chord you just played* — 3rd, 5th, 7th, ♭7 |
| **14–16** | two-note figures: root+3rd, root+leading tone, root+5th |

Both lanes sit an octave under the chord.

**A dark step can't sound.** Ask for a 7th on a plain triad and that button is
unlit before you reach for it — it tells you rather than playing nothing.

Steps 14–16 play both notes together, except in ARP where they alternate, which
gives you a bouncing bass figure.

---

## Timing

Press **Menu** — the steps become a rate strip and the button blinks. Steps 1–8
are dotted values, 9–16 straight and triplet. Pick as many as you like; press
Menu again to leave.

**Click the jog while it's open** for **dual rates** — sixteen two-speed
patterns. The first few notes go at one rate and the rest at another, then it
wraps round. `1/4>1/8` accents the first two notes; `3x8>16` gives you three
slow then a run.

**Capture** locks the arp to the bar: it plays its notes, then waits for the
next downbeat instead of restarting immediately. The button pulses at whatever
rate it's quantising. 4/4 only.

---

## Knobs

| | | Steps to move it |
|---|---|---|
| K1 | tempo, 40–240 | 5 |
| K2 | key | 6 |
| K3 | major / minor | 8 |
| K4 | spread — CLOSE, DROP2, DROP3, OPEN, ROOTLESS | 6 |
| K5 | inversion — 0 is automatic | 6 |
| K6 | strum spacing, 40–100 ms | 6 |
| K7 | arp note length, 10–200% | 6 |
| K8 | which note the arp starts on | 6 |

They're deliberately stiff so you don't knock them mid-performance.

**One click reports, it doesn't change.** Tempo, strum, gate and offset appear
nowhere on screen, so a single click tells you the current value.

**K5 at 0 is automatic voice leading** — chords step to each other instead of
leaping. Turn it up for manual inversions.

---

## The pages

![All twenty pages](docs/pages.svg)

| | | |
|---|---|---|
| 0 | POWER | fifths, no thirds |
| 1 | QUAD-ii | IV V I ii |
| 2 | QUAD-vi | IV V I vi |
| 3 | TRIADS-6 | the whole major scale — start here |
| 4 | BLUES | I IV V sevenths and the turnaround |
| 5 | SEC DOM | the scale plus borrowed dominants |
| 6 | CIRCLE | ii–V–I round the circle of fifths |
| 7 | 7THS | seventh chords |
| 8 | TENTHS | the same harmony, spread wide |
| 9 | MELODY | single notes, four octaves |
| A | INTERVL | two-note shapes |
| B | BUILDER | pick a chord quality, root from K2 |
| D | NINTHS | ninths, all in key |
| E | 11-13 | elevenths and thirteenths |
| F | ALTERED | altered dominants and tritone subs |
| G | SUS-DIM | suspended and diminished |
| C | USER | your own — see below |
| H | HELP | this, on the device |
| M | MONITOR | diagnostics |
| P | PALETTE | colour reference |

**On ALTERED**, columns are the alteration and **rows are the root** — the
opposite of everywhere else. It's the only way to fit both into 32 pads.

**On NINTHS**, the iii and vii chords have a flat ninth. That's correct theory,
not a mistake — it's what the scale gives you — but it'll sound darker than the
rest of the page.

---

## Building your own page

Press **Record** to capture the chord you just played into page **C USER**.
Keep going and you build a set of favourites, from whichever pages you like.

Captured chords are stored as *shapes*, so they follow you when you change key.

**They're lost when the module reloads.** Schwung gives JavaScript modules no
way to write files. You can also hand-write `pages/user.json` instead, which
does survive — the format's in the developer README.

---

## When something seems wrong

| | |
|---|---|
| No sound at all | it's USB only — check your DAW, not the Move |
| ARP silent | press Play; it's clock-driven |
| A step is dark | that chord hasn't got that note |
| Notes won't stop | **Undo**. Legato and sustain have no timeout |
| Screen looks stale | check the version on page **M** matches what you installed |

---

## Credits

Built on [Schwung](https://github.com/charlesvestal/schwung) by Charles Vestal.
Chord-map layout inspired by ChordMaps2. Thanks to
[schwung-control](https://github.com/chaolue/schwung-control) — reading its
source is what finally identified the working MIDI output path.
