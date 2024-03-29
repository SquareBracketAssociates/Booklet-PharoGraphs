## Shortest path problemThe shortest path problem consists on finding a path between two pairs of nodes in which the sumof the weights is minimized. For a general graph this problem is NP-hard. For some kind of graphs this problemcan be solved in linear time.In this chapter we will present multiple algorithms:- A version implementing breath first traversal,- Dijkstra's algorithm for weighted nodes,- an algorithm for Directed Acyclic Graphs and,- Bellman-Ford algorithm for negative weighted graphs.### ExamplesLet us take this graph as an example *@short_distance_graph@* As it is an unweighted graph, we can calculate the distance between 2 nodes using the BFS algorithm.![Short distance graph.](figures/generic_example.pdf width=48&label=short_distance_graph)But, if we add weights to the graph, as in *@short_distance_graph_2@* we cannot no longer use BFS. But we can find the shortest distance using the Dijkstra algorithm.![Short distance with weights.](figures/dag.pdf width=48&label=short_distance_graph_2)The Dijkstra's algorithm does not work on graphs with negative weights. So, if we add negative weights to the graph, we must use the Bellman-Fordalgorithm to solve the problem. Figure *@short_distance_graph_3@*![Short distance with negative weights.](figures/longest_path.pdf width=48&label=short_distance_graph_3)If the graph has no cycles, a Directed Acyclic Graph \(DAG\) \(like Figure *@short_distance_graph_3@*\), we can use an algorithm based on topological sort to findthe shortest distance. This algorithm works for both negative and positive weights as long the graph has no cycles. This algorithm is betterin terms of time complexity but is more restricted as it only runs on DAG. On the other hand, Dijkstra's and Bellman-Ford both run in both cyclic and acyclic graphs.### Shortest path on unweighted graphs \(BFS algorithm\)If the graph is unweighted or all edges have the same **non-negative** weight, the shortest path can be foundin linear time $O(V + E)$ using Breadth First Search \(BFS\) algorithm.BFS is an algorithm for traveling a graph in a traversal way. That means that the algorithm will travel the children of the starting node always in orderensuring that when the goal node, if exists, is founded the path will be the shortest one possible.BFS is a single source shortest path algorithm. That means that before running the algorithm it is neededto specify a starting node. Then, the algorithm can tell us the shortest path between the starting nodeand all the other nodes.The algorithm is the following one:```initialize a queue Q
mark start node as visited
Q.addLast(start)
while Q is not empty do:
  node := Q.removeFist()
  if node is the end then:
    return node
  for adjacent nodes of node do:
    if adjacentNode is not visited then:
      mark adjacentNode as visited
      Q.addLast(adjacentNode)```In the Pharo implementation we use a linked list as a queue. We use a `LinkedList` instead of an `OrderedCollection` because the `removeFirst` operation of `LinkedList`takes constant time and for an `OrderedCollection`, it takes linear time.We define a new subclass of `AIGraphAlgorithm` named `AIBFS`.And we define a new method `run` as follows:```AIBFS >> run

	| node neighbours |
	queue := LinkedList with: start.
	start visited: true.

	[ queue isNotEmpty ] whileTrue: [
		node := queue removeFirst.
		neighbours := node adjacentNodes.

		neighbours do: [ :next |
			next visited ifFalse: [
				queue addLast: next.
				next visited: true.
				next previousNode: node ] ] ]```After running the algorithm, to reconstruct the shortest path between the start and the endnode, we use the following method:```AIBFS >> reconstructPath

	| path previous |
	"If no path exists between the start and the end node"
	end previousNode ifNil: [ ^ #( ) ].

	path := LinkedList empty.
	previous := end.
	path addFirst: end model.
	[ previous = start ] whileFalse: [
		previous := previous previousNode.
		path addFirst: previous model ].
	^ path```### Case study![A graph for BFS.](figures/bfs.pdf width=40)For the BFS graph, the shortest path can be calculated using the class `AIBFS` like shown in the following example.```nodes := $a to: $i.
edges := #( #( $a $b ) #( $b $c ) #( $c $d ) #( $d $e ) #( $e $a )
            #( $b $e ) #( $e $b ) #( $e $f ) #( $f $g ) #( $g $h )
            #( $h $f ) #( $g $i ) #( $i $g ) ).
bfs := AIBFS new.
bfs
	nodes: nodes;
	edges: edges from: #first to: #second.

path := bfs runFrom: $a to: $g```The `path` variable contains all the nodes that are part of the path, if we inspect the variable we see:```#($a $b $e $f $g)```If we want to get the shortest path between the same starting node A and some other node, there is no needof re-running the algorithm. We only need to change the end node and call the method reconstruct path.```bfs end: $d.
pathToD := bfs reconstructPath```### Shortest path on weighted graphs \(Dijkstra's algorithm\)The Dijkstra's algorithm is one of the most-know algorithms for calculating the shortest path in a weighted graph.As BFS, this algorithm is also a single source shortest path algorithm. In its naive implementation has a timecomplexity of $O(V^2)$. But, it can be optimized using a heap or a Fibonacci's heap as a data structure to a timecomplexity of $O((V+E)*log V)$. If a Fibonacci heap is used we get the best possible time complexity$O(E + V * log V)$ possible \(at the moment\). Dijkstra's algorithm can handle a graph with cycles. But, it cannot handle negative weights.The algorithm idea is:1. Mark all nodes as unvisited.1. Assign to every node infinity as the distance value. Set it to zero for the initial. Set the initial node as current.1. Consider all of unvisited neighbours of the current node and calculate their distances through the current node. Compare the newly calculated tentative distance to the current assigned value and assign the smaller one.1. When all of the unvisited neighbours of the current node are checked, mark the current node as visited.1. Select the unvisited node that is marked with the smallest tentative distance, set it as the new "current node", and go back to step 3.Depending on the data structure that is chosen, the time complexity will vary.Using an array is the most inefficient one because to get the mostpromising pair it is necessary to travel all the array $O(N)$.But with a heap, or a Fibonacci heap, the time complexity for gettingthe most promising pair is in the logarithmic order.Our implementation uses an array for now.The implementation in Pharo is the following:```AIDijkstra >> run

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
						pq add: edge to -> newDistance ] ] ] ] ]``````AIDijkstra >> updateDistance: newDistance of: aNode previousNode: previousNode

	aNode previousNode: previousNode.
	aNode pathDistance: newDistance```In this implementation we will use an Oredered Collection as a data structure for the priority queue.```AIDijkstra >> newPriorityQueue
	"This is the naive implementation of the data structure."

	^ OrderedCollection new``````AIDijkstra >> removeMostPromisingPair: aPriorityQueue
	"This is the naive implementation of the data structure."

	| minValue |
	minValue := aPriorityQueue detectMin: [ :assoc | assoc value ].
	^ aPriorityQueue remove: minValue```### Case study![Dijkstra graph.](figures/dijkstra.pdf width=50&label=dij)In the graph shown in Figure *@dij@*, the shortest path between node A and B is `#( $A $C $B )`. The shortest path between node A and node F is `#( $A $C $B $D $E $F )````nodes := $A to: $F.
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
shortestPathBToE := dijkstra runFrom: $B to: $E.```### Shortest path on Directed Acyclic Graphs \(DAG\)If the graph is a directed acyclic weighted graph \(DAG\), we can calculate the shortest path using an algorithmbased on topological sort. Using this algorithm we have a time complexity of $O(V + E)$.This algorithm is also single source shortest path. The idea of the algorithm is to order the nodes in atopological order, using the topological sort algorithm. Then, keep track of the path weight from the start node to the other nodes.Then, start popping the nodes in order and store in a collection the ones that have the lowest path weight.As this algorithm runs in graphs that has no cycles, it _can_ handle negative weights.The pseudocode is:1. Initialize the initial distance to every node to be infinity and the distance of the start node to be 0.1. Create a topological order of all nodes.1. For every node u in topological order:- Do following for every adjacent node v of u- IF \(v pathWeight > u pathWeight $+$ weight\(u, v\)\) THEN v pathWeight: u pathWeight + weight\(u, v\)### DAG shortest path implementationThe Pharo implementation is as follows.```AIShortestPathInDAG >> initializePathWeights

	nodes do: [ :node | node pathWeight: Float infinity ].
	start pathWeight: 0``````AIShortestPathInDAG >> run

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
					nextEdge to previousNode: sortedNode ] ] ]```### DAG shortest path refactoredNow we are ready to refactor our code.```AIShortestPathInDAG >> run

	| stack sortedNode |
	self initializePathWeights.
	stack := self topologicalSortedNodes.

	[ stack isNotEmpty ] whileTrue: [
		sortedNode := self findNode: stack removeFirst.

		sortedNode outgoingEdges do: [ :nextEdge |
			nextEdge to pathDistance >
      (sortedNode pathDistance + nextEdge weight)
				ifTrue: [
          self updatePathDistance: nextEdge previousNode: sortedNode ] ] ]``````AIShortestPathInDAG >> topologicalSortedNodes

	| topSorter |
	topSorter := AITopologicalSorting new
		addNodesFromDifferentGraph: nodes;
    yourself.
	 topSorter run.
	 ^ topSorter topologicalSortedElements.``````AIShortestPathInDAG >> updatePathDistance: edge previousNode: previousNode

	edge to pathDistance: previousNode pathDistance + edge weight.
	edge to previousNode: previousNode```#### Case studyOn this weighted DAG \(See Figure *@weight@*\), the following snippet calculates the shortest path between node A and node F.```nodes := $A to: $G.
edges := #( #( $A $B 1 ) #( $B $C 5 ) #( $B $E 11 ) #( $B $D 8 )
            #( $D $E 6 ) #( $E $F 7 ) #( $G $D 4 ) ).
shortestPathInDAG nodes: nodes.
shortestPathInDAG
	edges: edges
	from: #first
	to: #second
	weight: #third

pathAtoF := shortestPathInDAG runFrom: $A to: $F.```![DAG with weigthed paths.](figures/dag.pdf width=50&label=weight)### Shortest path on weighted graphs with negative weights \(Bellman-Ford algorithm\)In the Dijkstra's algorithm when a node is marked as visited, the algorithm already found the best distance to it, because adding any positive numberswill only increase the path distance. When we are dealing with negative numbers the assumption is not true.The Bellman-Ford algorithm can handle negative weighted graphs. It runs in $O(V*E)$ time. As the other algorithms of this chapter, this is also a single sourceshortest path algorithm. The logic behind the algorithm is to perform at worst $V-1$ times an edge relaxation. Relaxing an edge means to update the value of thedistance from the starting node to the node to which to edge goes. Then, run the algorithm one more time, if an edge can reduce its distance \(be relaxed\) means that the nodeto which the edge goes is part of a negative cycle.The algorithm works as follows:1. Set the distance to every node to be infinity1. Set the distance to the starting node to be 01. Perform V-1 times the edge relaxation1. Run another V-1 times the edge relaxation, if an edge can be still relaxed means that is part of a negative cycle.### Pharo implementationThe Pharo implementation is:```AIBellmanFord >> run

	start pathDistance: 0.
	self relaxEdges.
	"Run the algorithm one more time to detect if there is any negative cycles.
	The variation is if we can relax one more time an edge,
	means that the edge is part of a negative cycle.
	So, we put negative infinity as the path distance"
	self relaxEdgesToNegativeInfinity``````AIBellmanFord >> relaxEdges

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
		anEdgeHasBeenRelaxed ifFalse: [ ^ self ] ]``````AIBellmanFord >> relaxEdgesToNegativeInfinity
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
		anEdgeHasBeenRelaxed ifFalse: [ ^ self ] ]```### Longest path problemTo calculate the longest path of a graph we can simply multiply all the nodes weights by $-1$ and thencalculate the shortest path. If the graph is a DAG, then we can use the topological sort based algorithm.If not, we can use Bellman-Ford.#### Case study![A negative weighted graph used for experimenting with Bellman-Ford algorithm.](figures/longest_path.pdf width=50)We multiply by $-1$ the weight of the edges of the previous graph used as an example for the DAG algorithmand then we calculate the shortest path between two nodes using the Bellman-Ford algorithm. Doing that, actuallywe are calculating the longest path for the original graph.```bellmanFord := AIBellmanFord new.
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
pathDistanceFromAToF := (bellmanFord findNode: $F) pathDistance```If we look at the path between `A` and `F`, we see ` #( $A $B $D $E $F )` which is actually the longest pathof the original graph.### ConclusionIn this chapter we saw some of the most-know algorithms for calculating the shortest \(and longest\) distance on graphs. Calculate the shortest path, or distance,in a graph is not a trivial problem and it is a NP-Hard problem. For some types of graphs, like trees or DAG \(Directed Acyclic Graphs\), we can solve the problem in linear time.