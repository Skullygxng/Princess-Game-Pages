# Princess Game — audit

A plain, zero-build web app: one `index.html` (all HTML, CSS and JS inline),
`manifest.json`, app icons, and 39 character pictures in `assets/`.

This is the record of the full audit: what was checked, what was wrong, what
was fixed, and what is left as a known limitation.

## What the game does

Ten rounds per game, picked at random from 39 characters (Disney, Pixar and
Nintendo). Each round:

1. A spoken cue — "Song 3. Who's singing?" — then a 3-2-1 countdown with beeps.
2. The official recording plays (Apple's public preview, capped at 15 seconds).
   The first 5 seconds cannot be answered over; a guess only wobbles the card.
3. Three picture answers (the right one plus two random others).
4. Correct: confetti, fanfare, spoken praise, the song title revealed, ⭐.
   Wrong once: a gentle "So close!" with Twinkle's clue and "Listen Again".
   Wrong twice: "Now you know!" with the answer and an ✨.
5. The end screen counts ⭐ (first-try) and ✨ (learned with Twinkle).

## Fixed in this audit

- **Four characters had no picture.** A refactor of the picture map dropped
  Elsa, Ariel, Simba and Woody, so those four silently fell back to the
  hand-drawn SVG shapes even though their `.webp` files existed. They now show
  their real pictures; every one of the 39 characters was verified as an
  `assets/<Name>-s.webp` image on the live site.
- **The game screen could open scrolled down.** The start screen can be taller
  than a small phone, and the scroll position was inherited when the game
  screen appeared, cutting off the top bar, question and part of the song card.
  The screen now always starts at the top of each round and on the end screen.
- **Stale grown-ups copy.** The panel claimed a "30-second preview" after the
  song length was capped at 15 seconds; the text now matches, and a stale code
  comment about `.png` artwork was corrected.
- **The "For grown-ups" link was a `div`.** It is now a real button with proper
  styles, so it is keyboard- and screen-reader-reachable like the rest.

## Checked and found healthy

- Data integrity: all 39 characters have a name, song title, preview URL,
  colours, clue, chords, melody and a picture; every name in `PNG_ART` matches a
  real asset; no duplicate names.
- Fallback chain: small `.webp` → full-size `.png` → drawn SVG, plus an 8-second
  watchdog for image requests that hang without firing an error.
- Audio: one reused player, muted priming during the tap, unmuted after the
  countdown, 15-second cap, natural-end handling, clip uploads, and a
  synthesiser fallback when offline. All sound paths are wrapped so a failure
  can't freeze a round.
- Voice: a single queue, never cancelled mid-sentence, with a completion
  safety net and a hard stop only when the round changes or an answer is tapped.
- Round logic: no double-scoring (round guard), correct/learned bookkeeping,
  dots (🔵 current, ⭐ first try, ✨ learned), star totals, Play Again reset.
- Button-mash protection: 800 ms between answer taps, no taps during the
  countdown, one song restart per 2 s, overlays only respond while showing.
- Performance: ~200 KB first load, small images decoded async, no blur filters,
  no idle animations, hidden panel not built until opened.
- Live play-through: start, several rounds, answer/correct/incorrect overlays,
  grown-up panel, and the end screen all work with no console errors and no
  failed requests.

## Known limitations (deliberate or accepted)

- **Pictures are official Disney character renders and Mario-series art**, not
  original drawings; the ten drawn SVG fallbacks look simpler than the photos.
  Any character can be given a custom picture from the grown-ups panel.
- **Songs are 15-second previews**, not full tracks — Apple's public preview
  clips. A grown-up can attach a full clip per character.
- **No offline play of the real songs**: without internet the game falls back to
  the synthesised tune. Custom clips work offline.
- **Zoom is disabled** (`user-scalable=no`) so a child can't accidentally zoom
  the layout; this is an intentional trade against accessibility.
- **Audio starts only after a real tap** — browsers require a user gesture; the
  countdown is primed within the tap that starts the round.
- The game keeps playing behind the grown-ups panel if it is opened mid-song.
