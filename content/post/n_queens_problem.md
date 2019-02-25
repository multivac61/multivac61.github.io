+++
title = "The problem with N queens"
author = ["Olafur Bogason"]
date = 2015-10-21
lastmod = 2019-02-24T17:46:10+00:00
tags = ["python", "ai"]
categories = ["python"]
draft = false
+++

I'm currently taking an introductory course in AI at McGill. In reality the course is not really an introductory course as it covers a broad range of theory and is extremely demanding (as in bloody hard). That being said I feel like I'm getting a good overview of the general landscape of AI, which is awesome.

The course textbook we're using is AI: A Modern Approach. In a chapter about classical search the authors talk about a problem called the 8-Queens problem..

_The eight queens puzzle is the problem of placing eight chess queens on an 8×8 chessboard so that no two queens threaten each other. Wikipedia_

I must admit that I'm not a big chess player but I did find this to be an interesting problem in its own right. Below are a couple of implementations for solving the 8-Queens problem using the simpleai library and its built in A\* search algorithm.

One short remark before we continue: There's really nothing that says that we have to limit ourselves to a 8x8 board instead of a board of maybe 9x9 or 99x99 squares. So from now on I will talk about the N queens problem on a NxN board instead. Now let's look at implementations...


## One queen at a time {#one-queen-at-a-time}

The first implementation is the most simple and straight forward I could come up with. The algorithm goes as follows: You start out with a blank board represented by a NxN array in python which we call our initial state. The set of actions we can do to generate a new state is to place a queen at some point (i, j) on the board where 0 <= i, j < N. You then run A\* search on the initial state.

At each state the algorithm will place one queen on the board and makes sure that it isn't being attacked. We keep on placing new queens on the board until the board has been filled with N queens where none of them are under attack. The heuristic I'm using is very simple: calculate how many queens we have left to place on the board at the given state, pretty simple.

```python
class NQueensSquare(SearchProblem):
    def __init__(self, N):
        super(NQueensSquare, self).__init__()
        self.N = N
        self.initial_state = tuple([tuple([0 for i in range(N)]) for j in range(N)])
        self._actions      = [(i, j) for i in range(N) for j in range(N)]

    def actions(self, s):
        '''Possible actions from a state.'''
        # generate every possible state then filter out invalid
        tmp = [a for a in self._actions if self._is_valid(self.result(s, a))]
        shuffle(tmp)
        return tmp

    def result(self, s, a):
        '''Result of applying an action to a state.'''
        (i, j) = a
        b      = list(s)    # make immutable object to place queen
        tmp    = list(b[i])
        tmp[j] = 1          # place queen to square (i,j) in board

        s    = list(s)      # extra steps for immutability
        s[i] = tuple(tmp)   # required by A* search in simpleai
        return tuple(s)

    def is_goal(self, state):
        '''Goal: N queens on board and no queen under attack.'''
        return self._num_queens_on_board(state) == self.N

    def heuristic(self, state):
        return self.N - self._num_queens_on_board(state)
```

At each time the A\* search generates a new state it tries out N^2 different placements of queens, in the worst case. This approach is very slow and we aren't taking advantage of the structure of the problem at all


## All queens on board and swap {#all-queens-on-board-and-swap}

Well what about instead of starting out with an initial state that corresponds to an empty NxN board, we place N queens randomly on the board so that we have one queen in each row of the board? We would start out in a state that potentially is not valid since there is a good chance that we will have some queens attacking each other. That's just fine because we only need to swap queens on the board until we have a legal state with no queen under attack.

To generate new states we swap two queens so that the resulting state will have fewer queens under attack than the prior state. The heuristic we use is the sum of all attacks (i.e. if a queen is under attack that would result in a +2 added to the sum etc.).

```python
class NQueensSwap(SearchProblem):
    def __init__(self, N):
        self.N = N
        init               = [i for i in range(N)]
        shuffle(init)   # randomize the initial array
        self.initial_state = tuple(init)    # states must me immutable
        self._actions      = [ (i, j) for i in range(N)
                                      for j in range(i,N)
                                          if i != j ]

    def actions(self, s):
        '''Possible actions from a state.'''
        # generate every possible state then filter out invalid
        tmp = [a for a in self._actions if self._is_valid(self.result(s, a))]
        shuffle(tmp) # randomize actions at each step
        return tmp

    def result(self, s, a):
        '''Result of applying an action to a state.'''
        b          = list(s)     # make board mutable to swap queens
        (i, j)     = a
        b[i], b[j] = b[j], b[i] # swap queens
        return tuple(b)         # make immutable again to search

    def is_goal(self, state):
        '''Goal: N queens on board and no queen under attack.'''
        return self.heuristic(state) == 0

    def heuristic(self, state):
        # scan the state horizontally, vertically and diagonally (left & right)
        # return the number of attacks summed for all queens
        nattacks = self._num_attacks
        return nattacks(state, (0, 1)) + nattacks(state, (1, 0)) + \
            nattacks(state, (1,-1)) + nattacks(state, (1, 1))

    def _is_valid(self, s):
        '''Check if a state is valid.'''
        # all states are valid by default
        return True
```

This approach naturally sounds like a better idea than placing one queen at a time, but I would argue that we haven't started to take any real advantage of the structure of the problem. Let's go and do something about that!


## One row at a time {#one-row-at-a-time}

We know that each row and column must have exactly one queen present. That essentially means that we don't really need to think about every single square on the board when placing a queen. Instead we can add a queen at the i-th index in a row, that is not present in the board. This brings our set of actions we can make at each state down from N^2 to N (in the worst case). That makes a huge difference in running time as we will see.

This implementation works by starting out with an empty board, expressed as an empty tuple in Python. When a new state is generated we append a row containing exactly one queen, to the previous state. We can think of this as adding a queen to the row that is nearest to the top of the NxN board which contains no queen yet. The heuristics is the same we used for the previous implementation. We continue adding queens to the board, one row at a time, until the board is full and no queen is under attack.

```python
class NQueensRow(SearchProblem):
    def __init__(self, N):
        super(NQueensRow, self).__init__()
        self.N             = N
        self.initial_state = ()
        self._actions      = range(N)
        shuffle(self._actions) # randomize actions

    def actions(self, s):
        '''Possible actions from a state.'''
        # generate every possible state then filter out invalid
        tmp = [a for a in self._actions if self._is_valid(self.result(s, a))]
        shuffle(tmp) # randomize actions at each step
        return tmp

    def result(self, s, a):
        '''Result of applying an action to a state.'''
        b = list(s)     # make mutable to add
        b.append(a)
        return tuple(b) # needs to be immutable again to search

    def is_goal(self, state):
        '''Goal: N queens on board and no queen under attack.'''
        return self.N == len(state)

    def heuristic(self, state):
        return self.N - len(state)

    def _is_valid(self, s):
        '''Check if a state is valid.'''
        # valid states: any arrangement of N queens with none attacking each other
        # check horizontal, vertical, right/left diagonals
        attacked = self._attacked
        return attacked(s, (1, 1)) and attacked(s, (1,-1)) and \
            attacked(s, (0, 1)) and attacked(s, (1, 0))
```


## Time comparison of implementations {#time-comparison-of-implementations}

<div align="center">
<iframe width="600" height="500" frameborder="0" scrolling="no" src="https://plot.ly/~horigome/97.embed"></iframe>
</div>
<br>

<div align="center">
<iframe width="600" height="500" frameborder="0" scrolling="no" src="https://plot.ly/~horigome/83.embed"></iframe>
</div>
<br>

<div align="center">
<iframe width="600" height="500" frameborder="0" scrolling="no" src="https://plot.ly/~horigome/44.embed"></iframe>
</div>
<br>

For each N I sampled 4 turns and took the average of their runtimes.

It's very difficult to say anything solid about this time comparison due to the way in which possible actions are randomized at each step of the algorithm. I can however say that by essentially adding knowledge to our implementation (by exploiting the problem structure) we were able to make the search go significantly faster than the most naive approach (compare NQueensSquare to NQueensRow).

In this post we went through three different solution implementation for the N-Queen problem and saw how important representing a problem in a smart way is when running A\* search. If you're interested in seeing more code, go get it at my Github page

I'm sure there must be faster ways to solve the N Queen problem and my implementations were merely meant as an exercise in thinking about problem representation and then coding it up in Python. If you have implemented or know of a faster way to do this or have anything to add just comment below.
