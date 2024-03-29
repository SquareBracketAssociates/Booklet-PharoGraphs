## Minimum spanning treesThe minimum spanning tree is a subset of the edges of a undirected weighted graph that connects all the nodes of the graphwithout any cycles and with the total sum of the weights minimized. There are several algorithms for obtaining the minimum spanningtree of a weighted graph, the most famous is the Kruskal's algorithm. In the case of an disconnected graph, the algorithmreturns a minimum spanning forest \(i.e., it will return a list a minimum spanning trees\)We will show you how to implement Kruskal but before that we have to introduce a little data-structure called _Disjoint-Set_.### Motivating scenarioImagine that you have a telecommunication company and you want to build a connection between different neighbourhoods.Some of the connections are more expensive than others. For example, one connection has to pass under the ground or above some mountains.So, you have a graph in which the nodes represent the different neighbourhoods and the edges represent all thepossible cables that can be built to make the connections between the neighbourhoods. The weights represent the cost ofactually building the connection.Imagine that we have the graph shown in Figure *@spanningsmall@*, we would like to get the tree that allows us to get all the nodes of the graph without circle as shown in Figure *@spanningsmall@*.![Connections costs between neighbourhoods.](figures/kruskal.pdf width=35&label=spanningsmall)![Minimum spanning tree with C as root.](figures/minimum_spanning_tree.pdf width=35&label=spanningsmall)### Disjoint-Set data structureA disjoint-set, also called union-find data structure, is a data structure that stores disjoint sets. It provides two operations:- _unite_ that groups two disjoint sets into one and- _find_ that returns two elements belong to the same disjoint set.For example, in Figure *@union_find_set@* we have two sets of elements $\{A, B, C\}$ and $\{D, E\}$. If we call the operation find with A and D nodes,as they do not belong to the same set, the operation will return false. With A and B nodes, the operation find will return true.![Two Union-Find sets.](figures/union_find.pdf width=35&label=union_find_set)But when we invoke the operation unite with A and D, it will join the two sets to have only one set with all the elements, as in Figure *@only_one_set@*.![Union-Find set: result of the unite operation between A and D. ](figures/only_one_set.pdf width=35&label=only_one_set)This data structure is used in Kruskal's algorithm to detect if adding a new edge creates a cycle in the minimum spanning tree that is being built.The time complexity of both of the operations is $O(a(n))$, where $a$ is the amortized time complexity.  Each time thatthe `find:` method is invoked an operation called _path compression_.This is due to the path compression operation that this data structure has an amortized linear time complexity.In Pharo, this data structure represents a node in the Kruskal's graph algorithm.```AIDisjointSetNode >> union: aDSNode

	| root1 root2 |
	root1 := aDSNode find.
	root2 := self find.

	root1 = root2 ifTrue: [
		"The nodes already belong to the same component"
		^ self ].

	root1 parent: root2``````AIDisjointSetNode >> find
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

	^ root```### Kruskal's algorithmAs said above, the Kruskal's algorithm calculates the minimum spanning tree \(or forest\) of an undirected weighted graph.The algorithm has a time complexity of $O(V*log(E)) = O(E*log(E))$. This time complexity is achieved thanks to theDisjoint-Set data structure. This algorithm uses the Disjoint-Set data structure to check if adding an edge to the spanningtree creates a cycle.The pseudocode is:```1. Sort edges in ascending weight.
2. Pick the smallest edge.
    Check if its two nodes are already unified.
      If they are not, unified them and include the edge to the spanning tree.
      Else, discard it.
3. Repeat step 2 until there are all nodes are connected.```This is the implementation of the algorithm in Pharo:```AIKruskal >> run

	| treeEdges sortedEdges |
  sortBlock := [ :e1 :e2 | e1 weight < e2 weight ].
	treeEdges := OrderedCollection new.
	nodes do: [ :node | node makeSet].
	sortedEdges := edges asSortedCollection: sortBlock.
	sortedEdges
		reject: [ :edge |
			"Only join the two nodes if they don't belong to the same component"
			edge from find = edge to find ]
		thenDo: [ :edge |
			edge from union: edge to.
			treeEdges add: edge ].
	^ treeEdges```### Kruskal's algorithm for maximum spanning treeAs contrary to the minimum spanning tree, the maximum spanning tree of a graph is a subset of edges of a graph that connects all nodes with the **maximum** possible distance.This is exactly the same algorithm except that we have to order the edges in descending weight instead of ascending.```1. Sort edges in descending weight.
2. Pick the biggest edge...```In the implementation we only need to change one line:```sortBlock := [ :e1 :e2 | e1 weight > e2 weight ].```### Case studyWe can now apply our algorithm to the graph shown at the beginning of this chapter in Figure *@spsituation@*.![Connections costs between neighbourhoods.](figures/kruskal.pdf width=55&label=spsituation)So, like in the other graph algorithms we only need to declare the nodes and the edges an then call the method `run`to obtain the result.```nodes := $A to: $J.
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
minimumSpanningTree := kruskal run```If we inspect the `minimumSpanningTree` variable, we get a collection the edges of the minimum spanning tree._DSN_ means _DisjointSetNode_.```DSN $B -> DSN $E weight: 1
DSN $J -> DSN $F weight: 4
DSN $F -> DSN $G weight: 5
DSN $F -> DSN $C weight: 6
DSN $I -> DSN $J weight: 7
DSN $H -> DSN $G weight: 7
DSN $A -> DSN $D weight: 8
DSN $A -> DSN $F weight: 11
DSN $C -> DSN $B weight: 12```![Minimum spanning tree.](figures/minimum_spanning_tree.pdf width=55)If we want to obtain the maximum spanning tree, we only need to call the `maxSpanningTree` methodwhen creating the graph algorithm.```kruskal := AIKruskal new maxSpanningTree.```### ConclusionData structures play a powerful role when it comes to algorithms. In this specific case thank to the Disjoint-Set data structure we can detect cycles in amortizedlinear time complexity with few lines of code. Also, the Kruskal algorithm has many real life applications and it is an important algorithm in the context of graph theory.