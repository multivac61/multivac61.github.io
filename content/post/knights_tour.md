+++
title = "A Knight's Tour"
author = ["Olafur Bogason"]
date = 2015-10-23
lastmod = 2019-02-24T13:08:33+00:00
tags = ["ai", "python"]
categories = ["ai"]
draft = false
+++

Another interesting problem relating to classical search in AI is the Knight's Tour.

_A knight's tour is a sequence of moves of a knight on a chessboard such that the knight visits every square only once. If the knight ends on a square that is one knight's move from the beginning square (so that it could tour the board again immediately, following the same path), the tour is closed, otherwise it is open. [Wikipedia](https://en.wikipedia.org/wiki/Knight's%5Ftour)_

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">
  <div></div>

{{< figure src="https://upload.wikimedia.org/wikipedia/commons/c/ca/Knights-Tour-Animation.gif" >}}

</div>

I implemented a solution that tries to perform a open path tour and returns a list of squares the knight has to travel to from begin to end. I found no need to use the simpleai library this time. Instead used a [greedy search](https://en.wikipedia.org/wiki/Best-first%5Fsearch) that will always take the action that results in a state with the fewest actions possible, otherwise known as the Warnsdorf's rule. If it doesn't find a path in the first case it will simply return an error for the given N and start point.

```python
class KnightsTour():
    '''Knights Tour using Warnsdorf's rule as heuristics'''

    def __init__(self, N, start_point=(0,0)):
        self.N             = N
        self.initial_state = (start_point, )
        # all relative moves expressed as displacement in 2D
        self._actions      = [(1,2), (1,-2), (2,1), (2,-1),\
                              (-1,2), (-1,-2), (-2,1), (-2,-1)]

    def actions(self, s):
        '''Possible actions from a state.'''
        # generate every possible state and filter unvalid ones
        return [a for a in self._actions if self._is_valid(self.result(s, a))]

    def result(self, s, a):
        '''Result of applying an action to a state.'''
        last_x, last_y = s[-1];
        delta_x, delta_y = a

        b = list(s)     # make mutable to add next move
        b.append((last_x+delta_x, last_y+delta_y))
        return tuple(b) # immutable to search

    def _is_valid(self, state):
        last_s = state[-1]
        if last_s in state[:-1]:
            return False

        for (sx, sy) in state:
            if ( not (0 <= sx < self.N) or  not (0 <= sy < self.N) ):
                return False

        return True

    def heuristic(self, state):
        # how many actions can be performed in given state
        return len(self.actions(state))

    def go_on_a_ride(self):
        s = self.initial_state
        for i in range(self.N**2-1):
            # get all state and their respective heuristics
            d = {i: self.heuristic(self.result(s, i)) for i in self.actions(s)}
            a = min(d, key=d.get) # get the action that leads to a state
            # that has least possible actions
            s = self.result(s, a)
            if not s:
                print("Sorry no Knights tour starting from", self.start_point)
                return
            self.print_board(s)

    def print_board(self, s):
        board = s
        N     = self.N
        for i in range(N):
            for j in range(N):
                if (i,j) in board:
                    print("%3s" % s.index((i,j)), end='')
                else:
                    print('.', end='')
                    print('')

## Define the size of board NxN and the start point
problem = KnightsTour(N=8, start_point=(0,0))
problem.go_on_a_ride()
```
