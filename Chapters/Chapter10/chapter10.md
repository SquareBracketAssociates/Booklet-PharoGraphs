## Maximum Flow

The Maximum Flow problem is about finding the maximum flow through a directed graph, from one node in the graph to another.

The problem can be used to model a wide variety of real-world situations, such as transportation systems, communication networks, and resource allocation.

### Flow network

A directed graph is called a flow network when each edge has a nonnegative **capacity** and each edge receives a flow. Furthermore, two nodes are distinguished, one as the **source** *s* where the flow comes from and the other as the **sink** *t* where the flow ends up.

![A maximal flow example. (source: Wikimedia)](figures/Max_flow.pdf width=50&label=maxFlow)

The amount of incoming flow on an edge cannot exceed the capacity of the edge. For all vertices except *s* and *t*, there is a **conservation** of flow, what means that the same amount of flow that goes into a vertex must also come out of it.

In Figure *@maxFlow@*, we see a flow network with maximal flow $5$. The edges are labelled with *flow/capacity*.

The **residual** capacity is defined as $capacity - flow$. Flow, capacity and residual capacity are accessed in our class `AINetworkFlowEdge`. 

### A naive greedy approach

A naive greedy approach to find the maximum flow consists of the following steps:

```
1. Start with zero flow on all edges.
2. Find an augmenting path where more flow can be sent.
3. Do a bottleneck calculation to find out how much flow can be maximally sent through that augmented path.
4. Increase the flow found from the bottleneck calculation for each edge in the augmenting path.
5. Repeat steps 2-4 until max flow is found. This happens when a new augmenting path can no longer be found.
```

An **augmenting path** is a path from source to sink where some edge has not reached its full capacity, i.e. $flow < capacity$ for at least one edge in the path. It takes $O(E)$ time to find an augmenting path using BFS (breadth-first search) or DFS (depth-first search).

The **bottleneck** is the minimum residual capacity in the augmenting path. Note that after step 4, the path is not anymore among the remaining augmenting paths to be found because the flow of every bottleneck edge in the path is augmented to the full capacity.

![Counterexample: greedy flow (top digraph) is not maximum flow (bottom digraph)](figures/greedy_counterexample.pdf width=50&label=greedyCounterexample)

As suspected, the naive greedy approach does not always yield the maximum flow. We see in Figure *@greedyCounterexample@* a counterexample. Suppose that the greedy algorithm initially finds the augmenting path $s\rightarrow v\rightarrow w\rightarrow t$. After that, it would not find any more augmenting path and keep the non-optimal flow value $3$ (top graph in Figure *@greedyCounterexample@*), missing the correct maximum flow value $5$ (bottom graph in Figure *@greedyCounterexample@*). 

### Ford-Fulkerson method

Fortunately, there is an enhancement of the naive greedy approach that leads to the optimal maximum flow: the Ford-Fulkerson method. The method applies the greedy approach explained before not on the original graph but on the so-called residual graph.

The **residual graph** is formed by adding for each edge in the original directed graph a corresponding **reverse edge** relating the edge nodes in reverse direction to complete a cycle between the two nodes. For each reverse edge, its residual capacity is always the flow of the corresponding original edge. Thanks to the residual graph, that has the double of edges than the original, the greedy algorithm doesn't stuck too early and can go on finding more augmenting paths.

### Edmonds-Karp algorithm

The Ford-Fulkerson procedure is often called method instead of algorithm because it doesn't specify how to find an augmenting path. The Edmonds-Karp algorithm implements the Ford-Fulkerson method explicitly using BFS to find the augmenting paths. BFS is better than DFS in this context and contributes to the time complexity of $O(V*E^2)$ for the Edmonds-Karp algorithm. 

In `AIEdmondsKarp`, the `run` method formulates the greedy steps like this:

```
AIEdmondsKarp >> run
  "Execute the Edmonds-Karp algorithm and return the maximum flow value"

  | maxFlow pathFlow |
  maxFlow := 0.

  [ self findAugmentingPathBFS ] whileTrue: [
    pathFlow := self findBottleneckCapacity.
    self updateFlowAlongPath: pathFlow.
    maxFlow := maxFlow + pathFlow ].

  ^ maxFlow
```

In the following method `updateFlowAlongPath:`, we see how it updates the current augmenting path by

- increasing for each of its edges the flow by the `flowValue` found in `findBottleneckCapacity` and

- adjusting the corresponding reverse edge flow to maintain the dependence between original (forward) edge and reverse edge.

```
AIEdmondsKarp >> updateFlowAlongPath: flowValue
    "Update the flow along the augmenting path found by BFS"

    | current |
    current := sink.

    [ current ~= source ] whileTrue: [
        | edge reverseEdge |
        edge := parent at: current.

        "Update forward edge flow"
        edge flow: edge flow + flowValue.

        "Update reverse edge flow"
        reverseEdge := self findReverseEdge: edge.
        reverseEdge flow: reverseEdge flow - flowValue.

        current := edge from ]
```

### Dinic's algorithm

Dinic's (Dinitz's) algorithm is a refinement of the Ford-Fulkerson method, designed to improve its efficiency by using a combination of BFS and DFS to speed up the process of finding augmenting paths.

Dinic's algorithm also operates on a residual graph, that is generated, in the same way as for Edmonds-Karp algorithm, at the time of entering the original graph edges:

```
AIDinic >> edges: aCollection from: source to: target capacity: capacityFunction

    | edge edgeRev |
    aCollection do: [ :eModel |
            edge := self addEdge: eModel from: source to: target.
            edge ifNotNil: [ edge capacity: (capacityFunction value: eModel) ].
            edgeRev := self addEdge: eModel from: target to: source.
            edgeRev ifNotNil: [ edgeRev capacity: 0 ] ]
```

#### Main steps

The main steps of Dinic's algorithm involve constructing a level graph, performing blocking flow augmentations, and repeating these phases until no more augmenting paths can be found:

```
1. Level graph construction (uses BFS)
2. Blocking flow augmentation (uses DFS)
3. Repeat: The process is repeated until no more augmenting paths can be found in the level graph.
```

The graph nodes class for Dinic's algorithm is `AIDinicNode`, where the values `level` (useful for level graph construction and for finding a blocking flow) and `currentIndex` (useful for finding a blocking flow) can be accessed.

Now, we outline to some extent both main steps: the level graph construction and the blocking flow augmentation.

#### Level graph construction

While the Ford-Fulkerson method uses a sequence of augmenting paths to update the flow in a network, Dinic's algorithm works in phases, using level graphs to find augmenting paths in a more structured manner, significantly reducing the number of iterations.

In Edmond's Karp algorithm, we use BFS to find an augmenting path and send flow across this path, whereas in Dinic's algorithm, we use BFS to check if more flow is possible and to construct a level graph.

The **level graph** is a subgraph of the original flow network, where edges only exist from vertices of a higher level to those of a lower level. This is used to speed up the process of finding augmenting paths. The level graph is basically mantained at each iteration by the following method `bfs`.

```
AIDinic >> bfs
    "This method uses bfs on the residual graph to construct a level graph.
    The level graph assigns levels or distances to each node, indicating the shortest path from the source.
    The returnBool boolean indicates if there exists and augmenting path (path from source to sink) in the residual level graph."

    | node ind returnBool |
    [ queue isNotEmpty ] whileTrue: [
            node := queue removeFirst.
            ind := adjList at: node.
            ind do: [ :i |
                    | e n |
                    e := edges at: i.
                    n := e to.
                    e residualCapacity >= 1 & (n level = -1) ifTrue: [
                            n level: node level + 1.
                            queue addLast: n ] ] ].
    returnBool := sink level == -1.
    ^ returnBool
```

#### Blocking flow augmentation

After having reconstructed the level graph, the next step uses DFS 

- to find a new **blocking flow**, a flow configuration where no more flow can be pushed from the source to the sink in the current level graph without violating capacity constraints and 

- to push the found maximum possible flow through the network, augmenting the flow along the paths in the level graph.

#### Complexity

Dinic's algorithm's ($O(V^2*E)$) complexity is a theoretical upper bound. In practice, it often performs much better, sometimes approaching O($V*E* log V$) or even better for certain graph classes or with the use of advanced data structures like dynamic trees.

### Conclusion

For most problems, greedy algorithms do not generally produce the best-possible solution. But it is still worth trying them, because the ways in which greedy algorithms break often yields insights that lead to better algorithms.

On the one hand, Edmond-Karps algorithm is easier to understand and implement than Dinic's algorithm. On the other hand, Dinic's algorithm ($O(V^2*E)$) is generally faster than Edmonds-Karp algorithm ($O(V*E^2)$), especially on larger dense graphs. 
