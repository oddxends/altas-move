# ATLAS — P5.1

ChordMaps-style chord map for Ableton Move. Schwung overtake module, JS only.

**Pad modes and a second bass lane.** The track buttons switch what a pad does;
steps 9–16 play chord tones that follow the harmony rather than the key. Chords
voice-lead between presses, K4 spreads and K5 inverts. Every pad on every page
sounds; PALETTE stays a diagnostic.

Measured on hardware: ~196 ticks/sec, ~28 ms full-grid sweep.

## Build and deploy

```bash
bash scripts/build.sh      # tarball is suffixed from the PHASE file

scp dist/atlas-module-P5.1.tar.gz ableton@move.local:/data/UserData/
ssh ableton@move.local 'tar -xzf /data/UserData/atlas-module-P5.1.tar.gz \
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
| Track 1–4 | pad mode: CHORD / ARP / +BASS / STRUM |
| Track 2 again | cycles arp direction: up / down / up-down / random |
| Loop | HOLD — latch the sounding chord |
| Copy | REPEAT — re-strike on the beat |
| K1 | tempo, 40–240 BPM — 5 detents per step, 1 detent reports it |
| K2 | key (C–B) — 6 detents per step |
| K3 | scale (major / minor) — 8 detents per step |
| K4 | spread: close / drop-2 / drop-3 / drop-4 — 6 detents |
| K5 | inversion: 0 = auto voice-lead, 1–4 manual — 6 detents |
| K6 | strum spacing, 40–500 ms — 6 detents |
| K7 | arp gate, 10–100% of the step — 6 detents |
| Up / Down | chord octave ±1 (clamped ±4) |
| Shift + Up / Down | bass octave ±1 (clamped ±4) |
| Shift (tap) | latch / unlatch the relative-minor layer |
| Shift (hold) | modifier — a combo suppresses the latch on release |
| Left / Right | page back / forward |
| Jog turn | browse the page rail — name blinks until committed |
| Jog click | commit the browse — or, if idle, the page's own action (BUILDER: root latch) |
| Capture | full grid repaint, and panic on a sustained chord |
| Play | start / stop; pulses on the beat. Also passed through to Move |
| Menu (hold) + steps 9–16 | rhythm rate: 1/1 … 1/16T |
| Back | all notes off, exit |
| Record | unused — reaches us as CC 86, no function assigned |
| Shift + Vol + Jog click | host escape, always works |

Steps **1–8** play degrees of the *key*; steps **9–16** play tones of the *last
chord played*. That difference is the point — the second lane stays right as
you move through a progression, which the first structurally cannot do.

### Encoders

| | Assigned | Notes |
|---|---|---|
| K1 (CC 71) | tempo | 5 detents |
| K2 (CC 72) | key | 6 detents |
| K3 (CC 73) | scale | 8 detents — the heaviest change, so the stiffest |
| K4 (CC 74) | spread | 6 detents |
| K5 (CC 75) | invert | 6 detents |
| K6 (CC 76) | strum spacing | 6 detents, 40–500 ms |
| K7 (CC 77) | arp gate | 6 detents, 10–100% |
| K8 (CC 78) | **free** | inert — turning it does nothing |

**Only K8 is free.** An unassigned knob sends CCs that are read and discarded,
so it does nothing rather than something surprising.

Tempo, strum spacing and gate appear nowhere on screen, so a **single detent on
K1, K6 or K7 reports the current value without changing it** — that's how you
read them. Five or six detents change them.

**Strum spacing is geometric, not linear.** The musically useful part of
40–500 ms is the fast end; a linear sweep would spend most of the knob between
300 and 500 ms where the differences are barely audible. Sixteen steps, each
about 18% longer than the last, so the low end has fine resolution.

**The gate is a fraction of the step, not a fixed time**, so it stays
proportional as you change rate or tempo. Below about 30% notes clip short and
staccato; at 100% they run into each other.

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

## Rhythm strip

Hold **Menu**: steps 9–16 become eight rates from 1/1 to 1/16T.

The five straight divisions ramp dim to bright — slate, blue, teal, green,
brightest at 1/16. All three triplets share one pink, because triplet-versus-
straight is a category you pick rather than a point on the speed ramp; giving
them their own colour also lets the ramp read monotonically instead of being
interrupted every other step.

The selected step goes white, so no rate may use white. The lower lane goes dark and inert while Menu is
held, so a Menu press can't leak a bass note into what you were playing.

Nothing consumes the rate yet — ARP and REPEAT arrive in P5. The strip and the
clock have to exist before there is anything to sync.

Menu was chosen over Shift because Shift already latches the minor layer,
shifts the bass octave, and suppresses on combo. A third job on it would be one
too many.

## Time

Everything here runs off the transport clock subdivided by the rhythm strip, so
it stays phase-aligned rather than free-running alongside the beat. Nothing
sounds until Play is pressed.

**ARP** takes the held notes one at a time. In ARP mode the pads contribute
notes silently — the arp decides when they sound, otherwise every note would
double. Up-down skips the turnaround notes so it reads as a cycle rather than
stuttering at each end. Note length is set by the gate on K7, defaulting to
80% of the step so successive notes articulate instead of blurring.

**HOLD** (Loop) latches whatever is sounding and keeps it after you release the
pads. It re-claims the pitches through the ref-counted registry, which is
exactly the case that registry was built for. Press again to release.

**STRUM** spacing is on K6, from 40 to 500 ms.

**REPEAT** (Copy) re-strikes the whole chord on each step — distinct from the
arp, which takes one note at a time. The two can run together.

**Changing rate re-anchors the step clock.** Without that, switching from a
slow rate to a fast one leaves the next step scheduled up to a whole slow-step
away and the arp appears to stall for a bar.

## Page sustain

A held chord now survives a page change and keeps sounding, so a progression
doesn't gap when you move pages. Three things end it: playing something new, an
eight-second timeout, or an explicit panic on **Back** or **Capture**.

The timeout matters — without it, changing page and walking away would sustain
indefinitely.

## Pad modes

| Button | Mode | A pad press |
|---|---|---|
| Track 1 | **CHORD** | plays the chord — the default |
| Track 2 | **ARP** | notes taken one at a time, on the beat. Press again to cycle direction |
| Track 3 | **+BASS** | the chord plus its own root two octaves down |
| Track 4 | **STRUM** | the same notes fired low to high, spacing on K6 |

Exclusive; the lit button is the active mode, also shown in the footer. Modes
change only what the *pads* do — both step lanes behave identically throughout.

+BASS takes its bass note from the chord's **root**, not the lowest sounding
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

PED holds the key root; DRN holds the chord root. One anchor ignores the
harmony, the other follows it. R+5 plays the key root with a fifth above it.

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

Three pages opt out of voicing entirely, because on them the shape *is* the
content: POWER (a power chord is defined by having no third), INTERVALS (the
row selects the interval width, so inverting a third into a sixth would
silently relabel the page) and TENTHS (every row already is a voicing). They
still get the register clamp.

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

## What P4 leaves open

- **P6 — shipping.** On-device help, settings that persist across loads, the
  USER page loader for page C, and CI so releases don't depend on a script run
  by hand.
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
