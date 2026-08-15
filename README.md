# ATLAS — P6

ChordMaps-style chord map for Ableton Move. Schwung overtake module, JS only.

**Pad modes and a second bass lane.** The track buttons switch what a pad does;
steps 9–16 play chord tones that follow the harmony rather than the key. Chords
voice-lead between presses, K4 spreads and K5 inverts. Every pad on every page
sounds; PALETTE stays a diagnostic.

Measured on hardware: ~196 ticks/sec, ~28 ms full-grid sweep.

A one-page control reference with panel and page diagrams lives in
[docs/QUICKREF.md](docs/QUICKREF.md); bullet-point release notes are in
[docs/RELEASES.md](docs/RELEASES.md).

## Build and deploy

```bash
bash scripts/build.sh      # tarball is suffixed from the PHASE file

scp dist/atlas-module-P6.tar.gz ableton@move.local:/data/UserData/
ssh ableton@move.local 'tar -xzf /data/UserData/atlas-module-P6.tar.gz \
  -C /data/UserData/schwung/modules/overtake/'
```

No reboot — JS-only changes need a rescan at most. Logs:

```bash
ssh ableton@move.local 'touch /data/UserData/schwung/debug_log_on'
ssh ableton@move.local 'tail -f /data/UserData/schwung/debug.log | grep atlas'
```

## Controls

| Control | Action |
|---|---|
| Pad | play; ruler column boxed, chord named below |
| Steps 1–8 | bass lane, diatonic degrees 1–8 (lit; bright while held) |
| Steps 9–16 | chord-tone lane: ROOT 3rd 5th 7th ♭7 PED DRN R+5 |
| Track 1–4 | pad mode: ADDED BASS / CHORD / ARP / STRUM |
| Track 3 again | cycles arp direction — nine of them |
| Loop | HOLD — latch the sounding chord |
| Copy | LEGATO — chord rings until the next chord |
| K1 | tempo, 40–240 BPM (default 140) — 5 detents, 1 detent reports it |
| K2 | key (C–B) — 6 detents per step |
| K3 | scale (major / minor) — 8 detents per step |
| K4 | spread: CLOSE / DROP2 / DROP3 / OPEN — 6 detents |
| K5 | inversion: 0 = auto voice-lead, 1–4 manual — 6 detents |
| K6 | strum spacing, 40–100 ms — 6 detents |
| K7 | arp gate, 10–200% of the step — 6 detents |
| Up / Down | chord octave ±1 (clamped ±4) |
| Shift + Up / Down | bass octave ±1 (clamped ±4) |
| Shift (tap) | latch / unlatch the relative-minor layer |
| Shift (hold) | modifier — a combo suppresses the latch on release |
| Left / Right | page back / forward |
| Jog turn | browse the page rail — name blinks until committed |
| Jog click | commit the browse — or, if idle, the page's own action (BUILDER: root latch) |
| Capture | full grid repaint, and panic on a sustained chord |
| Play | start / stop; pulses on the beat. Also passed through to Move |
| Menu (toggle) | open / close the rate strip; lit while open |
| Rate strip, steps 1–8 | dotted rates: 2/1. … 1/64. |
| Rate strip, steps 9–16 | straight and triplet: 1/1 … 1/16T |
| Back | all notes off, exit |
| Record | unused — reaches us as CC 86, no function assigned |
| Shift + Vol + Jog click | host escape, always works |

Both lanes sit an **octave below the chord's root** — the root as voiced, not
the lowest note, which under a drop voicing is often the third or fifth.

Steps **1–8** play degrees of the *key*; steps **9–16** play tones of the *last
chord played*. That difference is the point — the second lane stays right as
you move through a progression, which the first structurally cannot do.

### Encoders

| | Assigned | Notes |
|---|---|---|
| K1 (CC 71) | tempo | 5 detents, default 140 |
| K2 (CC 72) | key | 6 detents |
| K3 (CC 73) | scale | 8 detents — the heaviest change, so the stiffest |
| K4 (CC 74) | spread | 6 detents |
| K5 (CC 75) | invert | 6 detents |
| K6 (CC 76) | strum spacing | 6 detents, 40–100 ms |
| K7 (CC 77) | arp gate | 6 detents, 10–200% |
| K8 (CC 78) | **free** | inert — turning it does nothing |

**Only K8 is free.** An unassigned knob sends CCs that are read and discarded,
so it does nothing rather than something surprising.

Tempo, strum spacing and gate appear nowhere on screen, so a **single detent on
K1, K6 or K7 reports the current value without changing it** — that's how you
read them. Five or six detents change them.

**Strum spacing is geometric, not linear**, across 40–100 ms in twelve steps.
The old 500 ms top end was unusable — a strum that slow is an arpeggio.

**The gate is a fraction of the step, not a fixed time**, so it stays
proportional as you change rate or tempo. Below about 30% notes clip short and
staccato; past 100% they overlap into the next step, which is what gives legato
and drones.

## Transport lock

**What the JS API actually offers, measured rather than assumed:**

| | Result |
|---|---|
| MIDI realtime (F8/FA/FC) | filtered before JS — `int` past 15,000 with `rt` at 0 |
| `host_get_setting` | does not exist in this runtime — the probe returns `no api` |
| Reading tempo from disk | ruled out by choice |
| **CC 85 (Play)** | **arrives and counts.** Confirmed on hardware |

So the phase anchor is real and no tempo is reachable. ATLAS takes the tempo by
hand and lets the anchor do the rest.

That is bar-aligned rather than approximate: Move's tempo doesn't drift within
a set, so once the number matches, Play starts ATLAS on beat 1 with you and it
stays locked. Change the set tempo and you change this once.

| Control | Action |
|---|---|
| **K8** | tempo, 40–240 BPM, 5 detents per step |
| **Play (CC 85)** | start / stop, anchored to the press |

Play is still passed through, so Move's own transport runs alongside rather
than being replaced.

The tempo appears as a **toast**, never in the footer — the footer already
carries the layer, the mode and the meter, and a fifth item made it unreadable.
Nudging K8 a single detent is below the change threshold and **reports** the
current tempo without altering it, which is how you check it.

The Play button pulses on each beat. The beat steps forward by whole beats
rather than resetting to the current time, so a late frame can't accumulate
drift over a long take.

### Still open

Page **M (MONITOR)** stays, since it settled this and will settle the next one:
`int`/`ext`/`rt` counters, `P` for CC 85, a log of every button CC received,
and a jog-click tempo-key sweep. See the DSP item in the backlog for the route
to an automatic tempo.

## Display

```
3/15     TRIADS-6    ATLAS (o)
C Major                    V Gsus4
1  2- 3- 4  5  6- 7o b7
     OCT+1 BAS-1 SPD1 INV1
MAJ CHORD >             3 VOICES
```

Page index left, page name centred, wordmark right — the tightest row on
screen at a 9-character page name, so anything longer will collide. Row 2 carries the key on the left and what you just played on the right —
reference and feedback side by side, each anchored to its own edge so neither
shifts when the other changes length. The chord readout truncates rather than
running into the key.

Badges have a row to themselves, centred, and **all four are always shown** —
defaults included. An appearing badge made the row jump about and left it
ambiguous whether a value was zero or the badge simply absent. Each change also
raises a toast. The footer carries the
mode on the left (with `>` when the transport runs) and the **voice count** on
the right — just the number, in a fixed slot so it doesn't shuffle as it
changes. It's the one number worth watching while playing, since it's how you
spot a stuck note or an over-thick voicing without counting pads.

One line of eight fixed 16px cells, positionally mapped to the pad columns.
Tokens are capped at 2 characters — the panel is ~21 characters wide, so eight
full roman labels (25 chars) will not fit on a line. The detail line beneath
carries the real names.

Highlight style is the `HILITE` constant at the top of `ui.js`: `box` (default),
`underline`, or `invert`. Bold and italic are not available — the panel has a
single bitmap face.

## Time

Everything here runs off the transport clock subdivided by the rate strip, so
it stays phase-aligned rather than free-running alongside the beat. Nothing
sounds until Play is pressed.

**ARP** takes the held notes one at a time. It is clock-driven, so with the
transport stopped a pad press is silent — the screen says so rather than
letting it read as a fault. In ARP mode the pads contribute
notes silently — the arp decides when they sound, otherwise every note would
double. Note length is set by the gate on K7.

Press the ARP button again to cycle direction:

| | Direction |
|---|---|
| up | lowest to highest |
| down | highest to lowest |
| up-dn | up then back, turnaround notes dropped |
| dn-up | down then back, turnaround notes dropped |
| conv | outside in — lowest, highest, next lowest… |
| div | inside out |
| rand | random each step |
| order | the order you pressed the pads |
| chord | every note on each step |

Up-down and down-up drop the turnaround notes so they read as a loop rather
than stuttering at each end. **chord** is what the old REPEAT button did, moved
to where it belongs — one control deciding what the arp does, instead of two
fighting over the same notes.

**HOLD** (Loop) latches whatever is sounding and keeps it after you release the
pads. It re-claims the pitches through the ref-counted registry, which is
exactly the case that registry was built for. Press again to release.

Latching a second chord **flushes** the first rather than stacking on it, and
playing a new chord while HOLD is engaged replaces the latched one.

**LEGATO** (Copy) makes a chord ring until the next chord is triggered, rather
than until you lift the pads. It stays armed until you turn it off — a page
change or layer swap won't clear it — and the button blinks while it's on.

It claims pitches at press time, so arming it mid-chord affects the *next*
chord. Two modes need different handling: in **STRUM** it claims when the strum
finishes, since claiming at press time would grab every pitch before the
scheduler had delivered any and flatten the strum into a block chord; in
**ARP** it holds the note *pool* rather than the pitches, because there the
pads only supply notes and the arp decides when they sound.

**STRUM** spacing is on K6, from 40 to 100 ms.

## Rate strip

Press **Menu** to open the strip; press again to close. The button **blinks**
while it's open, and selecting a rate leaves it open so several can be
auditioned before exiting. The blink is what stops a latched strip being
forgotten — a steady light wasn't enough.

## Globe

A wireframe globe spins at the right of the header. It follows what's playing:
slow when idle, tracking the K6 spacing in STRUM, and the step time in ARP — so
it accelerates at 1/16 or a faster tempo.

The panel has no pixel or line primitive, so it's drawn from about 50 1×1 rects
per frame and capped at 20fps. That's the only thing on screen redrawing
continuously; the MONITOR page reports the achieved frame rate next to the
version so the cost is measurable rather than assumed.

| Steps | Lane |
|---|---|
| 1–8 | dotted: 2/1. 1/1. 1/2. 1/4. 1/8. 1/16. 1/32. 1/64. — one yellow |
| 9–16 | straight and triplet: 1/1 … 1/16T — straight ramp dim to bright, triplets share a pink |

Sixteen rates. Triplets have no useful dotted form, so the dotted lane is
straight-only. The selected step goes white, so no rate uses white.

**Changing rate re-anchors the step clock.** Without that, switching from a
slow rate to a fast one leaves the next step scheduled up to a whole slow-step
away and the arp appears to stall for a bar.

## Page sustain

A held chord survives a **page change or a Shift layer swap** and keeps
sounding until the next chord is played. There is deliberately no timeout — a
progression shouldn't gap because you paused.

The consequence is that changing page and walking away rings indefinitely, so
**Back** and **Capture** are the panics.

## Hardware checks

Ordered so a failure tells you which subsystem broke.

**Chassis**

1. Module appears under **Overtake** in the Schwung menu.
2. Grid fills left-to-right on load. No white flash, no stuck pads.
3. Screen shows `P3/15  TRIADS-6  ATLAS`, the key line, the ruler and the footer.
4. Read the footer meter — `<n>/s` is the real tick rate. Baseline ~196/s.
5. Tap Shift. Every column changes hue and the Shift LED brightens.
6. Hold Shift, press Right, release Shift. The page advances and the layer does
   **not** latch.
7. Jog one click — the page name blinks. Jog click commits and it goes solid.
8. Press pads across the grid. Each lights white, its ruler column is boxed, and
   the detail line names it in full.
9. Page **P (PALETTE)**: jog changes bank, Capture jumps +32, a pad reports its
   colour index.
10. Back exits to the Schwung menu.

**Sound**

11. Hold a chord on page 3, then press **step button 1**. Bass sounds an octave
    below.
12. Shift+Up twice, press step button 1 again — bass doubles the chord root.
    Release the **chord**: the doubled pitch keeps sounding. Release the
    **step**: now it stops. That is the ref-counted registry.
13. Turn K2 (six detents) — key changes, held notes drop cleanly.
14. Turn K3 (eight detents) to minor — the bottom row becomes Cm D° E♭ Fm Gm A♭ B♭.
15. Shift-tap on page 3 — reroots to A minor, colours swap to violet / teal /
    orange / green.
16. Change page while holding a chord — see *Page sustain* below.

**Voicing and pages**

17. Walk every page. Nothing silent, nothing out of key except the deliberate
    borrowed cells. Each page has a visibly different palette.
18. Play I–vi–IV–V on the bottom row of page 3. The chords step rather than
    leap — that's the voice leader. Turn K5 six clicks to override it.
19. On page 7, hold a chord and press step button 1. The chord thins by one note
    as the bass takes the root.
20. On CIRCLE, turn K2. Column labels **and** chords both move with the key —
    they must agree with each other, not just each look plausible.
21. Shift-tap on CIRCLE — eight minor keys, none dead, starting on the relative
    minor.
22. Set an octave, then change page and Shift-tap. The badge must not move.

**Modes and lanes**

23. Track buttons switch mode — 1 ADDED BASS, 2 CHORD, 3 ARP, 4 STRUM. The
    active one lights and the footer names it.
24. +BASS on page 3: press I, then IV. Each gains one low note, and it is the
    chord's root both times — not whatever note ended up lowest.
25. STRUM: the same chord, spread low to high. Tap quickly — no tail should
    keep sounding.
26. Play C, then press step 9 (ROOT). Play F, press step 9 again. The note
    follows the chord, not the key.
27. Play ii (D minor), press step 10 (3rd). You should get F, not F♯.
28. Play a plain triad and look at step 12 (7th) — **dark** before you press it.
    Play a seventh chord and it lights.
29. On CIRCLE, play a column then jog-click. The key commits and every other
    page follows. Playing a column without clicking must not move the key.

**LEDs**

30. Steps 1–8 lit on any sounding page. Step 1 and PED share a red; step 8 is a
    brighter red; 2–7 blue; 9–13 green; DRN violet; R+5 yellow.
31. Track buttons: the active mode carries its own colour, the rest are dark.

**Time**

32. Press Play, then Track 3 (ARP) and hold a chord. Notes come one at a time.
    Press Track 3 repeatedly to walk all nine directions — the footer names
    each. **chord** should strike the whole chord on every step.
33. Press **Menu** — it lights and latches. Pick 1/16, then 1/1: the arp speed
    follows immediately with no stall. Picking a rate closes the strip.
34. Open the strip and pick from **steps 1–8** — the dotted lane, in yellow.
    A dotted 1/4. should be half again as slow as 1/4.
35. Hold a chord, press **Loop**. Release the pads — it keeps sounding. Press
    Loop again to stop.
36. Press **Copy** to arm LEGATO, then play a chord and release. It rings until
    you play the next chord.
37. Play a chord and change page. It carries over, then stops after about three
    seconds. A new chord, Back, or Capture ends it at once.

**Tempo and transport**

38. Turn **K1** — a single detent reports the tempo (140 by default), five
    change it.
39. Press **Play**. The button pulses on the beat at the tempo you set, and
    Move's own transport still runs. Press again to stop; the light clears.
40. Change K1 while running — the pulse follows immediately.
41. In STRUM mode, sweep **K6** from 40 ms to 100 ms.
42. In ARP mode, sweep **K7** — 10% is staccato, 200% overlaps into the next step.
43. Turn **K8**. Nothing should happen; it is unassigned.
44. With the transport **stopped**, select ARP. The screen prompts you to press
    Play, and a pad press stays silent.
45. On **page 3**, press Up and Down. The chord should move by whole octaves —
    this was broken on every voiced page before P5.3.
46. Latch a chord with Loop, then latch a different one. The first should stop,
    not pile up underneath.
45. Cycle arp directions, toggle HOLD and LEGATO, then stop the transport.
    Nothing should be left ringing.

**Diagnostics**

46. Page **M (MONITOR)**: `int` climbs as you press things, `P` climbs on Play,
    `rt` stays at 0. Jog-click sweeps for a tempo key and reports `no api`.
    The footer shows the build version and the globe's frame rate — check the
    version first if the display looks like an older build.
47. The globe spins in the header. Switch to ARP with the transport running and
    pick 1/16: it should visibly speed up.
48. Open the rate strip — Menu blinks. Pick several rates in a row; it stays
    open. A second Menu press exits.

## Pad modes

| Button | Mode | A pad press |
|---|---|---|
| Track 1 | **ADDED BASS** | the chord plus its own root two octaves down |
| Track 2 | **CHORD** | plays the chord — the default |
| Track 3 | **ARP** | notes taken one at a time, on the beat. Press again to cycle direction |
| Track 4 | **STRUM** | the same notes fired low to high, spacing on K6 |

Exclusive; the lit button is the active mode, also shown in the footer. Modes
change only what the *pads* do — both step lanes behave identically throughout.

ADDED BASS takes its bass note from the chord's **root**, not the lowest sounding
note. The voicer inverts, so on an inverted chord the lowest note is often the
third — a bass note there would undercut the harmony.

STRUM is the first feature that schedules notes for later. Releasing a pad
mid-strum cancels whatever hasn't fired, so a quick tap can't leave a tail
sounding with nothing holding it.

## Chord-tone lane

Steps 9–16 resolve against the last chord played, preferring the tone the chord
is **actually sounding** over one derived from the scale — a diatonic third is
minor on some degrees and major on others, and guessing gets it wrong half the
time. If the chord genuinely has no such tone (asking for a 7th on a plain
triad), the lane says so rather than inventing one.

**Steps 14–16 are dyad figures** — root + 3rd, root + leading tone, root + 5th.
They sound together outside ARP and alternate on the step inside it, giving an
oscillating bass figure rather than a static dyad.

The leading tone comes from the scale's own seventh degree rather than being
assumed: a semitone below the root in major, a whole tone below in minor.

**A step goes dark when it cannot sound.** Ask for a 7th on a plain triad and
that button is unlit before you reach for it — a lit button promising a note it
won't play is worse than a dark one. PED and R+5 never depend on the chord, so
they stay lit.

### Step colours

Functional, not decorative. Red means *always the key root, never moves*.

| Steps | Colour | Meaning |
|---|---|---|
| 1 | red | root of the key |
| 2–7 | blue | other degrees of the key |
| 8 | brighter red | the root again, an octave up |
| 9–13 | green | chord tones — follow the harmony |
| 14 (PED) | red | holds the key root, so it matches step 1 |
| 15 (DRN) | violet | a moving anchor: follows the chord root |
| 16 (R+5) | yellow | root plus a fifth |

Step buttons take full colour indices through the same `setLED()` as the pads —
confirmed against
[TwinSampler](https://github.com/jrucho/schwung-twinsampler), which paints
notes 16–31 with arbitrary indices. Colour names are resolved defensively:
if a build lacks a named constant, it falls back to an index rather than
writing `undefined`.

## Help, settings and the USER page

**Page H** is on-device help: four lines at a time, jog to scroll, jog-click to
return to the top.

**Settings persist across loads** — page, key, scale, layer, octaves, spread,
inversion, mode, arp direction, rate, tempo, strum and gate. Writes are
debounced by five seconds since settings change in bursts as a knob turns.

This is written defensively. The P4.6 probe found `host_get_setting` **absent
at runtime**, so persistence works if the API exists and does nothing if it
doesn't; the MONITOR page reports `loaded`, `saved`, `no-api` or `error`. Every
value is validated on load, because a settings file from a different build
could otherwise point at a page that no longer exists.

**The USER page (C) loads from `pages/user.json`:**

```json
{
  "name": "MYPAGE",
  "cols":  ["I","ii","iii","IV","V","vi","vii","I8"],
  "abbr":  ["1","2-","3-","4","5","6-","7°","18"],
  "fam":   ["ton","sub","ton","sub","dom","ton","dom","ton"],
  "tiers": ["triad","sus4","7th","9th"],
  "grid":  [ [[0,4,7], ...8 columns], ...4 rows ]
}
```

`grid` holds semitone sets relative to the key root; row 0 is the bottom row.
`abbr` tokens are trimmed to 2 characters — the ruler is that wide. `fam` takes
`ton`, `sub`, `dom`, `bor`, `chr` or `off`. Every field is validated: a missing
or malformed file leaves the page as it shipped and reports on the MONITOR
page. An example ships in the tarball.

## Page behaviour

Most pages resolve by scale degree, so they follow K2/K3. Three don't, by
design — they use chromatic roots and would be silently re-spelled into the
scale if they did:

| Page | Root source |
|---|---|
| POWER | twelve chromatic roots from the tonic |
| CIRCLE | key centres cycling in fifths from the tonic |
| BUILDER | the key itself, plus an optional root latch |

BUILDER's latch is off by default and toggles on jog click — browsing 32
qualities must never disturb the key your progression pages are sitting in.

Out-of-key notes are deliberate in exactly three places: the ♭VII columns on
TRIADS-6 and SEVENTHS, the altered row on SEVENTHS, and the whole of BLUES
(a blues I7 is not diatonic — that's the point). Everywhere else, every pitch
on every diatonic page is in key; the test suite asserts it.

## Voicing

K5 at 0 hands inversion to the voice leader, which picks whichever rotation
moves least from the previous chord. Any other value is a manual inversion and
overrides it. K4 applies drop voicings rather than pushing voices up octaves,
which keeps the span inside about two octaves.

Three pages opt out of *automatic* voicing, because on them the shape **is** the
content: POWER (a power chord is defined by having no third), INTERVALS (the row
selects the interval width, so inverting a third into a sixth would silently
relabel the page) and TENTHS (every row already is a voicing).

Manual inversion on K5 **does** work there — it's asked for, unlike a voice
leader choosing an inversion on your behalf. Voice leading and spread stay off.

Each page shifts its whole palette by six colour indices so pages are
distinguishable at a glance. The trade: harmonic family is no longer a constant
colour across pages — the hue tells you where you are rather than what the
chord does. Only the ten base indices are hardware-confirmed; the offsets
aren't, so some pages may land on hues that read oddly. `PAGE_HUE_STEP = 0`
restores function-constant colour.

## Key-aware pages

CIRCLE and POWER label their columns from the live key. Their labels used to be
a fixed array, which lied: in the key of E, CIRCLE's first column still read
"C" while playing E. Turn K2 and both the ruler and the chords follow.

CIRCLE's shift layer is now a real minor circle — eight minor keys walking in
fifths from the relative minor, rows giving i6 / imM9 / V7 / iiø7. It was
previously four leftover major keys padded with dead columns.

## LEDs

Both step lanes light whenever the current page can sound, and brighten while
held. A dark lane means "nothing to play here" rather than "possibly broken".

Track buttons show the active pad mode in its own colour: CHORD green, +BASS
blue, STRUM yellow. Inactive modes sit dark. ARP is always dark — a dark button
is the honest signal for something that cannot be selected yet.

## What P6 leaves open

- **Unlit pads.** Pads are left dark in some situations; reproduction pending.
- **P7 — the DSP**, below.
- **P8 — the accumulated backlog**, everything else on this list. The strum scheduler built here
  is the machinery ARP needs, so it should be a small addition rather than a
  new subsystem.
- **BASS mode (backlog).** The fourth mode slot went to +BASS. A separate
  root-position, fifth-reinforced voicing mode was specced and shelved.
- **Voice-count meter (backlog).** A VU-style meter on the track buttons,
  display-only with a short decay. Shelved because those buttons are now mode
  selectors and Loop/Mute/Copy are wanted for P5.
- **DSP for tempo and phase (backlog).** The only route to an *automatic*
  tempo, and to phase from the host rather than from the Play press.
  `get_bpm()` and `get_beat_position()` exist on the C host API — the latter is
  derived from 24-PPQN clock and interpolated per audio block, so it is
  drift-free and bar-aligned, which is more than K8 can give. Movy does exactly
  this with a Rust engine and mirrors it into the UI via a status poll;
  TwinSampler proves an overtake module can ship a `dsp.so`.

  The risk is concentrated in one step: getting a value from the DSP back into
  JS. `host_module_get_param` exists but params normally flow UI→DSP, so the
  read path is unproven. Everything after that is ordinary work. Worth spiking
  just the read path before committing — and worth first finding out whether
  setting K8 once per project is actually annoying in practice.

  Guard `get_beat_position` for NULL: it was appended to the host API in
  2026-07 and older hosts won't have it.

- **Circle of 4ths page (backlog).** A companion to CIRCLE, cycling in fourths.
- **Silent step (backlog).** Step 16 was REST; now that it sounds, the
  chord-tone lane has no deliberate-silence button.
- **Bass sound modifier (backlog).** After latching a bass modifier, the bass
  note should sound with an added fifth from the scale.
- **Encoder LEDs (backlog).** Unconfirmed whether Move's encoders have
  addressable LEDs; the track buttons stand in for now.
- **UI ideas (backlog).** Borrow from
  [schwung-movy](https://github.com/DimaDake/schwung-movy) — Elektron-style knob
  pages, arc knobs, enum overlays. MIT licensed. Its parameter-page approach
  would suit K2–K5 better than the current badge strip.
- Row brightness variants — flat colour per family for now.
- **Shift-layer reroot (check 16) — backlogged for refinement.** Works, but the
  behaviour wants revisiting alongside the degree-6 modal-mixture idea below.
  Scope not yet pinned down — worth a decision before it's built.
- **Modal mixture.** Shift as a "move the root to scale degree 6, keep the same
  seven notes" lens. One rule that works in every mode, unlike relative-minor
  which needs a major/minor special case. Would let K3 offer all seven modes at
  no extra complexity.
- **P5 — page-change sustain.** Held notes should survive a page change and keep
  sounding until a pad on the new page is pressed. Needs a guard, since a page
  change followed by no press would sustain forever: 8-second timeout, plus
  Back and Capture as explicit panic. Built with hold/repeat/arp so the panic
  path is designed once rather than twice.
- `suspend_keeps_js` is off. It turns on at P5. `onResume()` is already wired.
