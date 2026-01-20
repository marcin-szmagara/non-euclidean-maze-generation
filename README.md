based on [Hyperrogue](https://github.com/zenorogue/hyperrogue)

# (Non)-euclidean maze generation

The aim of the project is to test different maze generation algorithms in non-euclidean geometries, with special focus on hyperbolic geometry.

### Progress

Currently the project implements a testing framework and a few generation algorithms. These are then tested.

Initially planed next steps:
* develop agents solving the mazes to quantitatively compare maze generation performance - won't do as the mazes we generate are small enough to completely judge by eye, no need for metrics

New next steps:
* Consider different navigation challenges than mazes, fully procedurally generated, in spirit with hyperrogue.

### Bulding

`HYPERROGUE_USE_GLEW=1 HYPERROGUE_USE_PNG=1 make`

### Setup

We use all nodes within a configurable radius of the starting cell.
There are two node types - passable and blocking. The starting cell is always passable.
We then use chosen maze generation algorithm. Finally we place an item to mark the goal cell in a random leaf.

### Tested geometries

Carefully selected to represent 3 major geometries while being comparable:

* {7, 3} GP(1, 1) (bitruncated) hyperbolic, with radius = 7
* {6, 3} euiclidean, with radius = 13
* {5, 3} GP(4, 4) spherical, with radius=inf

They all generate mazes with approximately 500 cells

### Simple backtrack

Note how we can only make vertices blocking or non-blocking, we don't control edges. This means we can't directly apply
standard graph-theory based maze generation algorithms.

For the first and simplest algorithm we chose a
DFS. It carves a path until it would create a cycle.

It creates a tree consisting of very long paths (we don't include any pictures but the experiments
should be very easy to follow given the provided menus; it's also much more pleasant to discover
the mazes interactively).

It's equally boring in all geometries - just follow the obvious long path and you will find the goal.

### Random cutoff

To improve the algorithm described above we added a parameters `p` and `n`. Whenever the algorithm visits a new node, with
probability `p`, backtrack `n` steps (if at least n steps away from start).

Experimentally parameters `p = 0.02` and `n = 10` were found to generate more interesting mazes.

However sometimes a backtrack would leave a major part of the space uncarved, thus the algorithm wasn't reliable.

Still this already allowed us to make some observations about how the mazes differ depending on selected geometry.

The long path usually split once or twice and these were the main difficulties of the mazes. In euclidean space there is a
clear sense of direction - that is you can be moving either towards the reward or away from it.
This makes the mazes relatively easy to solve by A* with straight-line distance as a heuristic.

The same cannot be said about spherical geometry. If the goal spawned on the other side of the map, any direction could be
viable. Moreover, unlike the other two geometries, we didn't need to bound the map by a radius. This means there are no boundaries
and paths often loop around almost all the way to the start. Subjectively solving some of these mazes was immersive and engaging,
much more so than the euclidean variant.

This depth-first approach was the most disappointing for the hyperbolic case. The vast majority of the cells are by the boundaries
in this geometry. In practice this meant that the path would usually create a long circle at the very end, providing little
challenge or novelty.


### Minimal spanning tree

Since it's hard to reliably get rid of the obvious long path problem using DFS, we next try a minimal-spanning-tree style algorithm.
We implement an adaptation of Kruskal's algorithm: we process the nodes in a random order, trying to turn each into grass iff it
wouldn't create a cycle.

This algorithm has the drawback that it doesn't guarantee that the graph will be connected and indeed we ran into this issue too often
to consider it viable.

As such, we chose to implement an adaptation of Prim's algorithm next. It has the nice property of keeping the maze connected at all
times.

It's known that in euclidean geometry minimal-spanning-tree-based algorithms generate mazes with lot's of short dead ends.
This is confirmed by our implementation across all geometries. The high branching factor and short paths especially underutilize
spherical geometry, rarely requiring the player to use anything beyond the common euclidean strategies.
However, the hyperbolic case presents something unique and highlights how different this style of maze is to the DFS-based one.
Instead of a long path at the border we see lots of leaves coalescing into longer progressively *thicker* branches. This makes
the mazes very easy to solve, the path is often almost straight.


### Uniform

[Supposedly](https://cse.buffalo.edu/~hungngo/classes/2003/Markov_Chains/papers/Algos/p296-wilson.pdf) if you repeatedly add paths
from random vertices through random loop-erased walks to the existing tree you get a uniformly random maze. Unfortunately just
erasing loops isn't enough in our setup to guarantee we don't create any cycles. As such it's unclear how to faithfully implement
it under our model.

Fortunately there is another algorithm which also promises to produce uniformly random trees - *Aldous-Broder*.
This one can be easily adapted.

Indeed, the generated mazes were the most unpredictable across the board.


### Remarks

* Each combination of `(algorithm, geometry)` provided a different experience.
* Non-euclidean geometries are less intuitive to navigate, posing unique challenges.
* Ultimately the size of the maze (in terms of number of nodes) best approximates its difficulty, assuming a balanced generation algorithm
