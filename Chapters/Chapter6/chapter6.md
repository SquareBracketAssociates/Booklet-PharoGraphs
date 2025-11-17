## Minimum Spanning Tree

The minimum spanning tree is a subset of the edges of an undirected weighted graph that connects all the nodes of the graph without any cycles and with the total sum of the weights minimized. There are several algorithms for obtaining the minimum spanning tree of a weighted graph, the most famous is the Kruskal's algorithm. In the case of a disconnected graph, the algorithm returns a minimum spanning forest \(i.e., it will return a list a minimum spanning trees\). The other most commonly used algorithm is Prim's algorithm.

We will first show you how to implement Kruskal's algorithm but before, we introduce a little data-structure called _disjoint-set_ that decisively contributes to Kruskal's algorithm efficiency.

### Motivating scenario

Imagine that you have a telecommunication company and you want to build a connection between different neighbourhoods. Some of the connections are more expensive than others. For example, one connection has to pass under the ground or above some mountains. So, you have a graph in which the nodes represent the different neighbourhoods and the edges represent all the possible cables that can be built to make the connections between the neighbourhoods. The weights represent the cost of actually building the connection.

Imagine that we have the graph shown in Figure *@inputgraph@* and we would like to get the tree that allows us to get all the nodes of the graph without cycle as shown in Figure *@outputtree@*.

![Connections costs between neighbourhoods.](figures/kruskal.pdf width=35&label=inputgraph)

![Minimum spanning tree with node C as root.](figures/minimum_spanning_tree.pdf width=35&label=outputtree)

### Disjoint-set data structure

A disjoint-set, also called union-find data structure, is a data structure that stores disjoint sets. It provides two methods:

- _union_ groups two disjoint sets into one and
- _find_ answers whether two elements belong to the same set or not.

For example, in Figure *@union_find_set@* we have two sets of elements $\{A, B, C\}$ and $\{D, E\}$. If we call the method *find* with A and D nodes, as they do not belong to the same set, the method will return false. With A and B nodes, the method *find* will return true.
![Two Union-Find sets.](figures/union_find.pdf width=35&label=union_find_set)

Further, when we invoke the method *union* with A and D, it will join the two sets to have only one set with all the elements, as in Figure *@only_one_set@*.

![Union-Find set: result of the union operation between A and D. ](figures/only_one_set.pdf width=35&label=only_one_set)

A particular implementation of this data structure is the so-called *disjoint-set forest*, which executes in the `find` method an operation called **path compression**. Thanks to this operation, a loop with few code, the time complexity of the methods `find` and `union` is nearly constant. Namely, It has been proven that with path compression, the *amortized* time complexity of both methods is $O(\alpha(n))$, where $\alpha$ is the Ackermann function's inverse, which grows very slowly remaining less than 5 for any practical input size.

In our Pharo library, the disjoint-set forest (with path compression) is supported by the node class `AIDisjointSetNode`.

```
AIDisjointSetNode >> union: aDSNode

    | root1 root2 |
    root1 := aDSNode find.
    root2 := self find.

    root1 = root2 ifTrue: [
        "The nodes already belong to the same component"
        ^ self ].

    root1 parent: root2
```

```
AIDisjointSetNode >> find
    "Return the root of the component but modifying the parent/child structure during the process of finding a root."

    | root next node |
    node := self.
    root := node.
    [ root = root parent ] whileFalse: [ root := root parent ].

    "Compress the path leading back to the root.
    This is the path compression operation that gives the linear amortized time complexity"
    [ node = root ] whileFalse: [
        next := node parent.
        node parent: root.
        node := next ].

    ^ root
```

### Kruskal's algorithm

As said above, the Kruskal's algorithm calculates the minimum spanning tree \(or forest\) of an undirected weighted graph. The algorithm uses the disjoint-set data structure to check if adding an edge to the spanning tree creates a cycle.

Kruskal's algorithm pseudocode is:

```
1. Sort edges in ascending weight.
2. Pick the smallest edge.
    Check if its two nodes are already unified.
      If they are not, unified them and include the edge to the spanning tree.
      Else, discard it.
3. Repeat step 2 until there are all nodes are connected.
```

This is the implementation of the algorithm in Pharo:

```
AIKruskal >> run

    | treeEdges sortedEdges |
    sortBlock := [ :e1 :e2 | e1 weight < e2 weight ].
    treeEdges := OrderedCollection new.
    sortedEdges := edges asSortedCollection: sortBlock.
    sortedEdges
        reject: [ :edge |
            "Only join the two nodes if they don't belong to the same component"
            edge from find = edge to find ]
        thenDo: [ :edge |
            edge from union: edge to.
            treeEdges add: edge ].
    ^ treeEdges
```

The algorithm has a time complexity of $O(E * log(E))$ due to the initial edge sort. The rest has linear time complexity thanks to the disjoint-set data structure.

### Kruskal's algorithm for maximum spanning tree

As opposite to the minimum spanning tree, the maximum spanning tree of a graph is a subset of edges of a graph that connects all nodes with the **maximum** possible distance.

This is exactly the same algorithm except that we have to order the edges in descending weight instead of ascending.

```
1. Sort edges in descending weight.
2. Pick the biggest edge...
```

In the implementation we only need to change one line:

```
sortBlock := [ :e1 :e2 | e1 weight > e2 weight ].
```

### Case study

We can now apply our algorithm to the graph in Figure *@spsituation@*, already shown at the beginning of this chapter in Figure *@inputgraph@*.

![Connections costs between neighbourhoods.](figures/kruskal.pdf width=55&label=spsituation)

So, like in the other graph algorithms we only need to declare the nodes and the edges an then call the method `run` to obtain the result.

```
nodes := $A to: $J.
edges := #( #( $A $B 25 ) #( $A $D 8 ) #( $A $F 11 ) #( $B $A 25 )
            #( $B $E 1 ) #( $B $C 12 ) #( $C $B 12 ) #( $C $D 16 )
            #( $C $F 6 ) #( $C $G 9 ) #( $D $A 8 ) #( $D $C 16 )
            #( $E $B 1 ) #( $E $G 14 ) #( $F $A 11 ) #( $F $C 6 )
            #( $F $G 5 ) #( $F $J 4 ) #( $G $F 5 ) #( $G $C 9 )
            #( $G $E 14 ) #( $G $H 7 ) #( $H $G 7 ) #( $I $J 7 )
            #( $J $F 4 ) #( $J $I 7 ) ).
kruskal := AIKruskal new.
kruskal nodes: nodes.
kruskal
    edges: edges
    from: #first
    to: #second
    weight: #third.
minimumSpanningTree := kruskal run
```

If we inspect the `minimumSpanningTree` variable, we get a collection the edges of the minimum spanning tree. DSN means *DisjointSetNode*.

```
DSN $B -> DSN $E weight: 1
DSN $J -> DSN $F weight: 4
DSN $F -> DSN $G weight: 5
DSN $F -> DSN $C weight: 6
DSN $I -> DSN $J weight: 7
DSN $H -> DSN $G weight: 7
DSN $A -> DSN $D weight: 8
DSN $A -> DSN $F weight: 11
DSN $C -> DSN $B weight: 12
```

![Minimum spanning tree.](figures/minimum_spanning_tree.pdf width=55)

If we want to obtain the maximum spanning tree, we only need to call the `maxSpanningTree` method when creating the graph algorithm.

```
kruskal := AIKruskal new maxSpanningTree.
```

### Prim's algorithm

Basically, Prim’s algorithm to find a minimum spanning tree is a modified version of Dijkstra’s algorithm for the shortest path problem. The algorithm starts with a single node and moves through several adjacent nodes, in order to explore all of the connected edges along the way.

Prim's algorithm pseudocode is:

```
1. Initialize a priority queue (or a min-heap) to store edges.
2. Start from an arbitrary node and add all its edges to the priority queue.
3. While the priority queue is not empty:
       Extract the edge with the minimum weight.
       If the edge connects to a node not already in the tree, add it to the tree and add its edges to the priority queue.
```

Again, it is decisive to use opportune data structures. By using a priority queue or a sophisticated heap to store the candidate edges, we can considerably improve the inner loop and achieve $O(E * log(V)) $ time complexity overall. Otherwise, we would clash with quadratic time complexity or worse.

### Conclusion

Both the Kruskal's and the Prim’s algorithm are greedy algorithms that find the minimum spanning tree for a connected, undirected graph. Both have many real life applications and are important in the context of graph theory.

The difference betweeen both algorithms lies in how the minimum spanning tree is constructed. Kruskal's algorithm sorts the edges according to their weights and adds them in ascending order. On the other hand, Prim's algorithm starts with one node. Starting from this node, a spanning tree is gradually formed until all nodes are considered. On this basis, their time complexity can attain, with the appropiate data structures, $O(E * log(E))$ respectively $O(E * log(V))$. In principle, because of this difference, Kruskal's algorithm is rather suitable for sparse graphs (with few edges) while Prim's algorithm should be preferred for dense graphs (with a lot of edges).

Data structures play a powerful role when it comes to algorithms. In this specific case, Kruskal's algorithm detects cycles in amortized linear time complexity with few lines of code thanks to the disjoint-set data structure and Prim's algorithm benefits from a well-designed priority queue or heap.  