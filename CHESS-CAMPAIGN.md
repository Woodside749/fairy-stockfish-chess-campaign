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

### 2026-10-21: Bombs and Medusas

New variant settings `explosiveTypes` and `petrifyingTypes` (piece letters):

* A capture by or of an explosive piece (Chess Campaign's Bomb) also removes the capturing
  piece. A king may never capture one, and a capture that leaves the square empty is only
  legal if it doesn't expose the capturer's own king (lines can open through that square).
* A piece that captures a petrifying piece (Chess Campaign's Medusa) turns to stone: a wall
  on that square for the rest of the game. A king may never capture one.

Checks after such captures are worked out from the board after the move, and the static
exchange evaluation treats the capturer as lost (like the existing petrifyOnCaptureTypes).

Files: `variant.h`, `parser.cpp`, `position.h`, `position.cpp`.

### 2026-10-25: Dragons

New variant setting `breathTypes` (piece letters): when such a piece (Chess Campaign's
Dragon) captures, it also removes the opponent's pieces, except the king, on the next two
squares beyond the captured piece in the direction of the move (both squares, even if the
first one is occupied; the board edge stops it). Its own pieces are never harmed.

Legality, checks and undoing the move all account for the burned squares: a burn that opens
a line onto the mover's own king is illegal, and one that opens a line onto the opponent's
king gives check.

Files: `variant.h`, `parser.cpp`, `position.h`, `position.cpp`.

### 2026-10-25: Flames and Suns

New variant settings `flameTypes` and `sunTypes` (piece letters), using the same fire as
the Dragons above (only the opponent's pieces burn, never a king):

* A capture by a Flame also removes the opponent's piece on the next square beyond the
  captured piece, in the direction of the move.
* Every move of a Sun, capture or not, removes the opponent's pieces on the squares around
  the square it lands on. Such a move resets the fifty-move counter.

In variants with fire, the bookkeeping for removed pieces now runs for every move (not only
captures), so that undoing a move always restores the right pieces.

Files: `variant.h`, `parser.cpp`, `position.h`, `position.cpp`.
