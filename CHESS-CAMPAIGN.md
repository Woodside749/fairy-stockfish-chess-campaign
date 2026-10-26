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

### 2026-10-25: Zombies

New variant setting `zombieTypes` (piece letters): when such a piece (Chess Campaign's
Zombie) captures, the captured piece (as it was before any promotion) goes to the capturer's
pocket, to be dropped later; captures of `zombieExcludedTypes` (piece letters, default:
pawns) and of promoted pawns don't. Other pieces' captures are unaffected. Used with `pieceDrops = true` and a drop region (Chess
Campaign: each side's first rank).

Files: `variant.h`, `parser.cpp`, `position.h`, `position.cpp`.

### 2026-10-25: Ghosts

New variant setting `ghostTypes` (piece letters): a side's own pieces move, capture and
attack through its Ghosts as if they weren't there (but can't stop on their squares; pawns
may double-step over them); for the opponent a Ghost blocks like any piece and can be
captured. Ghosts of the same side see through each other.

* Each side's moves and attacks use the board without its own Ghosts (`attacks_from`,
  `moves_from`, `attackers_to`, check squares, pin detection).
* For a move that hasn't been made yet, `ghosts_after` gives where a side's Ghosts will be
  (legality and giving check).
* With Ghosts, one move can stop two checks at once (capturing a Ghost that gives check and
  that another piece checks through), and a Ghost that captures a blocker of its own side's
  line stays see-through there; so in these variants check evasions and discovered checks
  are always worked out in full.

Files: `variant.h`, `parser.cpp`, `position.h`, `position.cpp`, `movegen.cpp`.

### 2026-10-25: Undead

New variant setting `undeadTypes` (piece letters): however such a piece (Chess Campaign's
Undead, Death Knight and Lich) leaves the board (captured, burned or blown up), it goes back
to its owner's pocket, to be dropped again (even when a Zombie captures it). One that turns
to stone stays a wall. Used with `pieceDrops = true` and a drop region (Chess Campaign: each
side's first rank).

Files: `variant.h`, `parser.cpp`, `position.h`, `position.cpp`.

### 2026-10-25: Mimics

New variant setting `mimicTypes` (piece letters): when such a piece (Chess Campaign's Mimic)
captures and survives (it doesn't blow up or turn to stone), it becomes the piece it
captured, in its own colour, for good (even a pawn). Giving check is worked out for the piece
it becomes, and where Ghosts are after the move counts a Mimic that became a Ghost.

Files: `variant.h`, `parser.cpp`, `position.h`, `position.cpp`.

### 2026-10-25: the tornado, and Rock Magicians

* A new Betza combination `qB`, a circular bishop (Chess Campaign's Tornado): it slides two
  squares diagonally, turns by 90 degrees (always the same way round) and slides two more,
  around a diamond, stopping on any square on the way; a piece on the way stops it. It is a
  new rider type (`RIDER_TORNADO`) like the rose, but its moves aren't symmetric (a square it
  reaches may not reach it back), so attacks on a square are worked out from each tornado
  itself.
* New variant setting `rockTypes` (piece letters): instead of moving, a Rock Magician may use
  its side's move to put a rock (a wall for the rest of the game) on an empty square next to
  it. Such a move is a pass by the magician with the rock's square as its gating square,
  written `d4d4,d4e5` (SAN `Md4,e5`). Putting a rock down never gives check, may block one,
  and two of them in a row don't end the game (unlike two passes).
* 2026-10-25, later: a rock may not complete a cage of walls (rocks, stone walls, the
  board's edge) around either king.
* 2026-10-25, later: `rockLimit` (a number, 0 = no limit): how many rocks each side may put
  down in a game. A FEN doesn't show the count, so it comes from the moves played (a GUI
  should send the whole game: `position fen <start> moves ...`).

Files: `types.h`, `piece.h`, `piece.cpp`, `bitboard.h`, `bitboard.cpp`, `psqt.cpp`,
`variant.h`, `parser.cpp`, `position.h`, `position.cpp`, `movegen.cpp`, `uci.cpp`,
`apiutil.h`.

### 2026-10-25: Archers and Checkers

New variant settings `shooterTypes` and `checkerTypes` (piece letters):

* An Archer's captures are shots: the captured piece is removed and the archer stays on its
  square (Chess Campaign's Archer is `mKcnDcnA`: steps like a king, shoots 2 squares away
  over an empty square). Legality and checks are worked out with the archer still on its
  square; a direct check is worked out from the square it moves to (its line of fire may
  cross the square it left).
* A Checker also captures like in draughts: it jumps diagonally over an enemy piece (not a
  king) right next to it onto the empty square behind, removing the piece it jumped over.
  Such a move has its own move type (SPECIAL, from and to two squares apart diagonally) and
  counts as a capture; the captured square is remembered like en passant's. One jump can
  stop a double check (capturing one checker and landing in the other's way), so with
  Checkers non-king moves are generated in double check too.
* Neither a shot nor a jump makes the shooter or jumper blow up or turn to stone.

Files: `variant.h`, `parser.cpp`, `position.h`, `position.cpp`, `movegen.cpp`.

### 2026-10-26: rebirth (the Phoenix)

New variant setting `rebirthPieceType` (pairs like `c:a`): however such a piece leaves the
board (captured, burned, blown up; even by a Zombie), the other piece goes to its owner's
pocket, to be dropped later (Chess Campaign's Phoenix comes back as an Egg, which can hatch
into a Phoenix again). One that turns to stone stays a wall.

Files: `variant.h`, `parser.cpp`, `position.cpp`.

### 2026-10-26: a wider shield

A shield piece (the Angel) now protects its side's pieces on every square it attacks (for
the Angel: along its queen lines and knight jumps), not only the squares next to it.

Files: `variant.h`, `position.h`.
