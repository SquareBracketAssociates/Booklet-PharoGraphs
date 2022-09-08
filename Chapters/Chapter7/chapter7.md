## Strongly Connected Components in a Graph
2. Start DFS. When visiting a node assign it an ID and a low-link value same as the ID.
3. Mark current node as visited and add it to the stack.
4. On DFS callback,
    First, min the current node's low-link value with the low-link value of the adjacent node.
    Then, if the adjacent node is on the stack
	            then min the current node's low-link with the adjacent node's ID.
5. After visiting all adjacent nodes, if the current node has its ID value as the same of its low-link value
		Then it means that there is a strongly connected component.
		So, pop all nodes from the stack until current node is reached.
	"Initialize an empty array for the strongly connected components"

	sccs := OrderedCollection new.
	stack := Stack new.
	runningIndex := 0.

	nodes do: [ :node |
		node isTarjanUndefined ifTrue: [
			"If the node has no low-link value set make a dfs call"
			self traverse: node ] ].
	^ self stronglyConnectedComponents

	aTarjanNode tarjanIndex: runningIndex.
	aTarjanNode tarjanLowlink: runningIndex.
	runningIndex := runningIndex + 1.

	self putOnStack: aTarjanNode.

	aTarjanNode adjacentNodes do: [ :adjacentNode |
		adjacentNode isTarjanUndefined
			ifTrue: [
				"If the adjacent node doesn't have a low link"
				self traverse: adjacentNode.
				aTarjanNode tarjanLowlink:
					(aTarjanNode tarjanLowlink
						min: adjacentNode tarjanLowlink) ]
			ifFalse: [
				"If the adjacent node had already a low link value"
				adjacentNode inStack ifTrue: [
					aTarjanNode tarjanLowlink:
						(aTarjanNode tarjanLowlink
							min: adjacentNode tarjanIndex) ] ] ].

	"If the node is the beginning of a strongly connected component"
	(self isRootNode: aTarjanNode) ifTrue: [
		self addNewSccForNode:: aTarjanNode ]

	stack push: aTarjanNode.
	aTarjanNode inStack: true

	| currentNode stronglyConnectedComponent |
	stronglyConnectedComponent := OrderedCollection empty.

	[ currentNode := stack pop.
	currentNode inStack: false.
	stronglyConnectedComponent add: currentNode ]
		doWhileFalse: [ currentNode = aTarjanNode ].

	sccs add: stronglyConnectedComponent.
	stronglyConnectedComponent do: [ :each |
		each cycleNodes: stronglyConnectedComponent ]

	sccs ifNil: [ self run ].
	^ sccs collect: [ :component |
		component collect: [ :each | each model ] ]
edges := #( #( $a $b ) #( $a $c ) #( $a $g ) #( $b $e ) #( $c $b )
            #( $c $d ) #( $d $f ) #( $f $c ) #( $g $h ) #( $g $d )
            #( $h $g ) ).
tarjan := AITarjan new.
tarjan
	nodes: nodes;
	edges: edges from: #first to: #second.
stronglyConnectedComponents := tarjan run
an OrderedCollection($b)
an OrderedCollection($c $f $d)
an OrderedCollection($h $g)
an OrderedCollection($a)
edges := #( #( $a $b ) #( $a $c ) #( $a $g ) #( $b $e ) #( $c $b )
            #( $c $d ) #( $d $f ) #( $f $c ) #( $g $h ) #( $g $d )
            #( $h $g ) ).
graphReducer := AIGraphReducer new.
graphReducer
	nodes: nodes;
	edges: edges from: #first to: #second.
reducedGraph := graphReducer run
Merged nodes: $b
Merged nodes: $e
Merged nodes: $h, $g
Merged nodes: $c, $f, $d