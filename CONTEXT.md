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
  moves. The compensation is measured as the displacement of the first
  stream element itself (its rect before vs after insertion — margins
  and separators included), never as a sum of offsetHeights, and the
  document sets `overflow-anchor: none`: the browser's own scroll
  anchoring must not also adjust, or the two compensations fight and
  every landing drifts (the pre-fix landing visibly re-anchored itself
  two or three times before settling; with this pair in place the
  restore landing's verification re-lands are no-ops). New sections' lines join the same flat line list — every lookup
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
  any setting (see the speed lesson below). If the next item is still
  being appended when the track ends, the rollover's play intent is
  recorded and the in-flight append honours it on completion — the
  boundary never stalls in silence.
- A failed audio load is never silent: an `error` on the audio element
  (or a `NotSupportedError` play rejection) resets the play icon and
  shows a toast ("Audio couldn't load — check your connection, then
  tap play").
- **Speed-hold lesson (hard-won):** browsers reset `audio.playbackRate`
  to 1 when a new track loads. Setting the rate from the UI control
  *before* `load()` is not enough — on an automatic Ang change the new
  track started at 1× while the UI still showed 1.25×. The rate is
  therefore held in one JS variable (the single source of truth) and
  **re-applied on `loadedmetadata` and on every `play` event**. Any
  rebuild must do the equivalent on its platform.
- Speeds offered: 0.75×, 1×, 1.25×, 1.5×, 1.75×, 2× — plus, only when
  **Advanced mode** is on (Menu → Reading, an Off/On seg, persisted
  as advanced-v1): 2.25×, 2.5×, 2.75×, 3×, 3.25×, 3.5×, 3.75×, 4×.
  Turning it off above 2×
  falls back to 2×; the cluster's speed pill cycles whatever ladder
  is currently offered. Skip ±5s buttons flank the
  transport. A seek bar reflects and controls position.
- **Jump history — Undo / Redo** (owner, Nov 25): the player carries
  Undo and Redo buttons (and Ctrl/⌘+Z, Ctrl/⌘+Shift+Z, Ctrl+Y). The
  model is the classic two-stack editor undo, and its entries are
  **lines**: a deliberate jump — double-tap a word or line to play
  from there, a seek-bar jump, or landing on an Ang (in any mode)
  — first pushes the line being left (item, line index, its start
  second, playing or paused) onto the undo stack and clears the
  redo stack; Undo and Redo walk the stacks, restoring through the
  ordinary switch/seek machinery, so the audio re-seeks to that
  line and the head lands on it (a restore writes the focus record
  for the restored line explicitly — a paused seek saves nothing on
  its own). Three jumps are kept **per reading context** (one stack
  pair per Paath, one for Default, one per Bani), on-device and in
  memory only — never synced — so Undo inside a Paath can never
  drag in a position from free reading or a Bani. Follow-along
  (continuous playback) never enters it and never clears the redo
  stack; the ±5s nudges never enter it either.
- Wake Lock is held while playing so the screen stays awake.
- Fullscreen: attempted once playback is flowing and auto-scroll is
  carrying the reading (not on the bare play tap); exiting is manual.

## 6. Auto-scroll — the tracker

- A `requestAnimationFrame` tracker drives the camera under **Model
  C — sentence-first, row-safety second** — one model for every font,
  layout and highlight mode (the camera grammar never changes with
  the highlight granularity). It serves the **focus section**: a
  fixed band of the *screen* — not of the text — in the upper-centre
  of the viewport. The section is anchored to the screen; lines are
  anchored to the section.
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
  - **Motion — three glides, in strict priority.** All glides use
    the same time-normalised easing (130px/frame cap, settle ~6px)
    toward a fixed absolute target; at most one glide runs at a time.
    - **G1, the sentence glide.** When the spoken pankti changes, the
      page glides once, landing the new pankti's first row at the
      section's top — in the same frame as the sentence-highlight
      change. Whenever a pankti fits inside the section this is the
      ONLY motion: the page is still for the whole pankti, exactly
      the pre-redesign behaviour the reader called smooth and in
      sync. In paragraph layout, if the new pankti's entry row is
      shared with the previous pankti, G1 is cancelled for that
      pankti (a shared row cannot be isolated) and only G2 may move.
      "Previous" means the last word of the immediately preceding
      pankti — after a seek, a distant word whose row coincidentally
      aligns does not cancel G1. A seek also resets G2's row key, so
      a re-entered row can glide again. The section's cache key
      includes the measured line height itself, so a webfont landing
      (or a paragraph-header style flip) re-derives the section
      instead of keeping fallback-font geometry all session.
    - **G2, the safety glide.** Within a pankti, only when the spoken
      word's own row has actually reached the section's bottom edge
      does the page move — the minimum shift that makes that row the
      section's last row (one row height for uniform rows). At most
      once per (pankti, row), never while a glide is running, never
      evaluated on a pankti-change onset. This is the whole of the
      large-font provision: a 5–6-row pankti gets its G1 plus one-row
      safety shifts, instead of top-resets mid-sentence. (A word left
      fully above the section by a backward seek within the pankti is
      landed at the section top by the same rule's recovery branch.)
    - **G0, the reset.** On an item load/switch the current pankti is
      accounted for without a sentence glide; a spoken row outside
      the section is landed at the section top. Seeks ride G1/G2.
    - The camera never looks ahead to the next word and never resets
      position mid-pankti — the two faults of the word-anticipatory
      model this replaced (it fired while the spoken word was still
      comfortably inside, because the *next* word was outside, and
      landed the spoken word at the section top: two ~2-row jumps
      inside one long pankti while the sentence highlight had not
      changed).
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
  A resize that flips the layout class rebuilds the stream in place:
  a paused or browsing reader keeps the pankti under his eye at the
  same screen offset — only a playing, following reader is
  re-anchored to the spoken line.
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

**The head (owner, Nov 25).** What a Paath remembers is its last
**focus line** — the head — and nothing else: scroll position is
never part of a Paath's stored or synced state (the device-local
session place in §8 is a separate, per-device convenience and never
enters a Paath record). There is exactly one head per Paath, and
the complete list of what can move it is short by design: audio
follow-along (bounded by speech — 3× at the fastest), a seek on the
audio bar, a double-tap on a line or word, and assign-to-Paath.
Hand scrolling and browsing never move a Paath's head; pressing
Play resumes at the head (playing from where you are *looking* is
what double-tap is for). Because the fastest mover is a voice,
head changes can never burst — sync traffic is a trickle by
construction, not by throttling heroics. **Landing:** a deliberate
navigation to an Ang — the steppers, the pill chevrons, the Ang
switcher, the in-stream end-of-Ang buttons — lands there, **in
every mode**: the head resets to that Ang's first line at once,
the head being left goes onto the jump history's undo stack (§5)
so Undo always brings it back, and the view is placed so the
first line already sits in the focus zone — pressing Play starts
exactly there, with no settling scroll. The one exception:
landing back on the Ang that already holds the head changes
nothing. A *targeted* arrival is not a landing: a search result
or a voice-search link sets the head to its own line (the detour
seed) and is never reset to line 1.

## 8. Persistence (device-local storage)

| Key | Holds |
|---|---|
| `gurbani-reader:last-v1` | The saved place (below) |
| `gurbani-reader:fontsize-v1` | Font size index |
| `gurbani-reader:focus-v1` | The global last focus (§9) |
| `gurbani-reader:speed-v1` | Playback speed |
| `gurbani-reader:advanced-v1` | Advanced mode (unlocks speeds to 4×): 1 / 0 |
| `gurbani-reader:hlmode-v1` | Follow-along mode: off / sentence / word |
| `gurbani-reader:autoscroll-v1` | Auto-scroll on/off |
| `gurbani-reader:nitnem-open` | Nitnem section expanded in switcher/Library |
| `gurbani-reader:paaths-v1` | Sehaj Paath trackers: the list + the current one's id (always a real tracker — there is no "Default" Paath) |
| `gurbani-reader:ctx-<id>` | One tracker's snapshot: `{ glf }` — the Paath's head (its last focus line) and nothing else; Ang state only, never a Bani's. A Paath never stores a scroll place (owner, Nov 25) |
| `gurbani-reader:paaths-open` | Sehaj Paath section expanded in switcher/Library |
| `gurbani-reader:appearance-v1` | Appearance: day / auto / night |
| `gurbani-reader:detour-v1` | Free (Default) reading's own place + focus — the standing slot of the not-in-a-Paath world |
| `gurbani-reader:mode-v1` | Which world the reader left in: `free` (Default reading) or `paath` |
| `gurbani-reader:sync-v1` | Cloud-sync meta (never itself synced): this device's writer id (`cid`), the account the sync state belongs to (`uid`), and per record only the exact JSON last sent or applied (`sent`) + the newest server stamp applied (`seen`) |

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
   A plain Ang link (no `line`) lands in the world the reader lives in:
   in Default free reading the visit saves to the free slot, never into
   the Sehaj Paath world's records. If the remembered world is free
   reading but its slot holds no usable place, the app still boots into
   free reading (at Ang 1) — never into the Paath place with the mode
   key disagreeing underneath.
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
   phone up sees "this was the one". It clears the moment anything
   else happens: a hand-scroll gesture, any press/tap elsewhere, a
   play, follow-along moving on to another pankti, or any glide away
   from the marked line. It never lingers behind playback.

## 9. UI elements & interaction rules

**Bars (top + player).** They hide on forward scroll and auto-hide ~3s
after playback starts; they return on a real gesture. During an audio
switch they must stay hidden — Android fires `pause` before the switch
code runs, so the switch is flagged *before* the pause handler can react,
and during auto-play a scroll recalls the bars only if a genuine gesture
happened within ~400ms. The reader's own programmatic scrolling must
never be mistaken for the reader's hand.

**Portrait strip (owner, Nov 26).** In portrait the top strip
carries only the original three buttons — **Menu, Search, Full
screen**, in that order: no Appearance button (it lives in the
Menu), and no pager — the Ang lives in the pill, whose chevrons
travel with it in portrait too. A crowded strip is how a Search
tap could land on the Ang switcher. Landscape earns extra
buttons per size tier, only where they fit (the rails, §10).
The Ang unit's home follows the chrome: at the very top when
the bars are hidden (full screen), tucked just below the top
bar while the bar is showing — it glides between the two. And
if the unit is already up when a tap calls the bars, it simply
disappears: the bar carries its own Ang switcher, so the unit
hands over instead of duplicating it (a fresh tap with no unit
up still flashes it below the bar). That handover belongs to
the top-bar layouts; in the side-bar landscapes the unit stays.

**Pager (in the top bar).** In the side-bar landscapes the bar
carries no pager at all — the pill unit is the Ang switcher
there, chevrons included, and it simply stays when the bars
show. Previous/next item as **chevron-only ghost
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
version popped up at every line change). It shows **whether the
audio is playing or not**: a paused reader who scrolls away from
the focus pankti gets the pill exactly as a playing one does. It
is therefore evaluated **on every scroll frame** and **at every
render**, plus on pause and on the playing progress interval —
never keyed only to the audio clock or the view tracker: both of
those stop mattering when paused (the interval and `timeupdate`
do not run, and the view tracker only fires on a viewed-item
change), which once hid the pill for a paused reader scrolling
away inside one Ang. One exemption matters: while playback is running
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
  streamed Ang/Bani), playback starts. The double-tap target is the
  **line**, not the exact word element (the triple-tap preview counts
  the same way): at big fonts the page drifts between taps and the
  second tap lands on a neighbouring word — it still plays, from the
  word actually tapped. The highlight is painted after any item switch
  the tap triggers, so cross-item taps keep their feedback too.
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
it fades on its own after ~5s. The Ang unit rises with the
cluster on that tap and shares its lifetime — it stays while
the cluster stays and leaves when the cluster leaves, instead
of flashing for 1.6s and dying long before the buttons do. The cluster carries the three things
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
the preview (rate restored). Position is NOT preview-temporary:
the preview starts exactly like a double-tap (same switch, seek and
focus behaviour), so the saved place/focus follow it — only the
rate is temporary.

**Two worlds: a Sehaj Paath, or Default (free) reading.** The
reader is always in one of two worlds. Inside a Sehaj Paath, its
place + focus ARE the app's place + focus and everything tracks to
it. **Default reading** is the other world: just reading Gurbani,
not part of any Paath — it has its own persisted place + focus (the
detour-v1 slot, same shapes as the real ones), so neither world
ever disturbs the other, and `mode-v1` remembers which world the
reader left in; at boot that world resumes (an existing install
with no mode key boots into its Paath world, unchanged). Default
reading is entered four ways: a **fresh install** starts there;
**opening any Bani** enters it (a Bani is Nitnem, never Sehaj Paath —
the outgoing Paath is snapshotted, Ang state only, and the Bani reads
and resumes in the free world); a
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
switchPaath trick in reverse) followed by every Sehaj Paath with
its place label and focus preview; choosing one switches to it.
The same Default-mode row also heads the Library/switcher Paath
lists — a plain mode line (no tracker dot, no pencil, separated by
a hairline), navigation only: nothing can be saved to Default. The
bookmark opens the same popup in **assign mode** (real Paaths only
— no Default row; the header keeps the current-context line and
adds the instruction under it): choosing a Paath makes
the pankti the reader is on (the focus if there is one, else the
pankti at the anchor — captured when the popup opens, since the
popup covers the anchor) that Paath's resume point, by rewriting
its snapshot (or the live place/focus when it is the current
Paath); the reader never moves, and a toast confirms. This is the
manual counterpart to a Paath's automatic tracking. Assign is an
Ang-only verb: in Bani mode the bookmark button is hidden, and the
assign function itself refuses a Bani record — a Paath snapshot is
only ever written from Ang state (snapshot writers substitute the
snapshot's previous Ang state when the live record is a Bani's).
While free reading is live, a Paath's snapshot IS its state: nothing
re-snapshots it from the live records (they belong to the free
world), so an assigned place survives the reader switching to
another Paath first. Picking the current Paath from free reading
installs its snapshot as the live records and resumes from it (with
no snapshot, the live place key is already its place).

**Appearance — Day / Auto / Night.** Controlled only by the top-bar
button beside Full screen, which cycles Day →
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
("Current · …", or the Default-mode line in free reading). Rows: a
**Default** mode line first (just reading — not a journey, no pencil,
never counted among the Paaths), then one row per journey, seeded with
four ("Sehaj Paath ੧–੪", Gurmukhi numerals; the first is the current
one) and addable without limit via "+ New Sehaj Paath". There is no
"Default" tracker: older builds kept a nameless one (id null, snapshot
at `ctx-default`, display name in `defName`) — a one-time migration
folds its journey into Sehaj Paath ੧ (or a new tracker if ੧ already has
one), drops `defName`, and scrubs any Bani focus (and any stored
place) out of every snapshot. The fold never renames a Paath: a renamed ੧ keeps its name
whether or not it has started (with a stored `defName` to honour and
a renamed ੧, the journey goes to a new tracker carrying that name). Each journey's whole stored world is its
**head** — its last focus line (`ctx-<id>` = `{ glf }`): switching
snapshots the old context's head, installs the target's, and derives
the landing place from that head (`placeFromFocus`), so "continue"
opens the journey's Ang exactly on its pankti, marked, audio parked.
A journey with no snapshot starts at ਅੰਗ ੧. Row sub-labels read the
head and nothing else: "Resume at ਅੰਗ ੧੬੫ · 12% complete" for a journey
in progress ("ਅੰਗ … · % complete" for the current one), "Not started"
otherwise (a snapshot whose head is a Bani's renders "Not started"
too — a Paath label only ever names an Ang). Every journey row carries a pencil to rename it inline;
renames are validated — "Default" (the mode's name), a duplicate of
another Paath's name, an empty name, and names over 40 characters are
refused with a toast. Duplicates are judged on the name as it will
exist after the next load (Latin-digit "Sehaj Paath 1" normalises to
"Sehaj Paath ੧") and case-insensitively. The rename input is focused
in the surface where the pencil was tapped (both lists render one).
In the lists exactly one row is ever marked current: in free reading
that is the Default row, not the underlying tracker (the popup marks
its tracker rows the same way, and its Default row is the same plain
mode line — no dot, no card). Transient messages (refusals,
confirmations, notes) appear in their own toast that wraps, caps its
width, and renders above sheets and scrims; the triple-tap preview
keeps its own single-line pill, and the two never dismiss each
other. Under the sub-label, each row previews its focus
pankti — the line's first few Gurmukhi words plus " …", the dandi
(॥) kept wherever it falls — stored on
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
(Lines/Paragraphs/Print) + **Paragraph headers** (Inline/Own line; the row
shows only in Paragraphs). **Print** is the third layout: the whole Ang as
one continuous page, readgurbani.com print-view style — panktis flow inline
line after line with NO paragraph breaks anywhere (headers flow inline
too); only the Mool Mantar keeps its own centred line, as in every layout.
Layout is
stored PER SIZE CLASS (`layout-wide-v1` / `layout-narrow-v1`; wide =
≥761px wide and ≥500px tall), defaults Paragraphs-wide / Lines-narrow, is
kept OUT of sync, and re-applies on rotation/resize class flips; switching
rebuilds mounted sections and re-anchors to the focus pankti. Header style
is `parahead-v1` (synced normally). Spec: ~/workspace/reader-design/paragraph-mode-spec.md.

**Large screens — designed per size, not the phone layout scaled.**
One measure for the reading column, top bar, pager and player card:
700px at ≥761px, **800px at ≥1024px, 840px at ≥1600px** (at desktop font
sizes 700px starves the line; full width would fatigue it — the margins
stay calm and empty by intent). First run on a ≥1280px screen with no
saved font size starts one step up (26px); a saved choice always wins.
**Portrait = top and bottom, landscape = the sides** (owner,
Nov 25). Portrait keeps the current chrome at every size — the top
bar and pager above, the player below; a tablet in portrait is the
phone design, larger. In **landscape** the reader stays centred and
untouched — **the Bani column never moves or resizes for any
panel** — and the chrome lives in the empty side space, **all of it
on the right**; the left side stays empty on purpose, reserved for
whatever comes next (meanings, notes):

- **Phone landscape** (short height): the top bar becomes a slim
  60px icon rail down the left edge (ੴ mark, Search, fullscreen,
  Appearance, Menu; the pager stacked at its foot), the stream gains
  a matching left inset, and the player stays one compact row along
  the bottom, clear of the rail. The rail slides away with the bars.
- **Landscape ≥1024px** speaks the phone landscape's language —
  the arrangement the owner picked as the best of the set. The
  reading column and the audio bar stay **centred on the window**,
  and the Recenter trio sits at the bottom centre where it always
  was. On the right edge, a slim **icon rail** carries navigation
  and reading controls only — ੴ mark, Search, Library, Menu, text
  A−/A+, Appearance, fullscreen — because everything already in the audio bar (play,
  jump undo/redo, follow-along) stays in the audio bar: one home
  per control. The rail slides away with the bars, like the
  phone's. The **Menu/Library/Search panel grows out of the
  rail** — parked immediately left of it, top-aligned, one visual
  unit, floating over the right margin, capped at 520px tall
  with its content scrolling inside — and inside it is the
  compact cut of the menu: small section headings, tight rows,
  each control full-width under its label, nothing oversized; the
  sync button is Google's own button (white, the four-colour G).
  The **Ang pill travels as a unit, top centre**: its own ‹ ›
  switchers flank it (the pager's chevrons live here now, their
  disabled state mirrored from the pager); the Ang number lives
  in the pill and nowhere else — the rail carries no Ang readout.
  Clicking the pill opens the Ang switcher as a centred floating
  popup. The unit shows on
  an Ang switch, **once on landing**, and **whenever a tap calls
  the floating UI up** — alongside the quick cluster and the bars.
  The **quick cluster** stands just left of the rail above the
  bar on the phone's own tap-to-show cycle. The side system is
  **responsive**: the panel steps 280/320px and the rail 60/68px
  across width tiers, and on short landscape windows (≤760px
  high) the rail, its buttons and the panel scale down a cut.
  Measures: 592px at 1024–1199, 648px at 1200–1399, 800px at
  1400–1599, 840px from 1600. There is no brand text in the
  rail — just the ੴ mark, like the phone.

**Book mode.** Menu → Display carries a Book row, shown only where a
book fits: landscape and ≥1024px wide (desktops and tablets; never on
portrait or phone). It opens a two-page spread over the reader — the
left page is the current Ang rounded down to odd, the right page the
next Ang — each page a whole Ang in Print flow, with the Ang number
centred at the foot. Pages TURN (‹ › buttons, arrow keys, swipe);
nothing scrolls and the page never clips: the book sets its own type
size per spread, fitted to the page box and the fuller page's length,
so the Text size setting does not apply in the book — the reader
zooms the spread instead (± buttons, Ctrl+wheel, 50–200%). Playback
pauses on entry; the stream underneath is untouched; closing returns
the stream to the spread's left Ang. Rotating out of landscape (or
below the size floor) closes the book.

**Pankti Search (in-Reader popup).** A search button sits in the top
bar (beside Appearance) and in the floating quick cluster. It opens a
sheet (the Menu sheet's shape and motion) holding one search field, a
mic button, a one-line status, and the results — all inside the popup;
the results list scrolls with the sheet. Typing works in English
(roman) or Gurmukhi and is fuzzy; the mic recites in Punjabi
(SpeechRecognition pa-IN where the browser offers it; the button
hides where it does not). What was heard is NEVER displayed in the
Reader — that readout is a debugging aid and lives only in the
standalone Pankti Search tool. The engine is that tool's matcher
(vendored `search-matcher.js` / `search-transliterate.js`); its index
(vocab + trigrams + per-100-Ang shards, fetched lazily and cached) is
served from the Pankti Search site. A result tap opens the pankti in
the Reader as a free-reading detour — the same landing as a Pankti
Search link — so no Sehaj Paath is ever disturbed by a search.
**Matcher sync rule (do not break):** `search-matcher.js` and
`search-transliterate.js` are VENDORED COPIES of the Pankti Search
repo's `src/matcher.js` / `src/transliterate.js`, which are the single
source of truth. Never edit the vendored copies directly — edit in
KaramOfficial/gurbani-voice-search, run its four test suites
(test_matcher, test_stress, test_gemini, test_resilience), copy the
files over, and verify md5 equality before deploying. Scoring is
resilience-first (IDF word-set overlap primary, alignment as a bonus —
extra words, wrapped/rotated starts, and kirtan word order are all
tolerated; see that repo's docs/RESILIENCE-FIX.md). The search sheet's
low-confidence threshold is 0.50, matched to that scoring — keep it in
step with the standalone tool's `app.js`.

**Top-bar context line.** The line under "Gurbani Reader" always says
where the reader is: a Bani shows its own name; an Ang in free
reading shows ਸ੍ਰੀ ਗੁਰੂ ਗ੍ਰੰਥ ਸਾਹਿਬ ਜੀ; an Ang inside a Sehaj Paath
shows "ਸਹਜ ਪਾਠ · <that Paath's name>" (ellipsis-truncated when long).
It repaints on every world/Paath/rename change (via updateRecenter).

**Account & cloud sync (optional).** The Menu carries a compact
Account block: "Sign in with Google", or the signed-in email + Sign
out. It uses the same Firebase project as the Nitnem Tracker
(`nitnem-tracker`, compat SDK vendored under `firebasejs/10.12.5/`)
and the user's own doc in its `progress` collection (merge writes
only; the two apps never touch each other's fields).

**sync4 (2026-11-25)** is the current engine. sync3 failed in the
field in three ways its stubbed tests could not see — the Account
note wedged on "Still syncing" on every browser, Kiwi lost a
session's progress when its tab was frozen/killed, and a second
account on the same browser inherited the first account's library.
sync4 redesigns the engine's internals around the battle-tested
shape (Readest's production sync, Chrome Sync's per-item model,
Antigravity's Reader audit) while keeping the owner's product
rules below unchanged. The regression suite `test_sync4.py` (in
~/workspace) simulates the hostile conditions — a `set()` that
never settles, per-account cloud docs, recovery hooks — and fails
on sync3 (11 checks) while passing on sync4.

Sync is **per record**, never a whole-state blob. What is shared is
durable progress: `rec_paaths` (the Sehaj Paath list — ids and names
only; which Paath is current is this device's own business and is
not synced — with one first-contact exception: a device that has
no established world of its own for an account **claims the
cloud's most recently updated Paath once**, at its first hydrated
snapshot (the newest `rec_ctx_<id>` server stamp names it;
`switchPaath` lands it, and the Library shows it as current,
because it is this device's current Paath now). The claim is
recorded per account in the device meta, outside the datasets;
an account whose parked dataset carries a mode of its own is
never re-claimed, and no later snapshot ever moves a device's
world), `rec_settings` (font size, speed, follow-along mode,
appearance, advanced mode, auto-scroll, paragraph headers), and one
`rec_ctx_<id>` per Sehaj Paath carrying **its head alone** (`{ glf }`
— the last focus line; no place, no scroll position, ever). What is
**never** shared is presence: free (Default) reading's place and
focus (`detour-v1`), the world this device is in (`mode-v1`), the
live records (`last-v1`, `focus-v1`), and the per-size-class layout
keys. Free reading is per-device, full stop. The current Paath's
shared record is *derived* from the live focus while this device
is in the Paath world (a Bani focus never enters it — the parked
Ang head stands, exactly like a switch-time snapshot); in free
reading the record is the parked snapshot, unchanged — free
reading can never dirty a Paath record.

Each record carries its own server timestamp (`<field>_at`, a
Firestore `serverTimestamp`) and its writer's device id
(`<field>_by` — a device never applies its own writes; it adopts
the stamp and moves on).

**Canonical record form.** A record's synced JSON is byte-stable
when its meaningful content is unchanged: the focus line's audio
position (`t`) is presence, not progress, and is stripped from a
Paath record's synced form on both sides (what is pushed, what the
sent-baseline stores, what an applied record is compared against).
`ts` — when the head line was SET — is kept; re-saving an unchanged
focus keeps it. Playback therefore never churns a record: while a
line plays, the record is identical, and a quiet app writes
nothing.

**One snapshot listener**, attached at sign-in, is the only reader:
its first delivery IS the sign-in merge (there is no separate pull
and no read polling), and every later remote write arrives the
same way. The listener is attached with `includeMetadataChanges`,
because the Account note is driven by `hasPendingWrites`.

**One write in flight.** Every push driver — the `reader-dirty`
save event (~1.5 s throttle), the 2.5 s reconcile interval,
snapshot echoes, pause/tab-hide/pagehide flushes — funnels into a
single guarded push. A driver that arrives mid-flight sets a flag
and coalesces into one follow-up round; duplicate writes can never
stack behind a slow acknowledgement. Per record the device
remembers only two things in `sync-v1`: the exact canonical JSON
last sent or applied, and the newest server stamp applied. A
record that vanishes from the document is republished by the
reconcile. Removing a Paath from the list deletes its `rec_ctx_*`
fields in the same merge write.

**Wedged-write recovery.** A `set()` that neither resolves nor
rejects within 25 s is declared wedged: the flight is abandoned
(a sequence token makes its late settle a no-op; the snapshot echo
and sent-guard reconcile whatever it landed) and the Firestore
connection is actively recovered with disable/enable network. The
same recovery fires when snapshots report pending writes for over
20 s with no flight outstanding (e.g. a persisted queue replaying
into a stalled channel). Recovery is cooldown-limited (30 s) and
its own flag has a safety timer — sync can never hang silently
again; the 12 s "Still syncing — will keep trying." note still
appears while a slow write is genuinely in progress, but recovery
now acts behind it instead of only repainting it.

**Transport hardening.** At init, before any other Firestore use:
automatic long-polling detection (`experimentalAutoDetectLongPolling`)
for browsers whose streaming channel stalls, and guarded offline
persistence (`enablePersistence({ synchronizeTabs: true })`,
silently degrading where unsupported) so the mutation queue is
durable in IndexedDB — a write queued as a tab is frozen or killed
still lands on the next launch.

Ordering is **plain last-writer-wins on the server stamp** — the
standard rule, with no position heuristics and no client-clock
comparisons (a position-first + 2-minute-margin rule shipped on
2026-11-25 and was removed the same night: it classified a
deliberate undo — a behind head with a seconds-newer stamp — as a
stale packet, vetoed it, and republished the wrong line back over
it). The head is one line and the newest write IS the reader's
latest intent, ahead or behind. Two guards only: a record for a
Paath this device's list does not carry is left unseen (a later
list that names it evaluates it fresh), and at first contact a
customised local list is never replaced by a factory-default cloud
list; settings take the cloud's word. Pending-write echoes carry a
null stamp and are skipped record by record.

Pushes are **gated on server hydration**: until the listener's
first non-cache snapshot has been handled, no push fires — a
booting device merges before it ever writes, so its stale boot
state cannot overwrite fresher cloud records (cache-only snapshot
deliveries settle and apply nothing). The gate **re-arms on
reactivation** (the Readest pull-on-open rule): when the tab
becomes visible again or the device comes back online, pushes
lock until a one-shot server `get()` has been pulled through the
same handler — a woken device applies the cloud's latest state
before it can write anything — with an 8 s watchdog that lifts
the gate if no server answer ever comes, so it can never wedge
shut. Every Paath snapshot write (the `writeCtx` funnel,
switch-time snapshots, assign-to-Paath) announces `reader-dirty`
like every other save.

**Per-account datasets.** The live local dataset (every
`gurbani-reader:*` key except the sync meta) belongs to one owner:
an account uid, or the guest. The sync meta records the owner
(`ownerUid`; null = guest) and whether any account has ever owned
a dataset here (`everAccount`). Identity transitions are handled
before the new listener attaches: signing in with a different
account parks the live dataset under its owner (stash key
`gurbani-reader:acct:<uid>`, guest under `acct:guest`), installs
the incoming account's parked dataset — or leaves the keys clear,
which the app reads as a fresh install — **in place, with no
reload**, re-enters its world exactly as a fresh boot would
(Paath data and focus re-read, settings re-applied, lists rebuilt,
world restored from the dataset's own mode/place), and resets the
record baselines. Sign-out parks the account's dataset and returns
to the guest dataset the same way. The one exception: a dataset
that has never belonged to any account (read before the first-ever
sign-in) is *adopted* by that first account instead of being
parked away. A fresh account therefore opens with a fresh library,
sees nothing of another account's Paaths or heads, and its cloud
doc only ever receives its own records. Any push still in flight
at a transition is abandoned by flight token, so the old identity
can never write under the new one.

**Build freshness.** The build carries a token (`reader-build:` in
an HTML comment at the top of `index.html`). On boot and on focus
(throttled to 10 min) a signed-in page fetches its own URL with
`cache: no-store` and compares tokens; a mismatch names itself in
the Account note ("A newer Reader version is ready — refresh this
page to update.") and never forces a reload mid-reading. A page
left open or served from cache otherwise runs its loaded sync
protocol indefinitely and silently — the cloud doc's frozen legacy
`readerJson` field is the fossil of exactly that.
The Library's Paath label/preview read the live records only while
actually in that Paath; in free reading they read the synced
snapshot. While audio plays, the focus is saved continuously
(throttled), and a real pause saves place as well as focus.

Applying a record is **in place**: it never reloads the page,
never closes an open sheet or popup, and never moves the reader.
A Paath record updates that Paath's stored resume point; if it is
this device's current Paath and playback is paused, the live
records move underneath the reader (the view does not — the
Kindle rule); while playback is running, only the stored point
updates and this session's own next save supersedes. A list record
that drops a Paath removes its local snapshot and repairs
`current` if needed; a Paath record for an id the list does not
name is ignored. Settings records re-apply the settings underneath
whatever is on screen. (The one deliberate exception to "never
reloads" thinking: an *account switch* re-enters the incoming
dataset's world in place — still no page reload; see Per-account
datasets above.)

A failed push or listen says so in the Account note **with the
Firebase error's short code** ("Couldn't sync (permission-denied)
— your reading is safe on this device and will retry."), the full
error in the console under `[reader-sync]`; a failed push retries
after 20 s and on the next reconcile. Sign-in is popup-first with
a redirect fallback when the popup is blocked or closed, and a
redirect result's error surfaces in the note. Signed out, SDK
blocked, or offline: local-only, exactly as before. The legacy
whole-state `readerJson` field is no longer written or read; old
builds keep their frozen copy and never touch the record fields.

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
   hand-browsing only; Recenter whenever the focus pankti is not in
   the main position, playing or paused, evaluated on every scroll
   frame; one tap model
   (single = fading feedback, double = play) (§9).
9. Keep the design tokens and the Cook-pass decisions (§10) — and get
   the owner's on-device verdict before calling any of it done.
