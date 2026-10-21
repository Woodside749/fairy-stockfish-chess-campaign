## Chess Campaign's modified version

**This is a modified version of [fairy-stockfish.wasm](https://github.com/fairy-stockfish/fairy-stockfish.wasm)** (Fairy-Stockfish for the browser, by Fabian Fichter and the Fairy-Stockfish and Stockfish authors), changed for the chess game [Chess Campaign](https://chesscampaign.com). It is not the official Fairy-Stockfish; please report problems with this version to Chess Campaign, not to the Fairy-Stockfish project.

Like the original, it is free software under the GNU General Public License, version 3 ([Copying.txt](Copying.txt)). The original authors are listed in [AUTHORS](AUTHORS).

Changes (on the `chess-campaign` branch, see [CHESS-CAMPAIGN.md](CHESS-CAMPAIGN.md)):

* 2026-10-21: a separate zone for piece promotions (`piecePromotionRegionWhite` / `piecePromotionRegionBlack`), so pieces with a `promotedPieceType` can promote somewhere other than the pawns' promotion zone. Variants that don't set it behave exactly as before.

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
