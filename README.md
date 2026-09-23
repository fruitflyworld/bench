<div align="center">
  <h1>fruitflyworld / bench</h1>
  <p><strong>Don't exam the model. <em>Starve it.</em></strong></p>
  <p>The exam harness of <a href="https://fruitfly.world">fruitfly.world</a>: same seed, same brain, run twice —<br/>a world is only a fair grader if the two runs are bit-identical.</p>
  <p>
    <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-baff35" alt="MIT"></a>
    <a href="../../actions/workflows/ci.yml"><img src="https://github.com/fruitflyworld/bench/actions/workflows/ci.yml/badge.svg" alt="CI"></a>
    <img src="https://img.shields.io/badge/dependencies-0-10b981" alt="zero dependencies">
  </p>
</div>

---

> Run an exam right now, no install:
> [fruitfly.world/play?bench=1&seed=42&brain=judgment&gens=2](https://fruitfly.world/play?bench=1&seed=42&brain=judgment&gens=2)

## The protocol

A model answer you cannot replay is an opinion. This harness turns the fruit-fly survival
world into a closed-loop exam:

1. **Fix the paper.** One editable seed derives everything — food placement, predator
   behavior, mutation drafts. Nobody, including us, cherry-picked it: `?seed=beacon`
   derives the seed from the latest Sepolia block hash, and the report links the block.
2. **Starve the model twice.** The render loop pauses; the world steps at a fixed 60 Hz
   through the full `(seed, brain, generations)` exam, then does it again from scratch.
3. **Compare every seal.** Every brain decision is hashed (`contentHash`, FNV-1a). If every
   hash and every generation outcome match, the run prints **IDENTICAL**. Anything else
   prints **DIVERGED** — *a bug report, not a score.*
4. **Grade the danger reads.** After the run, the brain's own danger scores are calibrated
   against reality: did a high score actually mean death within 5 seconds? Buckets plus a
   Brier score, over the sealed log only. Method, with its limits:
   [fruitfly.world/calibration](https://fruitfly.world/calibration).

Every result doubles as a challenge — one click copies a card with seed, brain, verdict and
eggs, plus a link anyone can run to try to beat it. Same paper, same grader, no favors.

## Golden runs

[`golden/seed42.json`](golden/seed42.json) holds production-verified rows. Anyone can re-run
the links and must reproduce the verdict. One seed is one row, not a theorem.

## Use the pure parts anywhere

`calibrateDeaths` and the aggregation run on any sealed decision log, no game required:

```js
import { calibrateDeaths } from "./js/bench.js";

const calib = calibrateDeaths(log, died, finalTimeLeft, 5);
// calib.buckets: [{n, deaths} ×3] by danger score 0–3
// calib.brier: mean (danger/3 − died_within_5s)² over the log
```

`contentHash` (from the vendored [`js/brain.js`](js/brain.js)) is the same hash the game
seals decisions with, so logs produced elsewhere verify here unchanged.

## The scene contract

`runBench(scene, opts)` drives any world that implements this surface (the reference
implementation is `scene-game.js` in [fruitflyworld/game](https://github.com/fruitflyworld/game)):

| The harness needs | For |
| --- | --- |
| `scene.update(t, dt)` at 60 Hz, `scene.ended` | stepping the world deterministically |
| `scene.state` (plain, deep-copyable), `scene.resetGenerationWorld()`, `scene.nextGen(cards)` | resetting between runs and generations |
| `scene.attachBrain(id, {silent})`, `scene.brainLog` | plugging the brain, reading the sealed log |
| `scene.playerFly` / `scene.rivalFly` with `.gf`, `.alive`, `.eggs`, `.deathReason` | outcomes and GF membrane state (which must be resettable — leftover membrane potential would fork the double run) |
| `scene.genTimeLeft`, `scene.predator`, `scene.visitedCells` | calibration windows and per-run residue clearing |

The harness itself is part of the exam: it clears every known source of cross-run leak
(novelty memory, clocks, predator phase, GF membrane) before each run. A world that forks
under it is a world whose `DIVERGED` you get to read.

## Honest boundaries

- The Brier score treats `dangerScore/3` as P(death within 5 s) — a convention, and
  closed-loop: it mixes perception with policy. Details on
  [/calibration](https://fruitfly.world/calibration).
- Determinism holds for the deterministic brains. A remote oracle cannot be re-run; the
  harness removes it before the exam.
- One seed is one row, not a theorem.

## Related repositories

| Repository | What it holds |
| --- | --- |
| [fruit-fly-world](https://github.com/fruitflyworld/fruit-fly-world) | The full site: game, exam room, missions, Passport contract |
| [game](https://github.com/fruitflyworld/game) | The playable game (Phaser), with this harness vendored |
| [sim](https://github.com/fruitflyworld/sim) | The simulation core and neural circuits this grades |

Vulnerabilities are reported privately through the
[main repository's security policy](https://github.com/fruitflyworld/fruit-fly-world/blob/main/SECURITY.md),
not in public issues.

## License

MIT © 2026 Fruit Fly World
