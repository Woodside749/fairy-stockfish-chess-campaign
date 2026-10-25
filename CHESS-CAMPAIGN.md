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

### 2026-10-25: the crooked bishop (Lightning)

A new Betza modifier `z` for the bishop: `zB` is a crooked bishop. It slides diagonally and
switches to the other diagonal at every step (one diagonal step, then a turn of 90 degrees,
and so on), zigzagging up, down or to either side, until it leaves the board or reaches an
occupied square, which it may capture. Chess Campaign's Lightning moves like this.

* Its moves are worked out step by step (a new rider type, `RIDER_CROOKED_BISHOP`) on the
  variant's real board size, so it never steps off the edge and back on again.
* It is a non-sliding rider, so its checks and the replies to them are always worked out in
  full. It can reach a square along two different paths, so it is left out of the quick
  pin detection (which assumes one straight line); legality is still checked in full.
* A direct check is worked out from the board after the move (its path may lead through the
  square it came from).
* Its estimated value counts it like a bishop.

Files: `types.h`, `piece.h`, `piece.cpp`, `bitboard.h`, `bitboard.cpp`, `ucioption.cpp`,
`position.cpp`, `psqt.cpp`.

### 2026-10-25: the rose

A new Betza modifier `q` for the knight: `qN` is a rose. It makes knight jumps that turn by
45 degrees after each jump, always the same way round, so they trace a circle: up to seven
jumps (the eighth would come back to its square), stopping at any of them. It jumps over
pieces like a knight, but a piece on one of its landing squares stops it there (and may be
captured). Chess Campaign's Rose moves like this.

It is built like the crooked bishop above: a new rider type (`RIDER_ROSE`) worked out step
by step on the variant's real board size, a non-sliding rider with full check detection, left
out of the quick pin detection, and valued like a nightrider.

Files: `types.h`, `piece.h`, `piece.cpp`, `bitboard.h`, `bitboard.cpp`, `position.cpp`,
`psqt.cpp`.

### 2026-10-25: immune pieces and shields (Angels and Devils)

New variant settings `immuneTypes` and `shieldTypes` (piece letters):

* Immune pieces (Chess Campaign's Angel and Devil) are as safe as a king from the effects
  above: fire (Dragons, Flames, Suns) never burns them, and they capture a Bomb or a Medusa
  without blowing up or turning to stone. They can still be captured normally.
* A shield piece (the Angel) protects its side's pieces next to it: they can't be captured
  and fire doesn't burn them. Shield pieces don't shield each other, so the Angel herself
  can always be captured.

Files: `variant.h`, `parser.cpp`, `position.h`, `position.cpp`.

### 2026-10-25: pieces that rest after moving (the Devil)

New variant setting `cooldownTypes` (piece letters): if a side's last move was made by one of
these pieces (Chess Campaign's Devil), none of them may move on that side's next turn. Each
move's state remembers whether it was made by such a piece (a null move never is). The
position alone (FEN) doesn't show this, so a GUI should give the engine the earlier moves,
e.g. `position fen <fen two moves ago> moves <move> <move>`.

Files: `variant.h`, `parser.cpp`, `position.h`, `position.cpp`.
