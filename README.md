## Chess Campaign's modified version

**This is a modified version of [fairy-stockfish.wasm](https://github.com/fairy-stockfish/fairy-stockfish.wasm)** (Fairy-Stockfish for the browser, by Fabian Fichter and the Fairy-Stockfish and Stockfish authors), changed for the chess game [Chess Campaign](https://chesscampaign.com). It is not the official Fairy-Stockfish; please report problems with this version to Chess Campaign, not to the Fairy-Stockfish project.

Like the original, it is free software under the GNU General Public License, version 3 ([Copying.txt](Copying.txt)). The original authors are listed in [AUTHORS](AUTHORS).

Changes (on the `chess-campaign` branch, see [CHESS-CAMPAIGN.md](CHESS-CAMPAIGN.md)):

* 2026-10-21: a separate zone for piece promotions (`piecePromotionRegionWhite` / `piecePromotionRegionBlack`), so pieces with a `promotedPieceType` can promote somewhere other than the pawns' promotion zone. Variants that don't set it behave exactly as before.
* 2026-10-21: Bombs and Medusas (`explosiveTypes`, `petrifyingTypes`): a capture by or of a Bomb also removes the capturer; a piece that captures a Medusa turns to stone. Variants that don't set them behave exactly as before.
* 2026-10-25: Dragons (`breathTypes`): a capture by a Dragon also removes the opponent's pieces (not the king) on the next two squares in the direction of the move. Variants that don't set it behave exactly as before.
* 2026-10-25: Flames and Suns (`flameTypes`, `sunTypes`): a capture by a Flame also removes the opponent's piece (not the king) on the next square in the direction of the move; every move of a Sun removes the opponent's pieces (not the king) around the square it lands on. Variants that don't set them behave exactly as before.
* 2026-10-25: the crooked bishop (`zB`, Chess Campaign's Lightning): slides diagonally and switches to the other diagonal at every step. Pieces that don't use `z` behave exactly as before.
* 2026-10-25: the rose (`qN`, Chess Campaign's Rose): knight jumps turning around a circle. Pieces that don't use `q` behave exactly as before.
* 2026-10-25: immune pieces and shields (`immuneTypes`, `shieldTypes`, Chess Campaign's Angel and Devil): immune pieces are never burned and survive capturing Bombs and Medusas; pieces next to their side's shield piece can't be captured or burned. Variants that don't set them behave exactly as before.
* 2026-10-25: pieces that rest after moving (`cooldownTypes`, Chess Campaign's Devil): after such a piece moves, none of them may move on that side's next turn. Variants that don't set it behave exactly as before.

The original README follows.

---

## fairy-stockfish-nnue.wasm

[![npm version](https://badge.fury.io/js/fairy-stockfish-nnue.wasm.svg)](https://badge.fury.io/js/fairy-stockfish-nnue.wasm)
[![CI](https://github.com/ianfab/fairy-stockfish.wasm/actions/workflows/ci.yml/badge.svg)](https://github.com/ianfab/fairy-stockfish.wasm/actions/workflows/ci.yml)

WebAssembly port of [Fairy-Stockfish](https://github.com/ianfab/Fairy-Stockfish) with NNUE support, currently used by [pychess.org](https://www.pychess.org) for client-side analysis.

See [fairy-stockfish-nnue-wasm-demo](https://github.com/ianfab/fairy-stockfish-nnue-wasm-demo) for a demo.

For development, see [`src/emscripten/README.md`](src/emscripten/README.md).

Current default branch is `nnue`.

To release a new version:
* Make sure CI passes (update reference bench if required)
* Bump version number in `src/emscripten/public/package.json`
* Create and push a tag with the name corresponding to the version number
