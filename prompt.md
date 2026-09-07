# ASTERISM — one-shot build brief

A deep indigo night over a black skyline; dormant constellations hang as dim grey stars joined by dotted lines. Your fireworks ignite them — and what explodes is literally the recipe you built: Split, Seeker, Echo, Twin.

Competition entry judged on functionality and edge cases, UX and design, and originality — build to win. Work fully autonomously: never ask, never stop; when uncertain, decide and continue. Budget ~60 minutes. Deliver exactly one file, index.html, in the working directory, opening from file:// with no build, server, network, CDN, external font or image — everything procedural (canvas 2D + WebAudio + DOM overlays). English in-game text. The game is ASTERISM.

## The game
- Logical 960×600, aspect-fit letterboxed; skyline along the bottom ~12%; launch pad (480, 560). Round Rn = 3 launches over 1 constellation (R1), 2 (R2), 3 (R3–4), 4 (R5+). Constellation = cluster: circle r 90, centre x 150–810 / y 150–300, centres ≥220 apart; 5–9 stars (R1: 6; R5+: 6–10) ≥40 px apart, ≤200 sampling attempts then 10% relaxed spacing. Edges = minimum spanning tree: 20%-alpha dotted while dormant, bright when complete. One accent colour each (rose/cyan/gold, all different when ≤3). Generate the next sky before the shop opens.
- Slingshot aim, drag anywhere: d = release − press; the shell launches from the pad with direction −d, speed 250–650 px/s from drag length 40–260 px, clamped ≥15° above horizontal; dotted trajectory preview; drags <20 px cancel. One launch in flight at a time (Twin forks count as one). Gravity 220 px/s².
- Detonate on pointerdown or Space keydown after 0.15 s of flight; a detonating pointer is consumed for its whole gesture and never becomes an aim drag; a pointerdown with nothing in flight and launches left starts the next aim. Auto-detonate 0.4 s after apex or on leaving x<0, x>960, y<0, y>560. Detonation runs the recipe at the shell; sparks inherit 30% of its velocity.
- A spark ignites a dormant star within its ignite radius (16 px; Comet ×1.5, Heavy ×1.5, multiplicative) of its movement segment this substep; a star ignites once. Star: +10 × (1 + 0.1k), k = stars lit so far by this launch (forks, echoes, children share k). Tint match: ×2 on that value only (untinted sparks are white). Last star of a constellation: +25 × star count (never ×2), lines draw over 0.6 s, a procedural name flashes.
- Aim the next launch as soon as the previous detonated; after the third, aiming is ignored and the round ends when no spark or pending Echo remains, or 6 s later at the latest. Pass (round score ≥ quota): round card with breakdown and coins, dismissable after 0.8 s, auto-continues after 4 s. Fail: straight to "THE SHOW ENDS" (score vs quota, total, best); one press (ignored 400 ms) restarts into R1 aiming. Quotas 50, 150, 320, 520, 750, 950, 1150, then ×1.2 rounded to 10; only the round's own score counts.

## Recipe — the depth engine
Ordered slots, 3 at start, up to 6 via Casing. Interpret left to right: modifiers accumulate on a stack applied to every emitter after them, persist, are never consumed; empty slots skipped. Numeric modifiers multiply across duplicates and with the emitter's own values; Seeker and Twin are booleans; n Echo = n extra repeats. One pure runRecipe(recipe, x, y, vx, vy, world) queues emissions; game and preview call it with different worlds.

Emitters (count · px/s · life): Peony 24 random 140–220 · 1.4 s (starting recipe) · Ring 18 evenly spaced at exactly 240 · 1.0 s · Willow 12 · 120 · 2.4 s, gravity ×1.5 · Comet 5 big (drawn ×2, ignite ×1.5) · 320 · 1.8 s.

Modifiers: Split — at 50% life the parent becomes 3 children at −30°/0°/+30°, speed ×0.8, life 1.0 s, same colour, stack minus Split; children never split (a second Split card grants one further split, grandchildren 0.6 s); at the cap the parent still dies. Seeker — each substep rotate velocity ≤200°/s toward the nearest dormant star, preserving speed, then gravity; none left → straight. Feather — gravity ×0.25, life ×1.5. Heavy — gravity ×2, life ×0.8, ignite ×1.5. Echo — emitters after it fire again 0.6 s later at the burst point, same stack and inherited velocity. Twin — position-independent, the one exception: 0.4 s after launch the shell forks into two at ±12°, same speed, both run the full recipe; manual detonation fires every live fork, auto-detonation per fork; one launch, shared k. Tint (one card per colour) — colours later emitters' sparks; a later Tint overrides.

Order must be observable (Split→Seeker→Peony = homing clusters whose children seek). Spark cap 450 in the real sky; excess is not spawned.

## Shop and economy
Coins after a passed round: $2 + $1 per 25 round points + $2 per completed constellation, award cap $12; coins carry over; start $4. The shop overlays the live next sky: recipe row, 3 offers, Reroll ($2, +$1 each; disabled when unaffordable), coins, and a preview canvas (~260×160) looping the recipe's detonation every 2 s in a separate world (own pool and cap, five dummy stars, 0.4× scale; never touches the real sky or score).

Slots (identical on every input): tap to select; ◀ ▶ swap with the neighbour, ✕ sells for $1; tap a second slot to swap; buying goes to the first empty slot, or into the selected slot refunding $1 for the replaced charge. Always keep ≥1 emitter: ✕ and replace are disabled on the last emitter ("Keep one emitter"), so LAUNCH (button, Enter/Space) is always enabled. Bought offers show SOLD, not refilled. Casing applies immediately (+1 slot, max 6). Prices: Peony, Ring, Feather, Heavy, Tint 3 · Willow, Comet 4 · Split, Seeker, Echo 5 · Twin, Casing 6. Offer weights: emitters 40%, modifiers 50%, Casing 10%; no duplicates in one offer; the first shop's initial offer contains Split or Seeker.

## Controls
Desktop: drag = aim, release = launch, pointerdown or Space = detonate; R reroll (shop); Enter/Space = LAUNCH — in the aiming state Enter launches only after a fresh keyup; P/Esc pause; M mute (persisted). Touch: identical; first pointer only; touch-action: none; preventDefault on touch handlers and game keys; contextmenu suppressed. Pause: P/Esc or a HUD button; overlay with touch-usable Resume and Restart run; auto-pause on blur/visibilitychange cancels any drag. Guards: the input dismissing title or end card never launches or detonates and doubles as the audio-unlock gesture; handlers are state-scoped; guard key autorepeat.

## Rendering & audio
Indigo-to-black sky, static starfield, skyline silhouette. Title sky: no constellations, a non-scoring demo Peony every ~4 s; dismissing it generates R1. Sparks draw on an offscreen effects canvas faded each frame with translucent black (alpha ~0.25) and composited with globalCompositeOperation 'lighter'; the scene itself is redrawn clean every frame. prefers-reduced-motion: no screenshake, shorter trails. DOM overlays sit in one wrapper over the canvas box scaled by font-size 16 px × boxWidth/960 (em units); portrait shows a rotate hint. HUD (pointer-events none): round, quota bar, launches left, coins, score, best, recipe. No shadowBlur in the loop, no per-pixel simulation, ≤1 full-frame gradient plus the effects canvas.

Audio, all WebAudio: AudioContext created lazily on the first gesture, webkitAudioContext fallback, every call guarded (blocked = silence). Launch whistle, detonation thump, sparse crackle ≤12/s, star = C-minor pentatonic note by height, constellation = rising arpeggio, quota pass chord / fail slide. Every gain ramp ends at 0.001, never 0.

## Robustness
requestAnimationFrame, frame delta clamped to 50 ms, fixed 1/120 s physics substeps; all timers game-time. SafeStorage: localStorage in try/catch with in-memory fallback, one versioned JSON blob; corrupt or blocked → silent defaults. devicePixelRatio backing store, aspect-fit letterbox, inputs mapped through the scale, recomputed on resize/orientationchange mid-game. Atomic restart: [Peony], 3 slots, $4, R1, fresh sky, no sparks, echoes or timers. Pool sparks; cull star checks per cluster. Detonate before launch or after all sparks died → ignored; Twin + Echo → both forks echo. Zero console errors.

## Process
Plan first. Core: sky generator, slingshot, flight, detonation, runRecipe with Peony/Ring + Split/Seeker/Tint, scoring, quota, round and end cards, restart — index.html playable at every stage. Then the shop (preview world, slots, reroll, Casing) → remaining charges → audio and juice. Cut order if time runs short: Willow, Comet → Echo → Twin. Never cut: slingshot, Peony, Split, Seeker, Tint, quota rounds, shop with preview, Casing.

At ~T−15 minutes stop adding features; verify with the strongest tooling available (headless browser if present, else syntax-check and trace the state machine by hand), walk the acceptance tests, tune, re-verify. Hunt for: stack leaking between recipes or worlds, children ignoring the cap, Seeker chasing lit stars, double ignition, a detonating pointer becoming an aim drag, quota compared with total score.

## Acceptance — all must pass
1. [Peony] alone: a careful first-timer passes R1 almost always and R2 usually, fails by R3–R4 buying nothing, never passes R5.
2. Split→Seeker→Peony aimed reasonably usually passes R4; its preview differs visibly from Peony→Split→Seeker; a strong build reaches R8+.
3. Early vs late detonation lights different stars; a downward pull still flies at 15°; a completed constellation draws its lines and shows a name; a Tint match doubles the popup.
4. 60 fps with 450 live sparks; zero console errors from title through fail and restart.
5. Touch: the aim drag never scrolls, the detonating tap never launches the next shell, the shop can never trap the player; blur, mid-flight resize, blocked storage, silent audio and triple restart behave as specified.
