## Shortest Path

The Shortest Path problem consists on finding a path between two pairs of nodes in which the sum of the weights is minimized. For a general graph, this problem is NP-hard. For some kind of graphs, like trees or directed acyclic graphs, this problem can be solved in linear time.

In this chapter, we will present multiple algorithms to solve the shortest path problem, including case studies for some of them. Most of the algorithms find the solution for one single source to all other nodes. One finds the solution for a single source to a single sink, and one resolves the shortest path between all pairs of nodes. The following table is an overview of the algorithms.

| Algorithm name                                            | Solution for  |
| --------------------------------------------------------- | ------------- |
| `AIBFS`: Shortest Path by Breadth First Search            | single-source |
| `AIDFS`: Shortest Path by Depth First Search              | single-source |
| `AIDijkstra`: Dijkstra's algorithm                        | single-source |
| `AIStar`: A* search algorithm                             | single-pair   |
| `AIShortestPathInDAG`: Sh. Path in Directed Acyclic Graph | single-source |
| `AIBellmanFord`: Bellmann-Ford algorithm                  | single-source |
| `AIFloydWarshall`: Floyd-Warshall algorithm               | all-pairs     |

Most of these algorithms have restrictions regarding the type of graph they can manage. The algorithms may be constrained to handle no edge weights, nonnegative edge weights or directed acyclic graphs. 

Let's see in the next section some examples about the suitability of each algorithm.

### Choice of the search algorithm

Beyond efficiency considerations, the choice of the search algorithm to find the shortest path in a graph must depend on the type of the graph, since not every algorithm can handle any graph.

Let us take the graph in Figure *@short_distance_graph@* as an example.  As it is an unweighted graph, we can calculate the distance between two nodes using the BFS (Breadth First Search) algorithm.

![Short distance graph.](figures/generic_example.pdf width=48&label=short_distance_graph)

But, if we add weights to the graph, as in Figure *@short_distance_graph_2@*, we cannot no longer use BFS. Instead, we can find the shortest distance using Dijkstra's or the A* search algorithm.

![Short distance with weights.](figures/dag.pdf width=48&label=short_distance_graph_2)

Dijkstra's and the A* search algorithm do not work on graphs with negative weights like in Figure *@short_distance_graph_3@*. So, if we add negative weights to the graph, we must use the Bellman-Ford or the Floyd-Warshall algorithm to solve the problem.

![Short distance with negative weights.](figures/longest_path.pdf width=48&label=short_distance_graph_3)

In case of a Directed Acyclic Graph (DAG), a graph with no cycles like in Figure *@short_distance_graph_3@*, we can use an algorithm based on topological sort to find the shortest distance. This algorithm works for both negative and positive weights as long as the graph has no cycles. This algorithm is better than others in terms of time complexity but is more restricted as it only runs on DAG. 

### Shortest path by BFS

If the graph is *unweighted* or all edges have the *same non-negative* weight, the shortest path can be found in linear time $O(V + E)$ using the **Breadth First Search** \(BFS\) algorithm.

BFS is an algorithm for traveling a graph in a traversal way. That means that the algorithm will travel the children of the starting node always in order ensuring that when the goal node is found, the path will be the shortest one possible.

BFS is a single source shortest path algorithm. That means that before running the algorithm, it is needed to specify a starting node. Then, the algorithm can tell us the shortest path between the starting node and all the other nodes.

The algorithm is the following one:

```
initialize a queue Q
mark start node as visited
Q.addLast(start)
while Q is not empty do:
  node := Q.removeFist()
  if node is the end then:
    return node
  for adjacent nodes of node do:
    if adjacentNode is not visited then:
      mark adjacentNode as visited
      Q.addLast(adjacentNode)
```

In the Pharo implementation, we use a linked list as a queue. We use a `LinkedList` instead of an `OrderedCollection` because the `removeFirst` operation of `LinkedList` takes constant time and for an `OrderedCollection`, it takes linear time.

We define a new subclass of `AIGraphAlgorithm` named `AIBFS`.
And we define a new method `run` as follows:

```
AIBFS >> run

    | node neighbours queue |
    queue := LinkedList with: start.
    start visited: true.

    [ queue isNotEmpty ] whileTrue: [
        node := queue removeFirst.
        neighbours := node adjacentNodes.

        neighbours do: [ :next |
            next visited ifFalse: [
                queue addLast: next.
                next visited: true.
                next previousNode: node ] ] ]
```

After running the algorithm, to reconstruct the shortest path between the start and the end node, we use the following method:

```
AIBFS >> reconstructPath

    | path previous |
    "If no path exists between the start and the end node"
    end previousNode ifNil: [ ^ #( ) ].

    path := LinkedList empty.
    previous := end.
    path addFirst: end model.
    [ previous = start ] whileFalse: [
        previous := previous previousNode.
        path addFirst: previous model ].
    ^ path
```

#### Case study

![A graph for BFS.](figures/bfs.pdf width=40)

For the BFS graph, the shortest path can be calculated using the class `AIBFS` like shown in the following example.

```
nodes := $a to: $i.
edges := #( #( $a $b ) #( $b $c ) #( $c $d ) #( $d $e ) #( $e $a )
            #( $b $e ) #( $e $b ) #( $e $f ) #( $f $g ) #( $g $h )
            #( $h $f ) #( $g $i ) #( $i $g ) ).
bfs := AIBFS new.
bfs
    nodes: nodes;
    edges: edges from: #first to: #second.

path := bfs runFrom: $a to: $g
```

The `path` variable contains all the nodes that are part of the path, if we inspect the variable we see:

```
#($a $b $e $f $g)
```

If we want to get the shortest path between the same starting node *A* and some other node, there is no need of rerunning the algorithm. We only need to change the end node and call the method reconstruct path.

```
bfs end: $d.
pathToD := bfs reconstructPath
```

### Shortest path by DFS

The shortest single-source path in an unweighted graph can also be searched by **Depth First Search** (DFS) instead of Breadth First Search (BFS). Both methods set the root of the search tree to the start node and grow it by adding the successors of the tree’s current leaves. In that way, DFS and BFS cover the whole graph until they find the goal node or exhaust the graph.

BFS grows the search tree layer by layer. In contrast, DFS adds to the search tree a child of the most recently included node with at least one unincluded child. So, DFS adds the start node, then its child, then its grandchild, and so on. For that reason, DFS increases the depth of the search tree in each step as much as it can. Then, if there aren’t more children of a node to add, it *backtracks* to the node’s parent.

A main difference between the two approaches is that BFS uses a queue (FIFO) for the pending nodes, while DFS uses a stack (LIFO) as we see in `AIDFS >> run`:

```
AIDFS >> run

    | node unvisitedNeighbor stack |
    self resetValues.

    stack := LinkedList new.
    start visited: true.
    stack addLast: start.

    [ stack isNotEmpty ] whileTrue: [
        node := stack last.
        unvisitedNeighbor := node adjacentNodes
                                 detect: [ :next | next visited not ]
                                 ifNone: [ nil ].
        unvisitedNeighbor
            ifNotNil: [ "Process unvisited neighbor"
                unvisitedNeighbor visited: true.
                unvisitedNeighbor previousNode: node.
                stack addLast: unvisitedNeighbor ]
            ifNil: [ "Backtrack when no more unvisited neighbors"
                stack removeLast. ] ]
```

Like in the case of `AIBFS`, the method `AIDFS >> reconstructPath` reconstructs the gathered shortest path between the start and the end node after running the algorithm.

The choice between BFS and DFS shortest path search depends on the structure of the graph and the problem requirements. Theoretically, both algorithms have linear time complexity $O(V + E)$. However, BFS is ideal for finding the shortest path in unweighted graphs because it explores all nodes at the current level before moving deeper (farther). Nevertheless, DFS uses less memory on deep but narrow graphs, as it only needs to store a single branch at a time.

### Dijkstra's algorithm

The Dijkstra's algorithm is one of the most-know algorithms for calculating the shortest path in a weighted graph. As BFS, this algorithm is also a single-source shortest path algorithm. In its naive implementation,  it has a time complexity of $O(V^2)$. But, it can be optimized using a heap or a Fibonacci heap as a data structure to a time complexity of $O((V+E)*log V)$. If a Fibonacci heap is used we get the best possible time complexity $O(E + V * log V)$ possible \(at the moment\). Dijkstra's algorithm can handle a graph with cycles, but it cannot handle negative weights.

The algorithm idea is:

1. Mark all nodes as unvisited.
2. Assign to every node infinity as the distance value. Set it to zero for the initial. Set the initial node as current.
3. Consider all of unvisited neighbours of the current node and calculate their distances through the current node. Compare the newly calculated tentative distance to the current assigned value and assign the smaller one.
4. When all of the unvisited neighbours of the current node are checked, mark the current node as visited.
5. Select the unvisited node that is marked with the smallest tentative distance, set it as the new current node, and go back to step 3.

Depending on the data structure that is chosen, the time complexity will vary.
Using an array is the most inefficient one because to get the most promising pair, it is necessary to travel all the array $O(N)$. But with a heap, or a Fibonacci heap, the time complexity for getting the most promising pair is in the logarithmic order. Our implementation uses an array for now.

The implementation in Pharo is the following:

```
AIDijkstra >> run

    | pq |
    pq := self newPriorityQueue.
    pq add: start -> 0.

    [ pq isNotEmpty ] whileTrue: [
        | assoc node minWeight |
        assoc := self removeMostPromisingPair: pq.
        node := assoc key.
        minWeight := assoc value.
        node visited: true.

        "Skip if the path weight is less than the one obtained from the pq.
        This is an optimization for not processing unnecessary nodes."
        node pathDistance < minWeight ifFalse: [
            node outgoingEdges do: [ :edge |
                edge to visited ifFalse: [
                    | newDistance |
                    newDistance := node pathDistance + edge weight.

                    newDistance < edge to pathDistance ifTrue: [
                        self updateDistance: newDistance of: edge to previousNode: node.
                        pq add: edge to -> newDistance ] ] ] ] ]
```

```
AIDijkstra >> updateDistance: newDistance of: aNode previousNode: previousNode

    aNode previousNode: previousNode.
    aNode pathDistance: newDistance
```

In this implementation, we will use an `OrderedCollection` as a data structure for the priority queue.

```
AIDijkstra >> newPriorityQueue
    "This is the naive implementation of the data structure."

    ^ OrderedCollection new
```

```
AIDijkstra >> removeMostPromisingPair: aPriorityQueue
    "This is the naive implementation of the data structure."

    | minValue |
    minValue := aPriorityQueue detectMin: [ :assoc | assoc value ].
    ^ aPriorityQueue remove: minValue
```

#### Case study

![Dijkstra graph.](figures/dijkstra.pdf width=50&label=dij)

In the graph shown in Figure *@dij@*, the shortest path between node *A* and *B* is `#( $A $C $B )`. The shortest path between node *A* and node *F* is `#( $A $C $B $D $E $F )`

```
nodes := $A to: $F.
edges := #( #( $A $B 5 ) #( $A $C 1 ) #( $B $C 2 ) #( $B $E 20 )
            #( $B $D 3 ) #( $C $B 3 ) #( $C $E 12 ) #( $D $C 3 )
            #( $D $E 2 ) #( $D $F 6 ) #( $E $F 1 ) ).

dijkstra := AIDijkstra new.
dijkstra nodes: nodes.
dijkstra
    edges: edges
    from: #first
    to: #second
    weight: #third.

shortestPathAToB := dijkstra runFrom: $A to: $B.
pathDistanceAToB := (dijkstra findNode: $B) pathDistance.

dijkstra end: $F.
shortestPathAToF := dijkstra reconstructPath.
pathDistanceAToF := (dijkstra findNode: $F) pathDistance.

dijkstra reset.
shortestPathBToE := dijkstra runFrom: $B to: $E.
```

### A* search algorithm

The A* search algorithm builds on the principles of Dijkstra’s shortest path algorithm to provide a faster solution when faced with the problem of finding the shortest path between two nodes. It achieves this by introducing a **heuristic** element to help decide the next node to consider as it moves along the path.

Compared to Dijkstra's algorithm, the A* algorithm only finds the shortest path from a specified source to a specified goal (sink), and not the shortest path tree from a specified source to all possible goals. This is a necessary trade-off for using a specific goal-directed heuristic. For Dijkstra's algorithm, since the entire shortest path tree is generated, every node is a goal, and there can be no specific goal-directed heuristic.

 A* evaluates paths using the following components:

- $g(n)$: The actual cost from the start node to node n.

- $h(n)$: A heuristic estimate of the cost from node n to the goal.

- $f(n) = g(n) + h(n)$: The total estimated cost of reaching the goal via n.

It is very important that the heuristic function $h(n)$ satisfies the following properties:

- *Admissibility*: the function never overestimates the true cost to reach the sink.

- *Consistency (monotonicity)*: the estimated cost should not suddenly drop unexpectedly.

The heuristic function is *problem-specific*. For example, if the cost relates to a distance, it could be estimated by calculating the euclidean, the Manhattan, the diagonal or the spherical distance. 

With an appropriate heuristic, the A* search algorithm ensures a time complexity of $O(E∗logV)$. As in the case of Dijkstra's algorithm, it assumes nonnegative edge weights. 

The algorithm main steps are as follows: In each iteration, A* selects the node with the lowest $f(n)$ from the open set, expands it, and updates its neighbors’ costs. If a neighbor’s tentative cost is lower than a previously recorded value, it is updated (and linked via a parent pointer for path reconstruction) and added to the open set. This process continues until the sink is reached or no path exists.

Here is the implementation of `AIStar >> run`:

```
    AIStar >> run

    | pq cameFrom gScore fScore gs fs |

    cameFrom := Dictionary new.
    gScore := Dictionary new.
    gScore at: start put: 0.
    fScore := Dictionary new.
    fScore at: start put: (self heuristicFrom: start to: end).

    gs := [ :p | gScore at: p ifAbsent: Float infinity ].
    fs := [ :p | fScore at: p ifAbsent: Float infinity ].

    pq := SortedCollection sortUsing: [ :a :b | (fs value: a) < (fs value: b) ].
    pq add: start.

    [ pq isEmpty ] whileFalse: [ 
        | current |
        current := pq removeFirst.

        current = end 
            ifTrue: [
                | path prev |
                path := OrderedCollection with: current.
               prev := cameFrom at: current ifAbsent: nil.
               [ prev isNotNil ] whileTrue: [ 
                    path addFirst: prev.
                    prev := cameFrom at: prev ifAbsent: nil.
                ].
                   ]
            ifFalse: [ 
                current outgoingEdges do: [ :edge |
                    | tentative_gScore |
                    tentative_gScore := (gs value: current) + (edge weight).
                    tentative_gScore < (gs value: (edge to))
                        ifTrue: [ 
                            "This path to neighbor is better than any previous one. Record it!"
                            self updateDistance: tentative_gScore of: edge to previousNode: current.
                        cameFrom at: (edge to) put: current.
                        gScore at: (edge to) put: tentative_gScore.
                        fScore at: (edge to) put: tentative_gScore + (self heuristicFrom: (edge to) to: end).
                        (pq includes: (edge to)) ifFalse: [ pq add: (edge to) ].
                        ] 
                ].
            ].
    ]
```

A `SortedCollection` is used for efficient node selection based on f-scores. Alternatively, a priority queue (`Heap`) may be preferred.

The method `AIStar >> heuristicFrom: startModel to: endModel` should be adapted according to the specific problem domain. By default, our implementation counts the number of nodes between intermediate node and target sink, ignoring the weight. This can be achieved with BFS linear time complexity.

### Shortest path in DAG

If the graph is a weighted **directed acyclic graph** \(DAG\), we can calculate the shortest path using an algorithm based on topological sort. Using this algorithm we have a time complexity of $O(V + E)$.

This algorithm provides also a single source solution. The idea of the algorithm is to order the nodes in a topological order, using the topological sort algorithm. Then, keep track of the path weight from the start node to the other nodes.
Then, start popping the nodes in order and store in a collection the ones that have the lowest path weight.

As this algorithm runs in graphs that has no cycles, it has no difficulty to handle negative weights. The pseudocode is:

1. Initialize the initial distance to every node to be infinity and the distance of the start node to be 0.
2. Create a topological order of all nodes.
3. For every node u in topological order:
- Do the following for every adjacent node v of u
  - `if (v pathWeight > u pathWeight + weight(u, v)) then v pathWeight: u pathWeight + weight(u, v)`

#### DAG shortest path implementation

The Pharo implementation is as follows.

```
AIShortestPathInDAG >> initializePathWeights

    nodes do: [ :node | node pathWeight: Float infinity ].
    start pathWeight: 0
```

```
AIShortestPathInDAG >> run

    | topSorter stack sortedNode |
    self initializePathWeights.
    topSorter := AITopologicalSorting new
    addNodesFromDifferentGraph: nodes;
    yourself.
    topSorter run.
    stack := topSorter topologicalSortedElements.

    [ stack isNotEmpty ] whileTrue: [
        sortedNode := self findNode: stack removeFirst.
        sortedNode outgoingEdges do: [ :nextEdge |
            nextEdge to pathWeight >
                    (sortedNode pathWeight + nextEdge weight)
                ifTrue: [
                    nextEdge to pathWeight: sortedNode pathWeight +
            nextEdge weight.
                    nextEdge to previousNode: sortedNode ] ] ]
```

#### DAG shortest path refactored

Now we are ready to refactor our code.

```
AIShortestPathInDAG >> run

    | stack sortedNode |
    self initializePathWeights.
    stack := self topologicalSortedNodes.

    [ stack isNotEmpty ] whileTrue: [
        sortedNode := self findNode: stack removeFirst.

        sortedNode outgoingEdges do: [ :nextEdge |
            nextEdge to pathDistance >
      (sortedNode pathDistance + nextEdge weight)
                ifTrue: [
          self updatePathDistance: nextEdge previousNode: sortedNode ] ] ]
```

```
AIShortestPathInDAG >> topologicalSortedNodes

    | topSorter |
    topSorter := AITopologicalSorting new
        addNodesFromDifferentGraph: nodes;
    yourself.
     topSorter run.
     ^ topSorter topologicalSortedElements.
```

```
AIShortestPathInDAG >> updatePathDistance: edge previousNode: previousNode

    edge to pathDistance: previousNode pathDistance + edge weight.
    edge to previousNode: previousNode
```

#### Case study

For the weighted DAG in Figure *@weight@*, the following snippet calculates the shortest path between node A and node F.

```
nodes := $A to: $G.
edges := #( #( $A $B 1 ) #( $B $C 5 ) #( $B $E 11 ) #( $B $D 8 )
            #( $D $E 6 ) #( $E $F 7 ) #( $G $D 4 ) ).
shortestPathInDAG nodes: nodes.
shortestPathInDAG
    edges: edges
    from: #first
    to: #second
    weight: #third

pathAtoF := shortestPathInDAG runFrom: $A to: $F.
```

![DAG with weigthed paths.](figures/dag.pdf width=50&label=weight)

### Bellman-Ford algorithm

In Dijkstra's algorithm, when a node is marked as visited, the algorithm already found the best distance to it, because adding any positive numbers will only increase the path distance. But when we are dealing with negative numbers, the assumption is not true.

The Bellman-Ford algorithm can handle *negative weighted* graphs. It runs in $O(V*E)$ time. It is also a single-source shortest path algorithm. The logic behind the algorithm is to perform at worst $V-1$ times an edge relaxation. Relaxing an edge means to update the value of the distance from the starting node to the node to which to edge goes. Then, run the algorithm one more time, if an edge can reduce its distance \(be relaxed\) means that the node to which the edge goes is part of a negative cycle.

A **negative cycle** is a cycle where the sum of all edge weights is negative. The Bellman-Ford algorithm is able to detect negative cycles. Finding the shortest distances in a graph with negative cycles does not make sense, because a shorter distance can always be found by checking all edges one more time.

The algorithm works as follows:

1. Set the distance to every node to be infinity
2. Set the distance to the starting node to be 0
3. Perform $V-1$ times the edge relaxation
4. Run another $ V-1$ times the edge relaxation, if an edge can be still relaxed means that it is part of a negative cycle.

#### Pharo implementation

The Pharo implementation is:

```
AIBellmanFord >> run

    start pathDistance: 0.
    self relaxEdges.
    "Run the algorithm one more time to detect if there is any negative cycles.
    The variation is if we can relax one more time an edge,
    means that the edge is part of a negative cycle.
    So, we put negative infinity as the path distance"
    self relaxEdgesToNegativeInfinity
```

```
AIBellmanFord >> relaxEdges

    | anEdgeHasBeenRelaxed |
    "Relax the edges V-1 times at worst case"
    nodes size - 1 timesRepeat: [
        anEdgeHasBeenRelaxed := false.

        edges do: [ :edge |
            edge from pathDistance + edge weight < edge to pathDistance ifTrue: [
                edge to pathDistance: edge from pathDistance + edge weight.
                edge to previousNode: edge from.
                anEdgeHasBeenRelaxed := true ] ].

        "If no edge has been relaxed means that we can stop the iteration before V-1 times"
        anEdgeHasBeenRelaxed ifFalse: [ ^ self ] ]
```

```
AIBellmanFord >> relaxEdgesToNegativeInfinity
    "This method is called after a first relaxation has occurred already.
    The algorithm is the same as the previous one but with the only difference that now if an edge can be relaxed we set the path distance
    as negative infinity because means that the edge is part of a negative cycle."

    | anEdgeHasBeenRelaxed |
    "Relax the edges V-1 times at worst case"
    nodes size - 1 timesRepeat: [
        anEdgeHasBeenRelaxed := false.

        edges do: [ :edge |
            edge from pathDistance + edge weight < edge to pathDistance ifTrue: [
                edge to pathDistance: Float negativeInfinity.
                anEdgeHasBeenRelaxed := true ] ].

        "If no edge has been relaxed means that we can stop the iteration before V-1 times"
        anEdgeHasBeenRelaxed ifFalse: [ ^ self ] ]
```

### Floyd-Warshall algorithm

Unlike the rest of the treated algorithms, Floyd-Warshall finds the shortest path solution for *every* node pair, with total cost $O(V^3)$. Because the complexity does not depend on the number of edges, it is especially suitable for dense graphs. Like the Bellmann-Ford algorithm, the Floyd-Warshall algorithm can detect negative cycles.

Conceptually, the algorithm consists of two algorithmic parts: Floyd's part calculates the shortest distances between nodes and Warshall's part constructs the shortest paths.

**Floyd's part** fills an *adjacency (distance) matrix* that represents the distances between nodes. Initially, this matrix is filled using only the direct edges between nodes. Then, the algorithm gradually updates these distances by checking if shorter paths from $i$ to $j$ exist through intermediate nodes $k$: $d[i, j] = min (d[i, j], d[i, k] + d[k, j])$.

The distance matrix only provides the shortest distances between the nodes, but not the actual path. This is the responsibility of the **Warshall's part**. For this, we need a second adjacency matrix. In this *path matrix*, the value $k$ in cell $[i,j]$ means that the node $k$ is an intermediate node in the shortest path from node $i$ to node $j$. If a shorter path is found in the course of Floyd's part, the path matrix (named `next`) is updated. 

Both algorithmic parts are merged in the following **triply nested iteration** to compute all pairs of shortest paths:

```
for every i from 1 to number of nodes:
    for every j from 1 to number of nodes:
       for every k from 1 to number of nodes:
           if distance[i][j] > (distance[i][k] + distance[k][j]) then
                distance[i][j] = distance[i][k] + distance[k][j] 
                next[i][j] = next[k][j]
```

**Negative cycles** lead to wrong results but the Floyd-Warshall is able to detect them. The preferred way to do this is by checking for negative cycles in the most inner loop, avoiding underflow integer problems. The nonnegative cycles can also be detected after the triply iteration, as our implementation does: if a node has a negative distance to itself at the end of the triply iteration, then a negative cycle exists. In such a case, both matrices must be corrected. 

Finally, the shortest path for any determined node pair we can be retrieved by the following method:

```
AIFloydWarshall >> pathFrom: sourceModel to: destinationModel

pathFrom: sourceModel to: destinationModel

    | source destination path current |
    source := self findNode: sourceModel.
    destination := self findNode: destinationModel.

    (distanceMatrix at: {
             source.
             destination }) = Float infinity ifTrue: [ ^ #(  ) ].

    (distanceMatrix at: {
             source.
             destination }) = Float negativeInfinity ifTrue: [
        ^ #( 'Path affected by negative cycle' ) ].

    source = destination ifTrue: [ ^ { source model } ].

    path := OrderedCollection new.
    path add: source model.

    current := source.
    [ current ~= destination ] whileTrue: [
        current := nextMatrix at: {
                           current.
                           destination }.
        current ifNil: [
            ^ #( 'Path broken - possible negative cycle effect' ) ].
        path add: current model ].

    ^ path asArray
```

We see that the method is able to recognize disturbances derived from any negative cycle.

### Longest path problem

To calculate the longest path of a directed graph, we can simply multiply all the edge weights by $-1$ and then calculate the shortest path. 

- If the directed graph is acyclic (DAG), then we can use the `AILongestPathInDAG` algorithm based on topological sort. 

- In case of a Directed Cyclic Graph (DCG), we can use `AILongestPathInDCG`, that is based on the Bellman-Ford algorithm.

#### Case study

![A negative weighted graph used for experimenting with Bellman-Ford algorithm.](figures/longest_path.pdf width=50&label=longestpath)

We multiply by $-1$ the weight of the edges of the previous graph in Figure *@weight@* used as an example for the DAG algorithm to obtain Figure *@longestpath@*, and then we calculate the shortest path between two nodes using the Bellman-Ford algorithm. Doing that, actually we are calculating the longest path for the original graph.

```
bellmanFord := AIBellmanFord new.
nodes := $A to: $F.
edges := #( #($A $B -1) #($B $C -5) #($B $E -11)
            #($B $D -8) #($E $F -7) #($D $E -6)
            #($G $D -4) ).
bellmanFord nodes: nodes.
bellmanFord
    edges: edges
    from: #first
    to: #second
    weight: #third.

pathFromAtoF := bellmanFord runFrom: $A to: $F.
pathDistanceFromAToF := (bellmanFord findNode: $F) pathDistance
```

If we look at the path between *A* and *F*, we see ` #( $A $B $D $E $F )` which is actually the longest path of the original graph.

### Conclusion

In this chapter, we saw some of the most-know algorithms for calculating the shortest \(or the longest\) distance on graphs. It is important to know the advantages and limitations of each algorithm in order to make the right choices for the given task.

The following table compares the weight and cycle limitations as well as the time complexity of each shortest path algorithm. 

| Class name            | Edge weights | Nonnegative cycles | Time complexity |
| --------------------- | ------------ | ------------------ | --------------- |
| `AIBFS`               | none         | yes                | $O(V + E)$      |
| `AIDFS`               | none         | yes                | $O(V + E)$      |
| `AIDijkstra`          | nonnegative  | yes                | $O(V^2)$        |
| `AIStar`              | nonnegative  | yes                | $O(E * logV)$   |
| `AIShortestPathInDAG` | yes          | no                 | $O(V + E)$      |
| `AIBellmanFord`       | yes          | yes                | $O(V * E)$      |
| `AIFloydWarshall`     | yes          | yes                | $O(V^3)$        |