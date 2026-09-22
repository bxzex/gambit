# Gambit

A chess engine I wrote from scratch. You can play it as white or black, or just watch it play itself.

https://bxzex.github.io/gambit/

The board is 0x88, and moves are generated pseudo-legally and filtered with make/unmake. Search is alpha-beta with a transposition table, null move pruning, late move reductions, killer moves and a quiescence search. It runs in a Web Worker, so the board doesn't freeze while it thinks.

I tested the move generator with perft on the standard positions. The start position at depth 5, Kiwipete at depth 4 and positions 3, 4 and 5 all match the known counts exactly. You can run the same suite from the page.

The evaluation is simple: material, piece-square tables, passed pawns and a mop-up term so it can actually finish a won endgame.
