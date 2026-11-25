# Pencil Sharpener — Design

A satisfying 3D pencil sharpener sim. One idea, done well: the feel of
sharpening a pencil. Built in a single `index.html` (Three.js vendored
locally in `vendor/`, no runtime CDN dependency), mobile-first, touch
friendly. Synthesized WebAudio sounds only, no audio assets.

## Slice 1.1 — blank scene on phones (fixed) + reset button
- Root cause: `renderer.setSize(w, h, false)` never set the canvas CSS size. A canvas is a replaced element, so `position: fixed; inset: 0` does not stretch it; it keeps its intrinsic size, which is the drawing buffer (CSS size x devicePixelRatio). On DPR 1 desktops the buffer equals the viewport so it looked fine; on any real phone (DPR 2-3) the canvas rendered 2-3x too big, anchored top-left, pushing the sharpener off-screen bottom-right and leaving only empty background visible. The game logic still ran, which is why the meters moved.
- Fix: `renderer.setSize(w, h, true)` plus explicit `width: 100%; height: 100%` on the canvas, and `setPixelRatio` re-applied on resize. Portrait camera re-framed (fov 60, pulled back to 12.6, centered on the scene) so sharpener, pencil, tray and chest all fit a narrow phone screen; fog pushed out to match. Crank enlarged (1.45x to 1.75x) for a bigger grab target.
- Reset button: small "Reset game" button at the bottom of the Shop panel. Two-tap arm/confirm (arms for 4 seconds, second tap wipes). Clears the localStorage save and restores the starting state: 0 shavings, fresh pencil, empty tray, no upgrades, manual mode.
- Verified headless in mobile emulation (390x844, DPR 2.25): canvas rect equals viewport, sharpener/pencil/desk/tray/chest all visible in a portrait screenshot, zero console errors; desktop 1280x800 also clean.

## The fantasy

You are at a desk with a chunky old-school sharpener, a box of pencils,
and a wooden chest. Sharpen pencils. Peel off curls. Dump the shavings.
Spend shavings on a better sharpener. That is the whole game, and the
whole game is the feel.

## Core loop

1. Sharpen a pencil until it has a perfect point (sharpness 0 to 100).
   Sharpening peels shavings into the tray under the sharpener.
2. When the tray fills up, sharpening stops. Slide the tray out and dump
   the shavings into the chest. Shavings only count once they are in the
   chest.
3. Spend chest shavings on upgrades. Repeat with a better machine.

## Manual sharpener (starting machine)

- Drag in circles around the crank on the side of the sharpener, mouse
  or touch. Each chunk of rotation is one crank step: a click, a curl
  of shavings peeling off, the pencil visibly getting sharper (the
  graphite point grows, the wood cone lengthens, the pencil gets
  slightly shorter as material comes off).
- A full pencil takes about 5 crank rotations. A full pencil fills
  most of the tray, so the dump rhythm is about one dump per pencil.
- A finished pencil pays a small bonus, then a fresh dull pencil
  appears.

## Electric sharpener (first big upgrade, 150 shavings)

- Press and hold anywhere on the scene to feed the pencil in.
- Feed rate ramps up while you hold. There is a sweet spot on the
  feed meter (green zone). Push past the motor's limit and it bogs
  down: the pitch drops, the cutting rate falls to a crawl, the meter
  goes red. The skill is pulsing the hold to stay in the green.
- Better motors raise the bog threshold and feed faster. Sharper
  blades cut faster and peel finer curls.

## Upgrades (all bought with shavings)

| Upgrade | Effect on feel, not just numbers |
|---|---|
| Electric sharpener | New machine, new input, new sound (motor hum vs crank clicks) |
| Sharper blades (3 levels) | Faster cutting, finer curls |
| Better motor (3 levels) | Higher bog threshold, faster feed, deeper motor note |
| Bigger tray (2 levels) | Longer sharpening runs between dumps |
| Brass gear set (3 levels) | More shavings per pass |
| Colored pencils | New pencil colors cycle in, same sharpening, prettier curls |
| Auto-empty tray | Tray dumps itself into the chest when full |

## Power surge

Every 100 to 160 seconds a 12 second power surge hits: the motor can
take anything, the LED goes amber, a light glows, a chime plays, and
the banner says to push it. No bogging during a surge, double cutting
speed. Zen mode turns surges off.

## Zen mode

A toggle. No surges, no pressure, just sharpening. The tray and the
shop still work.

## HUD

Shavings in the chest, pencil sharpness bar, tray fill bar, feed meter
(electric only, with green sweet spot and red bog zone). Shop, dump,
and mode buttons are big touch targets at the bottom.

## Save

localStorage: wallet, tray, sharpness, pencils done, upgrades, mode,
zen, sound. The game resumes where you left it.

## What is built (slice 1, Nov 24 2026)

- Full core loop: manual crank, electric feed with bog down, tray and
  dump, chest wallet, pencil completion and bonus, shop with all
  upgrades listed above, power surge events, zen mode, save and load.
- 3D desk scene: sharpener (manual green body, electric charcoal
  unit), crank, pencil with live sharpness visuals, sliding tray with
  a growing shavings pile, wooden chest with a lid that opens on
  dump, curl particles that fall and rest, paper and eraser props.
- Synthesized sounds: crank clicks, motor hum tied to feed rate and
  bogging, curl swishes, dump thump, surge chime, pencil done
  arpeggio.
- Verified headless (Playwright + SwiftShader): loads with zero
  console errors, crank drag raises sharpness and fills the tray,
  dump moves shavings to the wallet, electric purchase and feed work,
  mode switching works.

## Rough edges (known, not yet fixed)

- The crank is the main interaction and is still small on a phone
  screen. It works, but it deserves to be bigger and more obvious.
- The tray dump is a two tap button flow. The design calls for
  dragging the tray out by hand. That is the next feel upgrade.
- Curl particles rest on the desk and fade. They do not pile up in
  the tray as a visible heap yet (the pile mesh grows, the loose
  curls do not).
- No haptics. A tiny vibration on each crank step would help a lot
  on phones.
- Feel is unverified. The phone playtest is the verdict, never this
  document.

## Next slices

1. Tray drag: drag the tray out with a finger, tilt to dump, curls
   fly into the chest.
2. Sound and curl pass: richer crank ratchet, motor character per
   motor level, curls that land in the tray and stay until dumped.
3. Surge polish: screen glow, stronger cue, surge specific curl
   colors.
4. Scene and HUD polish: better desk dressing, pencil box, sharper
   visual hierarchy on small screens.
5. Progression tuning from playtest: costs, tray size, surge
   frequency.

## Monetization (undecided, not built)

Either a few bucks up front with no ads, or free with banner ads and
rewarded ads for power surges. The surge ad fits the design: watch an
ad, trigger a surge. No ads are in the prototype.
