# Princess Game — deep audit

A plain, zero-build web app: one `index.html` (all HTML, CSS and JS inline),
`manifest.json`, app icons, and the character pictures in `assets/`.

This is the record of the **deep audit**: a complete re-read of the current
source (every line), a full human-style playthrough on the live site, and the
state of every system in the game.

## Verdict

**Healthy.** No functional bugs were found in this pass. Two small copy items
were corrected (see below). Everything previously reported broken has been
fixed and was re-verified in this audit.

## How it was audited

1. **Full source re-read** — all 2,400 lines reviewed: CSS, HTML, all 39
   character records, the audio engine, the speech queue, the picture system,
   the game loop, mash protection, and every event wiring.
2. **Deployed-file verification** — the GitHub API listing of the live
   Pages site confirms every picture (small `.webp` + big `.png` fallbacks)
   and the page itself are actually served.
3. **Human-style playthrough on the live site** — start screen, full rounds,
   early taps, wrong answers, retries, the 15-second timeout, mash-spamming,
   the victory screen and Play Again.

## Fixed in this pass

- **Stale comment**: the code still described the old YouTube-error history;
  it now describes what actually happens (official recordings via Apple's
  previews, capped at 15 s).
- **Copy accuracy**: the grown-ups status text said "official Disney
  recording" even for Mario, Pikachu, Link etc. It now says "official
  recording".
- (Carried over from the previous audit pass, still verified good: four
  characters whose pictures silently stopped showing, the scroll-position
  bug, the 30-second copy, and the non-button link.)

## Full playthrough log (live site)

- **Start screen**: title, bouncing character row, "Let's Play!" — clean load.
- **Countdown**: big 3 → 2 → 1 on the purple card, ~1 second apart, spoken cue
  ("Song 1. Who's singing?") riding along; music starts about 4 s after the tap.
- **Early tap test**: tapping an answer within the first 5 seconds only makes
  the song card wobble — the song keeps playing and nothing is scored.
- **Answers**: correct → confetti + fanfare + praise + "That was [song]";
  wrong → "So close!" with Twinkle's clue and "Listen Again"; after a second
  wrong answer the reveal flow shows the character and "Got It! ➜".
- **Mash-spam test**: five rapid taps on one answer card and rapid taps on
  "Keep Going!" produced no skips, no double-scoring and no double overlays.
- **15-second timeout**: an unanswered song ends itself and the card returns
  to "Tap to hear the song!" — no forced answer, no stuck state.
- **Victory screen**: confetti, star score (⭐ first-try + ✨ learned tally),
  praise line, and "Play Again" which resets cleanly to round 1.
- **No console errors, no failed requests, no stall over 2 seconds** at any
  point in the run.

## System-by-system health check

- **Data**: all 39 characters have a name, song title, working preview URL,
  colours, clue, chords, melody, picture and voice aliases; every name in
  `PNG_ART` matches a real asset; no duplicates.
- **Pictures**: small `.webp` → big `.png` → hand-drawn SVG fallback chain,
  plus an 8-second watchdog for requests that hang without an error event.
  All 39 verified loading on the live site.
- **Audio**: single reused player, muted priming inside the starting tap,
  unmuted after the countdown, 15-second cap, natural-end handling with a
  "Who is it?" prompt and bouncing cards, pre-fetching of the next song during
  the praise, uploaded clips, and a synthesised tune as the offline fallback.
  Every sound path is failure-proof (can't freeze a round).
- **Voice**: single queue, never cancelled mid-sentence, completion safety net
  (works around an iOS Safari bug), hard stop only on round change or answer.
- **Round logic**: round guard against double-scoring, ⭐/✨ bookkeeping,
  progress dots, star totals, Play Again reset — all consistent.
- **Mash protection**: 800 ms between answer taps, no taps during the
  countdown, 5-second minimum listen, one song restart per 2 s, overlays only
  respond while showing, speaker badges rate-limited.
- **Performance**: ~200 KB first load, async-decoded images, no blur filters,
  no idle animations, the hidden panel is only built when opened.

## Known limitations (deliberate or accepted)

- **Pictures are official character renders** (Disney/Pixar/Nintendo art), not
  original drawings; the drawn SVG fallbacks are simpler than the photos. Any
  character can be given custom artwork from the grown-ups panel.
- **Mary Poppins' picture is a film still** with a tiny watermark — the only
  usable image found for her; replaceable from the panel.
- **Songs are 15-second previews**, not full tracks. Full custom clips work
  from the grown-ups panel.
- **Real songs need internet**; offline, the tune version plays (custom clips
  still work offline).
- **Rare edge**: if the pre-fetched next song finished while the praise was on
  screen, iOS may refuse its delayed replay — the game then asks for one tap
  on the card (shown as "Tap to hear the song!") and plays normally.
- **Voice can go quiet after a long session** on some iOS versions (a Safari
  limitation); the game keeps working — beeps, praise sounds and songs are
  unaffected, and closing/reopening restores the voice.
- **Zoom is disabled** so a child can't accidentally zoom the layout; an
  intentional trade against accessibility.
- The game keeps playing behind the grown-ups panel if opened mid-song.
