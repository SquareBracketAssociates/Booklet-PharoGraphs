## Graph Representation
graph := AIGraphAlgorithm new.
graph nodes: #( $A $B $C ).
graph edges: #(
				#( $A $B )
				#( $A $C )
				#( $B $C ) )
  from: [:each | each first]
  to: [:each | each second].
graph run.
graph := AIGraphAlgorithm new.
graph nodes: #( $A $B $C ).
graph edges: #(
				#( $A $B )
				#( $A $C )
				#( $B $C ) )
  from: #first
  to: #second.
graph run.
graph := AIGraphAlgorithm new.
graph nodes: #( $A $B $C ).
graph edges: #(
	#( $A $B 2 )
	#( $A $C 4 )
	#( $B $C 7 ) )
  from: [:each | each first]
  to: [:each | each second]
  weight: [:each | each third].
graph run.
  AIBFSNode
	AIDisjointSetNode
	AINodeWithPrevious
		AiHitsNode
			AIWeightedHitsNode
  AIPathDistanceNode
	AIReducedGraphNode
	AITarjanNode
	AIBFS
	AIBellmanFord
	AIDijkstra
	AIGraphReducer
	AIHits
		AIWeightedHits
	AIKruskal
	AIShortestPathInDAG
	AITarjan
	AITopologicalSorting