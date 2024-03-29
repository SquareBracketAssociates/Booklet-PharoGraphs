## Graph Representation@cha:repreSo far the graphs were represented as drawings. But, to program them we need a data structure.The most used data structures to represent graphs are:- An adjacency matrix where connection between nodes is basically a cross within a matrix whose entries are nodes. The cross can contain information when we want to manipulate weighted graphs.- An adjacency list where connection between nodes are given as a list of pairs of nodes or triples in case of weighted edges.Note that you can use an adjancency list to _define_ a graph and use internally a matrix to perform the computation.Basically the internal representation should be seen more as a design choice and it should not impactthe way we express algorithms. Providing a good API to manipulate graph will make the algorithm independent from the internal representationand let the developer implement optimizations when needed.### Graph descriptionIn this library we use the adjacency list data structure to specify a graph.For example, the following graph is created using the messages `nodes:` and `edges:from:to:`.It also defines that the edges are coming from the first element to the second one.```| graph |
graph := AIGraphAlgorithm new.
graph nodes: #( $A $B $C ).
graph edges: #(
				#( $A $B )
				#( $A $C )
				#( $B $C ) )
  from: [:each | each first]
  to: [:each | each second].
graph run.```The previous snippet is a template in the sense that:- First, we instantiate the graph algorithm \(in this case an abstract one\).- Then, we instantiate the nodes. And finally, we set the edges. Note that for the edges we need to pass a list. The elements inside a list can be any kind of object. In the above example the objects are also a list.- And then, we need to specific a block that is needed to obtain the `from:` and `to:` relationships.  In the example, the _in node_ is the first element of the list and the second one is the _out node_. So, we need only to send the messages `first` and `second`.We will often define our graphs this way.#### Blocks and symbols.Since we are a bit lazy to type, we pass directly the method names as symbol that we want to apply on the edge to extract information.```| graph |
graph := AIGraphAlgorithm new.
graph nodes: #( $A $B $C ).
graph edges: #(
				#( $A $B )
				#( $A $C )
				#( $B $C ) )
  from: #first
  to: #second.
graph run.```#### Weighted graphsTo represent weighted graphs we use the `edges:from:to:weight:` method.```| graph |
graph := AIGraphAlgorithm new.
graph nodes: #( $A $B $C ).
graph edges: #(
	#( $A $B 2 )
	#( $A $C 4 )
	#( $B $C 7 ) )
  from: [:each | each first]
  to: [:each | each second]
  weight: [:each | each third].
graph run.```So in this case the library is going to take the third element as the weight for each of the edges.### Basic graph elementsAs shown by Fig. *@architecture@*, a graph is basically a collection of edges and a collection of nodes.The nodes are entities that can contain some specific value from the domain but also for the algorithm execution.This is why we get a rich hierarchy of nodes as shown below.![Basic graph object-oriented representation: two collections of elements.](figures/architecture.pdf width=55&label=architecture)### About nodesThe graph algorithms of this library use different nodes. All the nodes inherit from the same abstract class called `AIGraphNode`and show below where indentation represents inheritance \(Fig. *@NodeHierarchy@*\).% ClassHierarchyPrinter new% 		forClass: AIGraphNode;% 		doNotShowState;% 		doNotShowSuperclasses;% 		print```AIGraphNode
  AIBFSNode
	AIDisjointSetNode
	AINodeWithPrevious
		AiHitsNode
			AIWeightedHitsNode
  AIPathDistanceNode
	AIReducedGraphNode
	AITarjanNode```![Node hierarchy.](figures/hierarchy.pdf width=55&label=NodeHierarchy)We have different subclasses because a specific algorithm may need some store some special state.But all the nodes share a common API.The most important methods of the API are:- `node from:`- `node from:edge:`- `node adjacentNodes`- `node model`- `node model:`- `node to:`- `node to:edge:`### Graph algorithm inheritance treeAll of the algorithms are subclasses of `AIGraphAlgorithm` as shown below where indentation represents inheritance and shown in Fig. *@AlgoHierarchy@*.% ClassHierarchyPrinter new% 		forClass: AIGraphAlgorithm;% 		doNotShowState;% 		doNotShowSuperclasses;% 		print```AIGraphAlgorithm
	AIBFS
	AIBellmanFord
	AIDijkstra
	AIGraphReducer
	AIHits
		AIWeightedHits
	AIKruskal
	AIShortestPathInDAG
	AITarjan
	AITopologicalSorting```![Algorithm hierarchy.](figures/hierarchyAlgo.pdf width=55&label=AlgoHierarchy)As the nodes, all the graph algorithms of this library share a common API also.The class `AIGraphAlgorithm` provides the common API to add nodes, edges, searching the nodes, etc.Some of the methods of the API are:- `algorithm nodes:`- `algorithm nodes`- `algorithm edges`- `algorithm edges:from:to:`- `algorithm edges:from:to:weight:`- `algorithm findNode:`- `algorithm run`% ClassHierarchyPrinter new% 		forClass: AIGraphAlgorithm;% 		doNotShowState;% 		doNotShowSuperclasses;% 		print### ConclusionAfter this basic introduction we are ready to present and implement the algorithms.