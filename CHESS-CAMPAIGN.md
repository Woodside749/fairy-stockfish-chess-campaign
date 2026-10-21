# Chess Campaign's changes to Fairy-Stockfish

This branch (`chess-campaign`) is fairy-stockfish.wasm with changes for the chess game
[Chess Campaign](https://chesscampaign.com). Every change is listed here, newest last. The
engine (stockfish.js / stockfish.wasm) and the rules library (ffish.js) are both built from
this source.

## Building

Engine, with Emscripten 2.0.26, from `src/`:

    make build ARCH=wasm KERNEL=Linux embedded_nnue=no -j8
    make emscripten_copy_files ARCH=wasm KERNEL=Linux embedded_nnue=no

Rules library (ffish.js, ES module), with Emscripten 3.1.45, from `src/`:

    make -f Makefile_js build es6=yes

## Changes

### 2026-10-21: a separate zone for piece promotions

New variant settings `piecePromotionRegionWhite` and `piecePromotionRegionBlack`: the
squares where pieces with a `promotedPieceType` may promote. If a variant doesn't set them,
pieces use the pawns' `promotionRegionWhite` / `promotionRegionBlack`, as before.

Chess Campaign uses it for pieces that turn into another piece when they capture, anywhere
on the board (with `piecePromotionOnCapture` and `mandatoryPiecePromotion`), while pawns
still promote only on the last rank.

Files: `variant.h`, `parser.cpp`, `position.h`, `movegen.cpp`, `position.cpp`, `evaluate.cpp`.
