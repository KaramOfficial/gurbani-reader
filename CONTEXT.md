# Gurbani Reader — Full Context & Behaviour Specification

**Purpose of this document.** This is the complete handover context for the
Gurbani Reader: what it is, how every feature behaves, why it behaves that
way, and what was tried and settled along the way. It exists so the app can
be rebuilt on another platform or in another language (native Android/iOS,
another web stack, an AI-driven rebuild) **without redoing any of the
discovery, tuning, or correction cycles** recorded here. Where this document
and the code disagree, the code is the implementation truth — but the
behaviour described here is the *intended* truth, and most of it was arrived
at through explicit user direction, not engineering preference.

**Canonical home.** Repo `KaramOfficial/gurbani-reader`, live at
https://karamofficial.github.io/gurbani-reader/ — a single self-contained
`index.html` (no build step, no dependencies). This is the ONE canonical
Reader; an earlier Muse artifact copy was deliberately deleted so there is
never a second version to confuse. A separate experimental `gurbani-reader-v2`
exists but is **not** the product and is not being looked at; do not redirect
anyone to it.

**Owner's standing rules for this app.**
- Bani names are taken **exactly** as the trusted source names them (the
  Learn Shudh Gurbani app / GursevakDB), never coined, shortened, or
  extended. See the table in §3 — this rule is absolute, in UI, datasets,
  and filenames alike.
- Bani work is done carefully (ਧਿਆਨ ਨਾਲ): verify numbers (line counts,
  timestamps, alignments) before publishing anything.
- Minimal UI means *less text*, never dropping earlier deliberate design
  choices (collection subtitle, Gurmukhi digits, design tokens).
- The UI appears only when called — by tap, by gesture, by scroll. Nothing
  pops up on its own during reading or auto-scroll.
- The owner's phone is the final judge of any behaviour; automated checks
  are supporting evidence, not the verdict.

---

## 1. What the app is

A phone-first web reader for Sri Guru Granth Sahib Ji (all 1430 Angs) and
the 8 Nitnem Banis, with synchronised audio (Sehaj Paath by Bhagat Jaswant
Singh Ji) and word-level timestamps. You read the text; the audio follows
the text and the text follows the audio. Everything is one continuous
scroll — Angs and Banis stream into each other like one book.

## 2. Data sources (all public, no keys)

Base: `https://karamofficial.github.io/gurbani-checker/`

- **Bani text+timings:** `data/<folder>/<file>.json`, and matching `.mp3`.
  Folders/files: `japji-sahib/japji_sahib`, `jaap-sahib/jaap_sahib`,
  `tav-prasad-savaiye/tav_prasad_savaiye`, `chaupai-sahib/chaupai_sahib`,
  `anand-sahib/anand_sahib`, `rehraas-sahib/rehraas_sahib`,
  `kirtan-sohila/kirtan_sohila`, `sukhmani-sahib/sukhmani_sahib`.
- **Ang text+timings:** `chunks/chunk-<range>.json` (note: **no** `/data/`
  segment), ranges of 100 Angs, shaped `{ angs: { "<ang>": { lines,
  duration } } }`.
- **Ang audio:** `https://media.gursevak.com/media/Sehaj_Paath_Pagewise/Sehaj_Paath_16kbps/<ang zero-padded to 4>.mp3`.
- **Line shape:** `{ start, end, text, words: [{ word, start, end }] }` —
  seconds into that item's audio. `text` is the full line including
  punctuation; `words` carries the per-word timings used for taps, the
  follow-along, and deriving the current line.
- **Text source of truth:** the Gursevak vishraam text (as on
  readgurbani.com). Never substitute plain SGGS text without vishraam
  marks when the app version is asked for.

## 3. Content rules

**Source-exact Bani names** (use these strings verbatim everywhere):

| id | Name |
|---|---|
| japji | ਜਪੁਜੀ ਸਾਹਿਬ |
| jaap | ਜਾਪੁ ਸਾਹਿਬ |
| tav | ਤ੍ਵਪ੍ਰਸਾਦਿ ਸ੍ਵਯੇ ← complete name; "Sahib" does **not** attach |
| chaupai | ਚੌਪਈ ਸਾਹਿਬ |
| anand | ਅਨੰਦੁ ਸਾਹਿਬ |
| rehraas | ਰਹਰਾਸਿ ਸਾਹਿਬ |
| sohila | ਕੀਰਤਨ ਸੋਹਿਲਾ ← **no** "Sahib" |
| sukhmani | ਸੁਖਮਨੀ ਸਾਹਿਬ |

Where the source name carries "Sahib", always use it; never the bare name.
Collections are titled ਸ੍ਰੀ ਗੁਰੂ ਗ੍ਰੰਥ ਸਾਹਿਬ ਜੀ (Angs) and ਨਿਤਨੇਮ (Banis).

**Rendering.**
- Lines render from `line.text` tokens (verified 100% against readgurbani
  Sehaj Paath, Angs 162–168). Rendering from word lists alone loses the
  dandi (।) — the dandi is glued to the previous word and must render.
- Word tokens are tappable spans carrying their timings.
- **Vishraams (readgurbani system): the symbols are HIDDEN; the word is
  coloured.** `;` (large vishraam) `#A0728E`, `,` (jamki) `#C9A634`,
  `.` (thamki) `#D18D5B`. An earlier build highlighted the symbols
  themselves — that was a misreading and was corrected: hide symbols,
  colour words.
- Ang numbers are shown in **Gurmukhi digits** everywhere in the UI
  (੧੬੫, not 165). A helper converts digits.

## 4. Reading model — one continuous stream

- **Ang stream:** opening any Ang starts a stream; following Angs append
  below with a minimal separator (hairline + `ਅੰਗ ੧੬੬ · Ang 166`, no Ek
  Onkar artwork). Playback rolls into appended Angs seamlessly; tapping a
  word/line in any streamed Ang switches the audio to that Ang.
- **Bani chain stream:** ਜਪੁਜੀ ਸਾਹਿਬ → ਜਾਪੁ ਸਾਹਿਬ → ਤ੍ਵਪ੍ਰਸਾਦਿ ਸ੍ਵਯੇ →
  ਚੌਪਈ ਸਾਹਿਬ → ਅਨੰਦੁ ਸਾਹਿਬ → ਸੁਖਮਨੀ ਸਾਹਿਬ flow in one scroll the same way.
  ਰਹਰਾਸਿ ਸਾਹਿਬ and ਕੀਰਤਨ ਸੋਹਿਲਾ are standalone (not in the chain).
- **Windowing:** far sections collapse to measured-height spacers and
  rehydrate on approach (IntersectionObserver sentinels), so a 1430-Ang
  stream stays memory-safe. Section heights are measured and remembered.
- **Bidirectional:** the stream is infinite in BOTH directions. Landing
  on any Ang also loads the Angs before it (head sentinels mirror the
  tail ones), so the reader can scroll up across separators exactly as
  they scroll down; the same applies along the Bani chain (a previous
  link exists for every chain Bani except the first). Prepending inserts
  the sections above and compensates the scroll position by their exact
  height in the same gesture, so the pankti under the reader's eye never
  moves. New sections' lines join the same flat line list — every lookup
  goes through the per-item first-line maps, never DOM order — and the
  pruning window is unchanged, so memory stays bounded no matter how
  far the reader travels in either direction.
- **Prefetch:** when playback comes within ~45 seconds of an item's end,
  the next item's audio *and* text chunk start loading, so the switch
  plays like one connected stream.

## 5. Playback

- One shared `<audio>` element for everything; a second hidden element
  only preloads the next track.
- Rollover at track end is seamless: the next item's audio is already
  loaded; the switch must not flash the bars, restart the scroll, or lose
  any setting (see the speed lesson below).
- **Speed-hold lesson (hard-won):** browsers reset `audio.playbackRate`
  to 1 when a new track loads. Setting the rate from the UI control
  *before* `load()` is not enough — on an automatic Ang change the new
  track started at 1× while the UI still showed 1.25×. The rate is
  therefore held in one JS variable (the single source of truth) and
  **re-applied on `loadedmetadata` and on every `play` event**. Any
  rebuild must do the equivalent on its platform.
- Speeds offered: 0.75×, 1×, 1.25×, 1.5×, 2× — plus, only when
  **Advanced mode** is on (Menu → Reading, an Off/On seg, persisted
  as advanced-v1): 2.25×, 2.5×, 2.75×, 3×. Turning it off above 2×
  falls back to 2×; the cluster's speed pill cycles whatever ladder
  is currently offered. Skip ±5s buttons flank the
  transport. A seek bar reflects and controls position.
- Wake Lock is held while playing so the screen stays awake.
- Fullscreen: attempted once playback is flowing and auto-scroll is
  carrying the reading (not on the bare play tap); exiting is manual.

## 6. Auto-scroll — the tracker

- A `requestAnimationFrame` tracker keys on the **spoken word**, never
  on line change (a pankti can span 5–6 rows at large fonts; the
  sentence highlight still paints the **full pankti**, but the scroll
  never waits for it). It serves the **focus section**: a fixed band
  of the *screen* — not of the text — in the upper-centre of the
  viewport. The section is anchored to the screen; lines are anchored
  to the section.
  - **Size — a whole number of lines, auto-determined.** From the
    measured line height (which carries the font size) and a height
    budget that depends on the viewport's shape: portrait phone 34%
    of height, short landscape 46%, desktop (≥1024 wide, ≥700 tall)
    36% — the line count is `clamp(floor(budget / line-height), 2, 4)`.
    So the section holds 2, 3 or 4 lines depending on the device, the
    orientation, the font size and the layout (lines or paragraph —
    the measured line height is the row height in both).
  - **Position — upper-centre, fixed fractions.** Top edge at 26% of
    height (portrait), 18% (short landscape), 24% (desktop); bottom
    capped at height − 120px (clear of the player), floored at
    top + 1.2 line-heights.
  - **Motion — anticipatory.** The page is **still** while the spoken
    word *and the word after it* are both inside the section. The
    moment the spoken word is the **last word in the section**, the
    tracker glides (time-normalised easing, 130px/frame cap, settle
    ~6px), landing that word's top at the section's top — so the next
    words flow in beneath it and are already in view when their turn
    comes. The reader's eyes never leave the section, and no spoken
    word sinks toward the bottom of the screen waiting to be rescued.
    Recenter, the restore landing and the rotation re-anchor all land
    their pankti's first word at the same section top.
  - If the spoken word's element is not in the DOM (spacer-swapped
    section), the tracker falls back to legacy line behaviour for
    that frame.
- Rollover between Angs must not slam the new first line to the top —
  the tracker carries it to the anchor like any other line.
- **The manual hold (Maps model, §9 Recenter):** the moment the reader
  scrolls by hand (touchmove / wheel / scroll keys), auto-scroll is
  *suspended indefinitely* — there is no timer after which it yanks the
  page back. (The original 5-second manual window did exactly that and
  was rejected.) The hold releases only when: the reader scrolls back
  so the audio's pankti is within the anchor band again, taps Recenter,
  presses play, taps a word/line to play from there, or the item
  changes. ("Back in place" for the hold and for the Recenter pill
  means the spoken/focus word is inside the focus section.) The audio itself is never paused or moved by any of this.
- **Rotation is a non-event:** rotating the phone mid-playback must not
  chase-scroll, flash the pill, or change the bars' state. On resize
  the tracker stands down for a short settle window (~400ms), then the
  spoken word is placed at the new section's top in ONE instant jump
  and the tracker resumes; the focus section itself is re-derived
  from the new orientation's dimensions at the same moment (its cache
  keys on width as well as height, font and layout), so a landscape
  section and a portrait section are each sized for their own screen.
  Fullscreen is the browser's to keep and is never touched. (The pre-fix behaviour — the tracker easing across the whole
  distance at full speed after every rotation — read as a fast,
  wrong-feeling scroll.)

## 7. The focus pankti — one source of truth

The "current line" is **derived, never stored as UI state**: binary-search
the current item's line timestamps for the line whose `[start, end)`
holds `audio.currentTime`, restricted to the audio item's line range in
the stream (`currentLineEl()` in the code). Everything that needs "where
the audio is" — the tracker, the recenter pill, the saved place — uses
this derivation.

> Trap recorded for rebuilds: the follow-along highlight state
> (`state.curLine`) is **not** maintained when the highlight mode is Off
> (the default). Any feature keyed off it will silently never fire.
> Always derive from the audio clock + timestamps instead. This trap
> has bitten twice: the Recenter pill at first, and the saved place —
> `focusedLine()` read `curLine`, so with highlighting off every save
> fell back to the anchor line and refreshes landed behind the audio.

## 8. Persistence (device-local storage)

| Key | Holds |
|---|---|
| `gurbani-reader:last-v1` | The saved place (below) |
| `gurbani-reader:fontsize-v1` | Font size index |
| `gurbani-reader:focus-v1` | The global last focus (§9) |
| `gurbani-reader:speed-v1` | Playback speed |
| `gurbani-reader:advanced-v1` | Advanced mode (unlocks speeds to 3×): 1 / 0 |
| `gurbani-reader:hlmode-v1` | Follow-along mode: off / sentence / word |
| `gurbani-reader:autoscroll-v1` | Auto-scroll on/off |
| `gurbani-reader:nitnem-open` | Nitnem section expanded in switcher/Library |
| `gurbani-reader:paaths-v1` | Sehaj Paath trackers: the list, the current one, the Default's custom name |
| `gurbani-reader:ctx-<id>` | One tracker's snapshot world: `{ place, glf }` (`ctx-default` for the Default) |
| `gurbani-reader:paaths-open` | Sehaj Paath section expanded in switcher/Library |
| `gurbani-reader:appearance-v1` | Appearance: day / auto / night |
| `gurbani-reader:detour-v1` | Free (Default) reading's own place + focus — the standing slot of the not-in-a-Paath world |
| `gurbani-reader:mode-v1` | Which world the reader left in: `free` (Default reading) or `paath` |
| `gurbani-reader:sync-v1` | Cloud-sync meta (not itself synced): last synced state's stamp + key fingerprint |
| `gurbani-reader:apply-guard` (sessionStorage) | One-shot: the cloud blob just applied, re-written once after the apply reload |

Font sizes: `[20, 22, 24, 26, 29, 32, 36, 40, 44, 48]px`, default
index 2 (24px). New sizes are only ever **appended** — a saved index
must keep its meaning. Font size and speed were originally **never
saved** (reset every visit) — the owner caught this; every
user-facing setting must persist.

**The saved place** is written on scroll (debounced), on page hide, on
item switches, and on tap-to-play. One guard matters: saves stand down
during transient layouts — the ~2s after a load begins (state already
names the new item while the screen still shows the old one), during a
prepend compensation, and mid-prepend. A save sampled in those windows
reads the anchor line off the *previous* item's layout and writes a
place whose `fl` points at the wrong Ang; the next refresh then
bounces back to it. Render's own post-landing save is exempt (it forces)
because by then the new layout is the truth. Shape:
`{ mode, id, ang, y, t, ts, fl }` where `t` is the audio position and
`fl` is the **focused pankti**: `{ item, idx, start, src }` — the item,
the line's index within it, its audio start, and where the focus came
from: `src: "audio"` (playback was following) or `src: "view"` (the
reader was browsing by hand — then the pankti in the *main position*,
the line at the 35% anchor, is the one saved, not a stale spoken line).

**Restore algorithm (on open, in priority order):**
1. A `?ang=N` deep link wins for that landing only, and is stripped from
   the URL immediately (otherwise every refresh re-applies the link —
   this caused a real bug: saved 165, link 295, refresh bounced forever).
2. If `fl.src === "view"`, reopen **that item** and restore the whole
   **screen**: land at the saved pixel scroll position, so the pankti
   under the reader's eye returns to the exact spot it left (the
   anchor line alone sits several lines above a natural reading
   position — restoring only it landed him that many lines back).
   The pixel landing is verified against the focus line at the first
   settle pass and falls back to the line landing if the stream
   layout shifted (the item was mid-stream when saved). The audio
   parks at the pankti's start — unless the recording had already
   played **past** the viewed pankti, in which case it parks at its
   true saved position, never rewound (the global focus owns the
   audio's place now; Recenter bridges eyes and audio).
3. Otherwise reopen the audio item and **derive the pankti from the
   saved audio position** against the line timestamps (§7). This also
   repairs places saved before `fl` existed.
4. Fallback for ancient saves: pixel scroll offset + audio seconds.
5. **Land directly:** the scroll to the pankti happens synchronously as
   the content renders, so the page *opens* on the pankti — no
   open-at-the-top-then-jump. Later settle passes (≈450ms, ≈1200ms, and
   when webfonts finish) correct for layout shift, and the browser's own
   scroll restoration is disabled so it cannot fight the placement.
6. **Refresh adopts the landed pankti as the focus:** when a restore
   lands on the SAME item the global focus already belongs to (the
   ordinary refresh), the landed pankti *becomes* the global focus —
   the reader refreshed while reading there, so that pankti is now
   "the one". Without this the pill kept offering a return to the
   older audio pankti and sat over a perfectly good landing. A restore
   onto a DIFFERENT item (he had clicked away before refreshing) does
   not adopt: the focus stays where it was and the pill keeps offering
   the way back.
7. **The focus mark:** the landed pankti is marked with the app's soft
   "lite" wash (§9 tap model) — but HELD, not faded: it stays for as
   long as the app sits idle at that focus, so anyone picking the
   phone up sees "this was the one". It clears the moment reading
   actually starts: a hand-scroll gesture, a play, or a new tap.

## 9. UI elements & interaction rules

**Bars (top + player).** They hide on forward scroll and auto-hide ~3s
after playback starts; they return on a real gesture. During an audio
switch they must stay hidden — Android fires `pause` before the switch
code runs, so the switch is flagged *before* the pause handler can react,
and during auto-play a scroll recalls the bars only if a genuine gesture
happened within ~400ms. The reader's own programmatic scrolling must
never be mistaken for the reader's hand.

**Pager (in the top bar).** Previous/next item as **chevron-only ghost
buttons — no text labels**; at the ends they disable faded. The centre
label is serif, shows the current item (ਅੰਗ in Gurmukhi digits, or the
Bani name), carries a caret, and **opens the switcher popup** (not the
Library). While hand-browsing the stream, the centre follows the item
*on screen*; the audio item is untouched. **Swipe navigation was
explicitly rejected** — it fights the browser's back/forward gestures.

**Switcher popup (centre-label tap, or island tap).** Title "Go to Ang";
a current-item line ("ਹੁਣ · ਅੰਗ ੧੬੫"); a number input + "Read Ang"; and
**−1 / +1 steppers only** — in a stream you only ever cross one
separator at a time, so coarse jumps (−50/−10) were removed as
impossible-feeling; the typed input covers exact jumps. Below a hairline,
an **expandable Nitnem section** (serif ਨਿਤਨੇਮ, note "Nitnem Banis · 8",
chevron that rotates; expand state shared with the Library and
persisted) reveals the 8-Bani grid; tapping a Bani closes the popup and
loads it. The popup works from the *viewed* item while browsing, not the
audio item.

**Library / Menu sheet.** Two-step: opens compact (Display — text size
A−/A+ with live preview on the page behind; Reading — follow-along
switch; a Library row), and Library extends on the second step with a
back row. Inside Library, the same pattern as the popup: Ang switcher
first, Nitnem expandable below, then the Sehaj Paath expandable, and
the audio credit ("Sehaj Paath audio by Bhagat Jaswant Singh Ji.") as
the panel footer. The sheet title follows the panel (Menu / Library).

**The island.** While hand-browsing, a floating pill drops in at the top
when the item on screen changes, showing that item (ਅੰਗ ੧੬੬ / Bani name).
View tracking runs **per animation frame during any scrolling** and
reads the section at the 35% anchor, so fast flings keep the number
live; it keeps updating while changes continue and fades ~1.6s after
they stop. It appears **only for hand-driven browsing** — during
auto-scroll the pager's centre follows silently instead (the number
still updates as a separator passes the main section). Tapping the
island opens the switcher from the viewed item.

**The global last focus.** One record (`gurbani-reader:focus-v1`:
`{ mode, item, idx, start, t }`) holds THE last focus pankti, and it
follows the **audio**: written during playback, sealed on pause, and
set directly by tap-to-play. Browsing and navigation loads **never**
move it — clicking away to another Ang and back used to lose the
place entirely (fresh load, audio at 0); now the focus survives
refreshes, click-aways and scrolls alike. Per-item places were
considered and deliberately skipped — one global focus, always there.

**Recenter pill.** Label **"Recenter"** (English), with a ↑/↓ arrow
pointing toward the pankti (the arrow hides when the focus is in
another item). It floats at the bottom, just above the player bar
(its offset is measured from the player's actual height). It belongs
to the global last focus: it appears whenever that pankti is **not in
the main position** — scrolled away by hand (the manual hold of §6),
its stream section pruned away, or the reader sitting on another item
entirely — **except** while auto-scroll is actively following the
focus (the tracker owns the position then; an earlier clock-keyed
version popped up at every line change). It is evaluated on scroll,
on the audio clock, and **at every render** (the view tracker alone
only fires on viewed-item changes and once left it stale after a
navigation load). One exemption matters: while playback is running
with auto-scroll on and no manual hold, the tracker owns the position
and the pill NEVER shows — the check must not require the focus
record to name the current item, because at an Ang rollover the
record lags the switch by a few hundred milliseconds and requiring
the match flashed the pill at every Ang change (the focus save's
throttle likewise applies only within one item, so the record
follows the audio across a switch at the first save). Tapping it: if the pankti is rendered, glide back
so it sits at the anchor, release the hold, and **mark it** on
arrival; otherwise reopen its item through the restore machinery
(§8), landing on the exact pankti with the audio parked there, and
the landing marks it.

**Tap model — one model everywhere.** For words *and* whole sentences:
- **Single tap = touch feedback only:** a soft accent wash ("lite",
  ~10% accent with a small halo) that **fades away on its own after
  ~1s** if no second tap comes. It is feedback that the UI is alive,
  never a selection and never a seek.
- **Double tap (within ~300ms, same word/line) = play from there:**
  full highlight, audio seeks (switching item if the tap was in another
  streamed Ang/Bani), playback starts.
Single taps must never jump the audio — too sensitive on a phone.

**Catch-up flash.** A one-time line highlight when auto-scroll *arrives*
somewhere: first arrival after refresh, and each return after scrolling
away. It is only a marker — it must still be on when the line lands
(an earlier version timed it from the scroll *start* and it died
mid-glide on long returns), then it turns off once auto-scroll is in
motion again. Always on; there is deliberately **no setting** for it.

**Follow-along highlight.** A 3-way setting in Menu → Reading:
**Off (default) / Sentence / Word**, persisted. Off means no moving
highlight at all — the Reader never highlights on its own; moving
word/line highlight as a *primary* behaviour belongs to the sister
Checker app, not here. Tap highlights (above) clear once the word's /
line's end time passes.

**Player layout.** Speed pill left, transport centred, auto-scroll
toggle right as an icon-only round button whose ON state is an accent
**ring**, not a filled disc. Auto-scroll defaults ON and persists.

**The quick cluster (immersive interruption).** While audio plays
with auto-scroll on and the bars hidden, a tap ANYWHERE — on a word,
on a line, on empty space — raises only a small floating cluster
(a vertical column along the right edge, stacked up from the
bottom-right corner — top to bottom: play/pause, the speed pill,
the A− / A+ font pair, and the settings orb at the corner; the
owner's layout calls Nov 24) instead of the whole chrome; a second tap dismisses it, and
it fades on its own after ~5s. The cluster carries the three things
a listener reaches for: a play/pause button (mirrors the player's,
icons painted by the same function — but pausing here deliberately
does NOT call the bars, the owner's ruling Nov 24: the button simply
becomes a play button and the cluster stays, exempt from the
pause/play bars reflexes via a toggle timestamp; the 5s fade also
belongs to playback only, so a paused cluster never times out), a
speed pill showing the current speed
("1.5×") that cycles the player's speed options on tap — applied
through the select's own change handler, so it saves and re-rates
exactly like the menu control — and the original sliders orb, which
calls the full bars. It never appears together with the Recenter
pill (Recenter outranks it). Outside immersive playback, taps behave
as they always did (a tap calls the bars). The point is a cheap,
low-commitment interruption while reading is flowing: one tap to
*maybe* interrupt, one more to actually open the UI. Cook jury:
Day 7.5/10, Night 8/10 (fixes applied: cluster dropped to the
bottom edge off the text, light-scheme pill borders strengthened,
night pause glyph washed one step).

**Tap anywhere to dismiss.** When the bars ARE showing, a tap
anywhere outside the UI itself — on a pankti, on empty space, on the
book head — puts the bars away; tapping again brings them back. The
tap's own jobs still happen (lite feedback on a word), and
double-tap keeps meaning ONLY play-from-that-sentence — it never
opens the UI; its second tap calls the bars back via the play path,
as it always did.

**Triple-tap a word — normal-speed preview.** Three quick taps
on a word (counted at line level — during playback the page eases
with the speech, so a word can drift a few px between taps) play
that word and the next few (up to five words total, never crossing
into the next track) one notch clearer than the current speed,
under a floating pill: at 1× when listening faster ("Normal speed
preview · 1×"), at 0.75× when already at 1× ("Slow preview ·
0.75×"; 0.5× from 0.75×). When the segment ends,
the reader's own speed resumes from that same spot — hear a section
clearly without breaking the flow. The speed control and its stored
value are never touched: the preview sets the element's rate
directly, re-asserts it against the app's own play/metadata rate
resets, and applySpeed() restores the reader's rate at the end. The
third tap is counted ahead of the tap model and marked, so the
bars/orb handler ignores it entirely (the second tap still plays
from the word, as always — the preview then takes over from that
same word). A long-press was the original trigger; it fought the
browser's text selection, so the owner moved it to triple-tap.
Pausing, seeking out of the segment, or the track ending cancels
the preview (rate restored).

**Two worlds: a Sehaj Paath, or Default (free) reading.** The
reader is always in one of two worlds. Inside a Sehaj Paath, its
place + focus ARE the app's place + focus and everything tracks to
it. **Default reading** is the other world: just reading Gurbani,
not part of any Paath — it has its own persisted place + focus (the
detour-v1 slot, same shapes as the real ones), so neither world
ever disturbs the other, and `mode-v1` remembers which world the
reader left in; at boot that world resumes (an existing install
with no mode key boots into its Paath world, unchanged). Default
reading is entered three ways: a **fresh install** starts there; a
**Pankti Voice Search deep link** (?ang=N&line=L — the search index
and the Reader's chunks enumerate an Ang's lines identically,
verified line-for-line) lands there on that exact pankti as the
focus, centred at the anchor, lite-marked, audio parked at its
start, the link stripped from the address bar; and the **Paath
popup's Default row** (below). A plain ?ang=N link (no line) keeps
the older in-world behaviour. In Default reading, Recenter behaves
NORMALLY — it answers to the free world's own focus (seeded from
the link pankti when there is one, afterwards moved only by the
audio and tap-to-play, never by the view).

**The floating pair + Paath popup.** Recenter stays centred above
the player, flanked by two small icon buttons that show whenever
Recenter does (and when there is no focus yet), in BOTH worlds:
a **book icon** (left) and a **bookmark icon** (right). The book
opens the **Paath popup**: a "Default — just reading · not a
Paath" row (marked current in Default reading; choosing it leaves
the Paath world via a snapshot of the outgoing Paath, the
switchPaath trick in reverse) followed by every Sehaj Paath (the
Default Paath included) with its place label and focus preview;
choosing one switches to it. Default-the-mode appears ONLY in this
popup — never in the Library/switcher Paath lists. The bookmark
opens the same popup in **assign mode**: choosing a Paath makes
the pankti the reader is on (the focus if there is one, else the
pankti at the anchor — captured when the popup opens, since the
popup covers the anchor) that Paath's resume point, by rewriting
its snapshot (or the live place/focus when it is the current
Paath); the reader never moves, and a toast confirms. This is the
manual counterpart to a Paath's automatic tracking.

**Appearance — Day / Auto / Night.** A Display setting (seg in the
Menu, and a top-bar button beside Full screen that cycles Day →
Auto → Night, showing a sun / half-circle / moon for the current
mode, washed accent when a mode is forced). **Day** forces a
bright palette tuned for outdoors (near-white paper, deep ink,
stronger hairlines, vishraam colours deepened a step past the
settled light ones). **Auto** is the original behaviour — the
light palette, or the dark palette under the phone's dark scheme.
**Night** forces a least-light palette: true black background,
washed warm-gray text, every accent and the vishraam colours muted
in proportion, and even solid accent buttons (Sign in / Go) washed
to a soft fill — no surface left glowing. Deepened Nov 24 on the
owner's word (the first cut still read too bright): a Cook jury
over four candidates picked "warm ash" 9/10 — --ink #6e6759,
--ink-soft #59544a, --muted #47443c, --accent #826226 /
--accent-strong #997431, vishraams #66525e / #625a3d / #62513b
(the mauve capped at/below ink luminance — distinguishable by hue
only); one step darker (#625b4f) failed readability. The class lives on
<html>; a head script sets it from the stored key before first
paint, so a forced mode never flashes the wrong palette. Cook jury:
Night 8/10, Day 6.5/10 (fixes applied: the night sign-in slab
washed; day vishraams deepened).

**Sehaj Paath trackers.** A Sehaj Paath is one whole read of Sri Guru
Granth Sahib Ji — days to years — and a reader may keep several going
at once. Below the Nitnem expandable (in BOTH the Library and the
switcher popup, same pattern, shared persisted expand state) sits a
ਸਹਜ ਪਾਠ expandable whose header note names the current context
("Current · …"). Rows: the **Default** state (the untracked reading —
the app's original behaviour) plus one row per journey, seeded with
four ("Sehaj Paath ੧–੪", Gurmukhi numerals) and addable without limit
via "+ New Sehaj Paath". Each journey is a complete snapshot world —
its own saved place + its own global focus (`ctx-<id>`): switching
stashes the live records under the old context, installs the target's,
and lands through the ordinary restore machinery, so "continue" opens
the journey's Ang on its pankti, marked, audio parked. A journey with
no snapshot starts at ਅੰਗ ੧. Row sub-labels say the action: "Resume at
ਅੰਗ ੧੬੫ · 12% complete" for a journey in progress ("ਅੰਗ … · % complete"
for the current one), "Not started" otherwise; the Default row shows
just its Ang. Every row — Default included — carries a pencil to
rename it inline. Under the sub-label, each row previews its focus
pankti — the line's first few Gurmukhi words plus " …" — stored on
the focus record itself, so snapshots carry it (rows whose line text
isn't loaded derive it when the item is in memory, else no preview).
Rename reads the input in the surface actually used: the list exists
twice (Library + popup) and both render an input while editing, and
reading the first in DOM order silently saved the untouched
surface's old value — the rename "reverted" (owner-reported, fixed
Nov 23). Nothing in the reading/playback code knows trackers
exist; the live records always belong to the current context.

**Paragraph mode (LIVE Nov 24, Cook jury 8/10).** Print flow as on
readgurbani.com: panktis run inline and wrap at the column edge. Paragraph
breaks: before a header pankti (ਮਹਲਾ line ≤44 chars, class `hdr` — bold
accent inline, or its own centred line when the header setting says Own
line), after a ਰਹਾਉ pankti, and after 5 panktis at the next verse-end;
never across a section/Ang. Word + ॥ + verse number are one unbreakable
unit (`joinParts` glues punctuation-only parts with NBSP in para mode).
Auto-scroll and Recenter work exactly as in lines mode — the focus
section (§6) measures its line height from the wrapped rows. Menu → Display: **Layout**
(Lines/Paragraphs) + **Paragraph headers** (Inline/Own line). Layout is
stored PER SIZE CLASS (`layout-wide-v1` / `layout-narrow-v1`; wide =
≥761px wide and ≥500px tall), defaults Paragraphs-wide / Lines-narrow, is
kept OUT of sync, and re-applies on rotation/resize class flips; switching
rebuilds mounted sections and re-anchors to the focus pankti. Header style
is `parahead-v1` (synced normally). Spec: ~/workspace/reader-design/paragraph-mode-spec.md.

**Large screens (≥761px, Cook jury Nov 24).** ONE measure — 700px — for the
reading column, top bar, pager, player card and sheet content (they were
700/760/680 stacked, which read as a phone layout centred by accident).
First run on a ≥1280px screen with no saved font size starts one step up
(26px); a saved choice always wins. At ≥1024px the Menu sheet is a centred
700px panel (side hairlines, top-drop kept) instead of a full-width curtain,
and the quick cluster anchors to the reading column
(`right: calc(50% - 412px)` — column edge + 16px gap) instead of the
viewport edge. Phone CSS untouched.

**Landscape phones.** In landscape (short height), the chrome merges
to one row each: the top bar drops the wordmark to the ੴ mark alone so
the pager (chevrons + current item) sits centred as one unit, the
collection subtitle is dropped, and the player drops its now-playing
block so the seek bar gets the width (~288px vs ~170px) — total chrome
≈117px of a 412px height (was ~196px, nearly half the screen), with
44px minimum touch targets. Rotating changes nothing else: bars
hidden stays hidden, bars shown stays shown, fullscreen persists.

**Account & cloud sync (optional).** The Menu carries a compact
Account block: "Sign in with Google", or the signed-in email + Sign
out. It uses the same Firebase project as the Nitnem Tracker
(`nitnem-tracker`, compat SDK vendored under `firebasejs/10.12.5/`)
and mirrors the Tracker's pattern: every `gurbani-reader:*` key
(place, focus, settings, Paaths + snapshots — the sync meta itself
excluded) is serialised into the `readerJson` field of the user's
own doc in the Tracker's `progress` collection (merge write; the
two apps never touch each other's fields, so the rules that let
the Tracker sync let this sync). Newer state wins wholesale by
client stamp; applying cloud state reloads once, with a one-shot
sessionStorage guard re-writing the applied keys after the reload
(the dying page's own place saves can otherwise clobber the synced
place in the reload window). No push happens before the sign-in's
initial cloud check settles, and pushes fire only when the key set
actually changed (2.5s poll, debounced, plus on pagehide) — an idle
signed-in tab never overwrites another device. Last-writer-wins is
the known limit, same as the Tracker. Signed out, SDK blocked, or
offline: local-only, exactly as before — sign-in is invisible
unless wanted.

## 10. Design language

Settled in a dedicated design pass (the "Cook" jury process — benchmark
first, juror-scored, fixes applied; the switcher/island pass scored
8/10). Tokens: paper background, hairline borders, one soft shadow
token, serif (Noto Serif Gurmukhi) for titles/labels, accent colour used
sparingly (rings, arrows, the lite wash at 10%). Pills are fully rounded.
Disabled states fade rather than disappear. The top bar carries the
brand lockup (badged ੴ + "Gurbani Reader", never truncated) and the
collection subtitle (ਸ੍ਰੀ ਗੁਰੂ ਗ੍ਰੰਥ ਸਾਹਿਬ ਜੀ / ਨਿਤਨੇਮ). One switcher
concept, one name ("Go to Ang"), on both the popup and the Library.
When simplifying, keep these decisions — minimal is a text diet, not a
design reset.

**Mool Mantar (Nov 24).** The opening pankti of Japji Sahib and of
Ang 1 (ੴ ਸਤਿਨਾਮੁ … ਗੁਰਪ੍ਰਸਾਦਿ ॥ — detected by content in all three line
builders; it occurs nowhere else in this form) always stands on **its
own ceremonial line**: a centred block at 1.12× the reader font size
with generous air above and below, and ੴ — still the line's first
word, with its own timing/tap/highlight — at 1.45× in
`--accent-strong`. **No frame and no special background** (a framed
version shipped first and the owner rejected it the same day): the
special place comes from standing alone, never inline with the rest —
in paragraph/print flow too, where it is a forced block that breaks
the flow before and after.

## 11. History

This document describes the solution **in its final state only** — it
is a solution architecture document, not a changelog. When a design
changes, the relevant section above is rewritten to the new final
state; nothing historical accumulates here. The full record of how
the design evolved is the git log of this repository, whose commit
messages are written to be read as the changelog.

## 12. Rebuild checklist (any platform)

1. Data first: Bani JSON/MP3 + Ang chunks + Ang MP3 URL patterns (§2);
   render from `line.text`; dandi glued; vishraam words coloured,
   symbols hidden (§3).
2. Source-exact Bani names everywhere; Gurmukhi digits for Ang numbers.
3. One audio engine; seamless rollover; prefetch next item near the
   end; **hold the playback rate across track loads** (§5).
4. Continuous stream with windowing for both Angs and the Bani chain
   (§4); Rehraas/Sohila standalone.
5. Auto-scroll: the focus section — screen-anchored in the
   upper-centre, sized in whole lines (2–4) from line height and
   viewport shape, anticipatory glides that begin when the spoken
   word is the section's last word — and the **manual-hold
   suspension**, never a timed yank-back (§6).
6. Derive the current pankti from clock + timestamps for every feature
   that needs it; never key features off highlight state (§7).
7. Persist **every** setting and the place record with `fl.src`
   semantics; restore by derivation with direct landing + landing
   highlight (§8).
8. UI rules: bars only on call; chevron-only pager; switcher with
   −1/+1 steppers and the expandable Nitnem section; island for
   hand-browsing only; Recenter only under manual hold; one tap model
   (single = fading feedback, double = play) (§9).
9. Keep the design tokens and the Cook-pass decisions (§10) — and get
   the owner's on-device verdict before calling any of it done.
