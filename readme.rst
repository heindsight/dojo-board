Board Game for Python Dojos
===========================

Introduction
------------

Often, when running a Python Dojo, we've ended up with a challenge
based around some kind of board or tile-based landscape. In these
situations it's not uncommon to spend a lot of the time building up
your basic board functionality in order to support the more interesting
gameplay algorithm.

This module implements a general-purpose board structure which
has the functionality needed for a range of purposes, and lends itself
to being subclassed for those particular needs.

Dependencies
------------

None - stdlib only

Tests
-----

Fairly decent coverage (not actually checked with coverage.py): test.py

Getting Started
---------------

Absolutely basic usage::

    import board
    #
    # Produce a 3x3 board
    #
    b = board.Board((3, 3))

    b[0, 0] = "X"
    b[1, 0] = "O"

Usage
-----

Board is an n-dimensional board, any of which dimensions can be of
infinite size. (So if you have, say, 3 infinite dimensions, you have
the basis for a Minecraft layout). Dimensions are zero-based.

Cells on the board are accessed by item access, eg board[1, 2] or
landscape[1, 1, 10].

A board can be copied, optionally along with its data by means of the
.copy method. Or a section of a board can be linked to the original
board by slicing the original board::

    b1 = board.Board((9, 9))
    b1[1, 1] = 1
    b2 = b1.copy()
    b3 = b1[:3, :3]

Note that the slice must include all the dimensions of the original
board, but any of those subdimensions can be of length 1::

    b1 = board.Board((9, 9, 9))
    b2 = b1[:3, :3, :1]

A sentinel value of Empty indicates a position which is not populated
because it has never had a value, or because its value has been deleted::

    b1 = board.Board((3, 3))
    assert b1[1, 1] is board.Empty
    b1.populate("abcdefghi")
    assert b1[1, 1] == "e"
    del b1[1, 1]
    assert b1[1, 1] is board.Empty

Iterating over the board yields its coordinates::

    b1 = board.Board((2, 2))
    for coord in b1:
        print(coord)
    #
    # => (0, 0), (0, 1) etc.
    #

Iteration over a board with one or more infinite dimensions will work
by iterating in chunks::

    b1 = board.Board((3, 3, board.Infinity))
    for coord in b1:
        print(b1)

To see coordinates with their data items, use iterdata::

    b1 = board.Board((2, 2))
    b1.populate("abcd")
    for coord, data in b1.iterdata():
        print(coord, "=>", data)


To read, write and empty the data at a board position, use indexing::

    b1 = board.Board((3, 3))
    b1.populate("abcdef")
    print(b1[0, 0]) # "a"

    b1[0, 0] = "*"
    print(b1[0, 0]) # "*"

    del b1[0, 0]
    print(b1[0, 0]) # <Empty>

To test whether a coordinate is contained with the local coordinate space, use in::

    b1 = board.Board((3, 3))
    (1, 1) in b1 # True
    (4, 4) in b1 # False
    (1, 1, 1) in b1 # InvalidDimensionsError

One board is equal to another if it has the same dimensionality and
each data item is equal::

    b1 = board.Board((3, 3))
    b1.populate("abcdef")
    b2 = b1.copy()
    b1 == b2 # True
    b2[0, 0] = "*"
    b1 == b2 # False

    b2 = board.Board((2, 2))
    b2.populate("abcdef")
    b1 == b2 # False

To get a crude view of the contents of the board, use .dump::

    b1 = board.Board((3, 3))
    b1.populate("abcdef")
    b1.dump()

To get a grid view of a 2-dimensional board, use .draw::

    b1 = board.Board((3, 3))
    b1.populate("OX  XXOO ")
    b1.draw()


To populate the board from an arbitrary iterator, use .populate::

    def random_letters():
        import random, string
        while True:
            yield random.choice(string.ascii_uppercase)

    b1 = board.Board((4, 4))
    b1.populate(random_letters())

To clear the board, use .clear::

    b1 = board.Board((3, 3))
    b1.populate(range(10))
    b1.clear()
    list(b1.iterdata()) # []

A board is True if it has any data, False if it has none::

    b1 = board.Board((2, 2))
    b1.populate("abcd")
    bool(b1) # True
    b1.clear()
    bool(b1) # False

The length of the board is the product of its dimension lengths. If any
dimension is infinite, the board length is infinte. NB to find the
amount of data on the board, use lendata::

    b1 = board.Board((4, 4))
    len(b1) # 16
    b1.populate("abcd")
    len(b1) # 16
    b1.lendata() # 4
    b2 = board.Board((2, board.Infinity))
    len(b2) # Infinity

To determine the bounding box of the board which contains data, use .occupied::

    b1 = board.Board((3, 3))
    b1.populate("abcd")
    list(c for (c, d) in b1.iterdata()) # [(0, 0), (0, 1), (0, 2), (1, 0)]
    b1.occupied() # ((0, 0), (1, 2))

To test whether a position is on any edge of the board, use .is_edge::

    b1 = board.Board((3, 3))
    b1.is_edge((0, 0)) # True
    b1.is_edge((1, 1)) # False
    b1.is_edge((2, 0)) # True

To find the immediate on-board neighbours to a position along all dimensions::

    b1 = board.Board((3, 3, 3))
    list(b1.neighbours((0, 0, 0)))
    # [(0, 1, 1), (1, 1, 0), ..., (1, 0, 1), (0, 1, 0)]

EXPERIMENTAL: To iterate over all the coords in the rectangular space between
two corners, use .itercoords::

    b1 = board.Board((3, 3))
    list(b1.itercoords((0, 0), (1, 1))) # [(0, 0), (0, 1), (1, 0), (1, 1)]

EXPERIMENTAL: To iterate over all the on-board positions from one point in a
particular direction, use .iterline::

    b1 = board.Board((4, 4))
    start_from = 1, 1
    direction = 1, 1
    list(b1.iterline(start_from, direction)) # [(1, 1), (2, 2), (3, 3)]
    direction = 0, 2
    list(b1.iterline(start_from, direction)) # [(1, 1), (1, 3)]

Properties
----------

To determine whether a board is offset from another (ie the result of a slice)::

    b1 = board.Board((3, 3))
    b1.is_offset # False
    b2 = b1[:1, :1]
    b2.is_offset # True

To determine whether a board has any infinite or finite dimensions::

    b1 = board.Board((3, board.Infinity))
    b1.has_finite_dimensions # True
    b1.has_infinite_dimensions # True
    b2 = board.Board((3, 3))
    b1.has_infinite_dimensions # False
    b3 = board.Board((board.Infinity, board.Infinity))
    b3.has_finite_dimensions # False

Local and Global coordinates
----------------------------

Since one board can represent a slice of another, there are two levels
of coordinates: local and global. Coordinates passed to or returned from
any of the public API methods are always local for that board. They
represent the natural coordinate space for the board. Internally, the
module will use global coordinates, translating as necessary.

Say you're managing a viewport of a tile-based dungeon game where the
master dungeon board is 100 x 100 but the visible board is 10 x 10.
Your viewport board is currently representing the slice of the master
board from (5, 5) to (14, 14). Changing the item at position (2, 2) on
the viewport board will change the item at position (7, 7) on the master
board (and vice versa).

As a user of the API you don't need to know this, except to understand
that a board slice is essentially a view on its parent. If you wish
to subclass or otherwise extend the board, you'll need to note where
coordinate translations are necessary.

