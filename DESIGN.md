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

## Slice 2 — drag the tray out by hand, with haptics
- The tray is a real grab target now: raycast against the tray meshes (generous invisible hit box for touch), pull it out toward you, then drag it toward the chest to dump. Drag it back up to slide it back in. Short pulls snap back, long pulls snap out.
- Haptics via navigator.vibrate: grab, full extension, dump.
- The dump pours: the tray tilts and shifts toward the chest while shaving curls arc into it over 0.9s, the chest lid opens, the wallet ticks up.
- The "Slide tray out" button stays as the fallback path; both paths share the same pour/finish code.
- One regression caught in verification: the edit dropped the pointerAngle helper and broke cranking. Restored and re-verified (crank raises sharpness, tray fills, dump banks, button fallback works, desktop drag works, zero console errors).

## Pivot (Nov 24, 11:51 PM) — manual first, electric shelved
Brandon's call: nail the manual before anything else. Electric is shelved for now (it comes back eventually: slowly pressing a pencil in and grinding it to the ground, which you cannot do in real life but he always wanted to). Also: make it look really good, not just feel good.
- Camera: straight on at the crank and the pencil hole. The sharpener faces the camera, barely tilted H/V, just enough tilt to read as 3D. The crank and the hole are the shot.
- The crank moved from the side face to the FRONT face, facing the camera, so a circular drag on screen turns it 1:1. It spins continuously with the hand now (before, the visual angle snapped back every step). Dead zone near the center so tiny moves do not swing the angle wildly, per-move delta clamped, no dead zones otherwise.
- Grab affordance: the crank handle and arm glow amber and the cursor turns to a grab hand when the pointer is over the crank; grabbing shows the grabbing cursor.
- Sound tracks the turn: a looping scrape noise whose gain and filter follow crank speed, plus the ratchet click per step.
- Pencils grind down: every sharpening step eats a little length. A full pencil lasts about 10 crank rotations, then it is a tiny sharp stump and an eraser. Sharpness still climbs per rotation and pays the perfect-point bonus once per pencil.
- Spent pencils become stubs: a second currency. Stubs pile visibly in a little round dish on the desk, colors saved so a reload rebuilds the same pile. Stub count shows in the wallet bar. Spending stubs comes later.
- A fresh full pencil takes the spent one's place automatically, so the loop never stalls.
- Verified headless (390x844 portrait + 1280x800 desktop): zero console errors, hover highlights the crank, 3 crank circles take sharpness to 65% and length to 0.74 with the crank spinning exactly 3 revolutions, grinding a pencil to a stump banks a stub and spawns a fresh pencil, stubs persist across reload.

## Slice 2a (stashed, not shipped)
- Curls that land in the tray stay there and ride with the tray instead of fading, and get flung into the chest on dump. Implemented and half-verified (persistence confirmed, dump fling not yet verified) then stashed when the pivot landed: `git stash` "slice 2a WIP". Revisit after the manual feel is settled.

You are at a desk with a chunky old-school sharpener, a box of pencils,
and a wooden chest. Sharpen pencils. Peel off curls. Dump the shavings.
Spend shavings on a better sharpener. That is the whole game, and the
whole game is the feel.

## Core loop

1. Grab the crank and turn it in circles. Each turn sharpens the
   pencil (sharpness 0 to 100) and grinds it shorter. Sharpening
   peels shavings into the tray under the sharpener.
2. A pencil ground down to a stump becomes a stub in the round dish.
   A fresh pencil takes its place. Stubs are a second currency
   (spending comes later).
3. When the tray fills up, sharpening stops. Slide the tray out and dump
   the shavings into the chest. Shavings only count once they are in the
   chest.
4. Spend chest shavings on upgrades. Repeat with a better machine.

## Manual sharpener (starting machine, THE interaction)

- The crank is on the front face, big, facing the camera. Grab it
  (it glows when the pointer is over it) and drag in circles, mouse
  or touch. The crank follows your hand 1:1, the scrape sound tracks
  your speed, and every chunk of rotation is a crank step: a click,
  a curl of shavings peeling off, the pencil visibly getting sharper
  (the graphite point grows, the wood cone lengthens) and shorter
  (the eraser end rides down toward the tip).
- A full pencil takes about 10 crank rotations to grind to a stump.
  Reaching a perfect point (100% sharpness) pays a bonus once per
  pencil. A spent pencil becomes a stub in the dish and a fresh dull
  pencil appears.

## Electric sharpener (SHELVED for now, Nov 24 pivot)

- Press and hold anywhere on the scene to feed the pencil in.
- Feed rate ramps up while you hold. There is a sweet spot on the
  feed meter (green zone). Push past the motor's limit and it bogs
  down: the pitch drops, the cutting rate falls to a crawl, the meter
  goes red. The skill is pulsing the hold to stay in the green.
- Better motors raise the bog threshold and feed faster. Sharper
  blades cut faster and peel finer curls.
- Shelved, not deleted: the code and shop entry still work, but no
  further effort goes here until Brandon says so. It comes back
  eventually as the "grind a pencil to the ground" fantasy.

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

Shavings in the chest, stub count, pencil sharpness bar, tray fill bar, feed meter
(electric only, with green sweet spot and red bog zone). Shop, dump,
and mode buttons are big touch targets at the bottom.

## Save

localStorage: wallet, tray, sharpness, pencil length, stubs and stub
colors, pencils done, upgrades, mode, zen, sound. The game resumes
where you left it, stubs included.

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

- The crank is the main interaction. It is big and front-facing now,
  but the feel is unverified. The phone playtest is the verdict.
- Curl particles rest on the desk and fade. They do not pile up in
  the tray as a visible heap yet (the pile mesh grows, the loose
  curls do not). Slice 2a implemented this and is stashed, pending
  the manual feel settling.
- No haptics on the crank itself yet (tray drag has them). A tiny
  vibration on each crank step would help a lot on phones.
- The stub dish sits at the left edge of the portrait frame and the
  chest is partly cut off on the right. The hero shot is the crank,
  but the framing could still be tightened.
- Feel is unverified. The phone playtest is the verdict, never this
  document.

## Next slices (manual feel only, until Brandon says otherwise)

1. Crank tuning from playtest: step size, resistance sound, curl
   rate, pencil lifetime (currently ~10 rotations per pencil).
2. Crank haptics: vibrate per step, stronger on the perfect point.
3. Curls that land in the tray and stay until dumped (slice 2a,
   stashed, revive and verify).
4. Stub spending: the first thing stubs buy.
5. Scene polish: desk dressing, pencil box, warmer light.

## Monetization (undecided, not built)

Either a few bucks up front with no ads, or free with banner ads and
rewarded ads for power surges. The surge ad fits the design: watch an
ad, trigger a surge. No ads are in the prototype.
