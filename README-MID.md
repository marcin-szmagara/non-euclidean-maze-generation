# (Non) euclidean maze generation

The aim of the project is to test different maze generation algorithms in non-euclidean geometries, with special focus on hyperbolic geometry.

Currently the project implements a testing framework and simple algorithms. These are then tested.

Next steps:
* implement more algorithms in the current setup
* focus on hyperbolic geometry with higher radius (much bigger mazes)
* develop agents solving the mazes to quantitatively compare maze generation performance

### Setup

We use all nodes within a configurable radius of the starting cell.
There are two node types - passable and blocking. The starting cell is always passable.
We then use chosen maze generation algorithm. Finally we place an item to mark the goal cell.

### Tested geometries

Carefully selected to represent 3 major geometries while being comparable:

* {7, 3} GP(1, 1) (bitruncated) hyperbolic, with radius = 6
* {6, 3} euiclidean, with radius = 10
* {5, 3} GP(4, 4) spherical, with radius = 12

They all generate mazes with approximately 330 cells.

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

To improve the algorithm described above we added a parameter `p`. Whenever the algorithm visits a new node, with
probability `p`, don't visit the neighbors.

Note that this algorithm doesn't guarantee any kind of maximality of the generated maze. In particular with probability `p`
it will terminate at the start node, generating nothing.

With high cutoff probabilities `p > 0.1`, the algorithm branched a lot, generating shallow trees. These were very easy to
navigate. The experience was the same across all geometries.

With `p = 0.015` things started to get more interesting.

We got branching which sometimes led to longer, non-obvious dead-ends. However most branches were short and didn't really
contribute to the gameplay.

Interestingly the spherical and hyperbolic geometries usually provided a more difficult challenge than euclidean.
