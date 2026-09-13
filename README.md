# Kick the Green Building

Physical force in, 153 windows out.

A visitor kicks a football at a netted target on the lawn. Sensors read the kick. A scoring engine turns it into power, accuracy and an estimated ball speed. An interpreter turns *that* into a reaction with a personality. And MIT's Green Building answers across all 153 of its windows.

This repo contains a complete, playable simulation of that loop, plus the bridge code to drive the real facade.

**[▶ Play it](https://wilsonwu-ai.github.io/KickTheGreenBuilding/)** — no install, no build step, works on a laptop or a phone.

Hosted on GitHub Pages from the root of the `main` branch. Updates pushed to `main` are published automatically.

**GitHub Pages availability:** Standalone gameplay and the local leaderboard work here. The upstream AI commentary, live phone/display synchronization, and shared leaderboard require Claude runtime services that GitHub Pages does not provide. Frame export falls back to an inline preview. The upstream documentation below also references `docs/` and `bridge/` files that are not included in this repository.

---

## The display is real, and this is built to its spec

In April 2012 MIT hackers installed 153 wirelessly controlled RGB lights into every window above the first floor of the Green Building (Building 54) and played Tetris on it. The installation stayed. Its parameters are not invented here:

| | |
|---|---|
| Windows | **153** |
| Grid | **9 columns × 17 rows** |
| Color depth | **24-bit** |
| Frame rate | **15 fps maximum** |
| Bytes per frame | **459** (153 × RGB) |
| Row 0 | Floor 18 · Row 16 → Floor 2 · first floor is unlit |
| Building | 18 floors, 277 ft, counted by MIT as 21 stories |

The simulator renders that exact grid, at the real 6 × 8 ft window proportions and pitch, and generates frames at 15 fps rather than at display refresh — so what you see on screen is the same data volume and cadence the building would receive.

---

## What's in here

```
index.html              the whole simulator — one file, no build, no dependencies
docs/ARCHITECTURE.md    signal chain, scoring model, and what we honestly measure
docs/frame-protocol.md  the 459-byte frame format and NDJSON replay format
bridge/KickPlugin.java  d54 DisplayPlugin implementation for the real facade
bridge/server.mjs       scoring service + frame broadcaster (WebSocket)
```

`index.html` has no dependencies and no build step. Open it, or serve it:

```bash
python3 -m http.server 8000    # then open http://localhost:8000
```

---

## How to play

Three input adapters, all of them phone-or-laptop only — no hardware required for the demo.

**Charge meter** (default, and the one to demo on a projector). Hold to charge, release at peak power, then stop a sweeping reticle on the target. The reticle's horizontal position *is* the impact column, so your aim picks which of the 9 window columns ignites. Space bar works.

**Flick.** Swipe velocity is power; deviation from a straight line is accuracy.

**Swing phone.** DeviceMotion peak acceleration. It self-tests on activation and falls back automatically if the browser withholds sensor data, which embedded frames often do. Don't build a live demo on this one.

### Modes

- **Arcade** — building and controls on one screen. The safe demo. Needs nothing but a space bar.
- **Building** — big-screen view for a projector, with a join QR code.
- **Controller** — phone view. Scan, switch to this, kick; scores land on the projector live.

---

## How the building answers

Score bands pick a reaction, and the reaction is shaped by *how* you got the score — not just the number.

| Score | State | Reaction |
|---|---|---|
| 0–25 | DORMANT | A few windows flicker. "The building barely noticed." |
| 26–50 | STIRRING | Energy climbs partway and stalls. |
| 51–75 | AWAKE | A wave oscillates through the powered floors. |
| 76–90 | LOUD | Accurate → a narrow **beam** races to the roof. Powerful but wild → a **scatter** burst that sprays sideways and misses. |
| 91–100 | RIOTING | All 153 windows, hue-cycling ripple rings. "WHO JUST KICKED THE GREEN BUILDING?" |

Between kicks the facade idles: random lit offices breathing, a watermark of your last score, and a thin magenta ledge marking the record to beat.

### Two interpreters

A rule-based one always runs: score band, power-versus-accuracy imbalance, improvement over your last kick, streaks, proximity to the daily record.

An optional AI interpreter (the **AI commentary** toggle, available in the hosted build) hands the model the whole kick — power, accuracy, speed, impact column, your previous scores, today's record — and gets back both a line *and* a light pattern. The building's animation changes based on what the model picks, so the same force can produce different behavior for different people.

---

## Beat the Building

Every day the building sets its own score, derived deterministically from the date, in the 78–97 range. Players aren't competing on a football game — they're competing against the building. Beat it and the record becomes yours for the day.

---

## What we claim to measure — read this before you demo

**We do not measure kick force.** A football leaving a foot carries force we never see.

What *is* honestly measurable with cheap hardware is **ball velocity** and **impact location**. A radar module or a 120 fps camera gives both. Real impact force needs an instrumented target: a piezo or load-cell backing board.

So the scoring model takes velocity and impact point as ground truth and calls power an **estimate**, with a calibration constant you set against a radar reading. The UI has a field for it: kick a real ball past a radar gun at full power, enter the reading, and that number becomes 100%.

Say this out loud in the demo. It is the difference between a toy and a prototype.

---

## Throughput, for a real installation

The facade is a single output device: however many phones are connected, one kick plays at a time.

- Building reaction: **3.82 s** (420 ms impact, 1.1 s climb, 1.5 s verdict, 800 ms settle)
- Ball back from a rebound net: 3–5 s, overlapping the reaction
- Player swap: ~5 s
- **New player scanning a QR and naming themselves: 20–30 s** ← the actual bottleneck

With per-player scanning you get ~2 players/minute. Keep one or two pre-paired phones at the target as kiosk controllers and hand them over, and you get **~5 players/minute, around 300 an hour**. That difference matters more than anything in this codebase.

> **Known limitation:** there is no queue. A kick arriving mid-reaction overwrites the one playing. Fine for one person holding a phone; visibly broken the moment two people kick together. See [Issues](../../issues).

---

## Safety and approvals

Nobody kicks toward the building. The target is a netted rebound goal facing *away* from the facade, on the Great Sail lawn — roughly where the 2012 Tetris controller sat, about 70 m out. The facade is the **output**; the ball never goes near it.

That framing is what makes the approval conversation short.

---

## Credits and references

- [`mitrisdev/d54`](https://github.com/mitrisdev/d54) — the Green Building display plugin interface this bridge targets
- [IHTFP Hack Gallery: Hacks on the Green Building](https://hacks.mit.edu/Hacks/by_location/54.html)
- [Green Building (MIT)](https://en.wikipedia.org/wiki/Green_Building_(MIT)) — I. M. Pei and Araldo Cossutta, 1964

Not affiliated with or endorsed by MIT. Running anything on the actual facade requires the cooperation of the people who maintain it.

## License

MIT — see [LICENSE](LICENSE).
