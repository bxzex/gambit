# Gambit

A chess engine written from scratch. The legal move generator is checked against
the standard perft counts. The search is alpha-beta with a transposition table
and runs in a Web Worker, so the board stays responsive while it thinks.

Live: https://bxzex.github.io/gambit/

## The board

The board is **0x88**: 128 cells, of which only the left half of each row of 16
is real. Stepping off the board always sets bit `0x88`, so bounds checks are a
single AND. Moves are packed into one integer: from, to, promotion piece and
flags for capture, double push, en passant and castling.

The generator produces pseudo-legal moves. `make` plays the move, and if it
leaves your own king attacked, it undoes the move and returns `false`. Every
`make` saves what `unmake` needs to restore the position exactly: the captured
piece, castling rights, en passant square, halfmove clock and hash.

Positions are hashed with **Zobrist keys** kept in two 32-bit halves. Each
`make` updates the hash incrementally. One half indexes the transposition
table and the other is the lock that confirms a hit.

## Proving the move generator

Perft counts every leaf at a fixed depth. The published counts for these
positions were computed independently by many engines, so a single missing
en passant or a castle through check shows up as a wrong number. Gambit matches
every one:

| Position | Depth | Expected | Gambit |
|---|---|---|---|
| Start | 5 | 4,865,609 | 4,865,609 |
| Kiwipete | 4 | 4,085,603 | 4,085,603 |
| Position 3 | 5 | 674,624 | 674,624 |
| Position 4 | 4 | 422,333 | 422,333 |
| Position 5 | 4 | 2,103,487 | 2,103,487 |

Kiwipete packs castling, pins, promotions and en passant into one position.
Position 3 tests en passant captures that would expose the king along a rank.
Position 4 is the mirror-image trap for promotions and castling rights.

Each run also confirms the position afterwards: after every perft, the FEN and
a full recomputation of the Zobrist hash must match the start. That proves
`unmake` puts everything back, including the parts perft never looks at
directly.

You can run the suite from the page. It takes well under a second in Chrome.

## The search

- Iterative deepening negamax with **principal variation search**
- **Transposition table**: 2^20 entries holding the best move, score, depth
  and bound type. Mate scores are stored relative to the node, so they stay
  correct at any ply.
- **Null move pruning**, skipped when in check and in pawn-only endgames,
  where zugzwang makes it unsound
- **Late move reductions** for quiet moves ordered late, with a re-search at
  full depth when a reduced move beats alpha
- Move ordering: TT move first, then captures by MVV-LVA, queen promotions,
  two killer slots per ply, then a history table
- **Quiescence search** over captures and promotions, with delta pruning
- Check extensions, mate distance pruning, and draw detection for repetition,
  the fifty move rule and insufficient material

Evaluation is material plus piece-square tables. The king has separate
middlegame and endgame tables, blended by a game phase counted from the
remaining non-pawn material. On top of that it scores passed pawns by rank,
doubled pawns and the bishop pair. In won endgames a mop-up term drives the
losing king to the edge, so Gambit converts king and queen against king
instead of shuffling. Given a bare king and queen, it announces a forced mate
in 6 and plays it out.

## Notes

One HTML file. No libraries, no build step. The engine source is written once
and runs twice: on the page for legal moves and SAN, and in two workers (one
for search, one for perft) built from the same text as a Blob.

Built by [bxzex](https://bxzex.com).
