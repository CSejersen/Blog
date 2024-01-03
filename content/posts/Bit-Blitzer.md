+++
title = '♟️  Bit Blitzer: Building A C++ Chess Engine From Scratch'
description = 'Using BitBoards and magic indexing for move generation, and an improved min max search algorithm with alpha beta pruning to find the best moves.'
date = 2024-01-02T12:39:38+01:00
+++

After spending more than a decade pushing pawns myself, I recently stumbled upon the concept of Bitboards in chess programming. This intruiged me, 
as representing the position with bitboards enables a large part of the computations to be done with bitwise operations and blazingly fast look-up tables. The prospect of 
theese speeds made me embark on the biggest programming challenge of my life. A full-blown chess engine written in C++.

# What is a Bit Bitboard?

Bitboards work by representing the 64 square of the chessboard with a single 64 bit binary number. Setting a bit to 1 on the Bitboard 
represents a piece on the corresponding square. Having just 1 bitboard to represent an entire position is of course not enough, 
as we have to know the type of piece on a given square. To solve this problem I use a total of 12 bitboards - 1 for every type of piece.  

If we have the starting position, visualizing the 6 white bitboards with an X for 1's and a dash for 0's looks like this:

![White bitboards start position](/bitboards_startpos.png)

In C++ I use the uint64_t type for my Bitboards, and a few useful bit manipulation functions to operate on them.

```C++
void setBit(U64 &bb, int i){
    (bb) |= (1ULL << i);
}
void clearBit(U64 &bb, int i) {
    (bb) &= ~(1ULL << i);
}
int getLSB(const U64& bb) {
    return (__builtin_ctzll(bb));
```

Using Bitboards also makes it super easy to represent entire ranks and files.

```C++
const U64 FILE_A = 0x8080808080808080ULL;
const U64 FILE_B = 0x4040404040404040ULL;

const U64 RANK_7 = 0xff000000000000ULL;
const U64 RANK_8 = 0xff00000000000000ULL;
```

The most beautiful aspect of bitboards however, come to light when we start moving the pieces around. Lets say we want to move a pawn 1 square forwards. Simply bitshift its
position by 8 and there you go. Moving a knight forwards and to the right? Easy, bitshift it by 17 and Bob's your uncle. 


# Here comes the speed!

We can pre-calculate all of these possible attacks, and store them in blazingly fast lookup-tables on startup. This is easily done for the pawns, knights and kings 
as their path cannot be blocked. Generating all knight attacks and storing them in an array only takes a few lines of code:

```C++
// Generating lookup table for knights
U64 knight = 0ULL;
for (int i = A1; i <= H8; i++) {
  // placing knight on current square
  setBit(knight, i);
  // generate attacks
  knightAttacks[i] =  (((knight >> 6) | (knight << 10)) & ~FILE_GH) |
    (((knight >> 10) | (knight << 6)) & ~FILE_AB) |
    (((knight >> 15) | (knight << 17)) & ~FILE_H) |
    (((knight >> 17) | (knight << 15)) & ~FILE_A);
  // removing knight from board
  clearBit(knight, i);

```

When it comes to the sliding pieces (rooks, bishops and queens) it get quite a bit trickier. 

# Move generation



