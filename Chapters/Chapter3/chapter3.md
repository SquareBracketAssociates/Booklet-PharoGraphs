## Graph Representation

@cha:repre

So far the graphs were represented as drawings. But, to program them we need a data structure.
The most used data structures to represent graphs are:

- An adjacency matrix where connection between nodes is basically a cross within a matrix whose entries are nodes. The cross can contain information when we want to manipulate weighted graphs.
- An adjacency list where connection between nodes are given as a list of pairs of nodes or triples in case of weighted edges.

Note that you can use an adjancency list to _define_ a graph and use internally a matrix to perform the computation.
Basically the internal representation should be seen more as a design choice and it should not impact
the way we express algorithms. Providing a good API to manipulate graph will make the algorithm independent from the internal representation
and let the developer implement optimizations when needed.

### Graph description

In this library we use the adjacency list data structure to specify a graph.
For example, the following graph is created using the messages `nodes:` and `edges:from:to:`.
It also defines that the edges are coming from the first element to the second one.

```
| graph |
graph := AIGraphAlgorithm new.
graph nodes: #( $A $B $C ).
graph edges: #(
                #( $A $B )
                #( $A $C )
                #( $B $C ) )
  from: [:each | each first]
  to: [:each | each second].
graph run.
```

The previous snippet is a template in the sense that:

- First, we instantiate the graph algorithm \(in this case an abstract one\).
- Then, we instantiate the nodes. And finally, we set the edges. Note that for the edges we need to pass a list. The elements inside a list can be any kind of object. In the above example the objects are also a list.
- And then, we need to specific a block that is needed to obtain the `from:` and `to:` relationships.  In the example, the _in node_ is the first element of the list and the second one is the _out node_. So, we need only to send the messages `first` and `second`.

We will often define our graphs this way.

#### Blocks and symbols

Since we are a bit lazy to type, we pass directly the method names as symbol that we want to apply on the edge to extract information.

```
| graph |
graph := AIGraphAlgorithm new.
graph nodes: #( $A $B $C ).
graph edges: #(
                #( $A $B )
                #( $A $C )
                #( $B $C ) )
  from: #first
  to: #second.
graph run.
```

#### Weighted graphs

To represent weighted graphs we use the `edges:from:to:weight:` method.

```
| graph |
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
```

So in this case the library is going to take the third element as the weight for each of the edges.

### Basic graph elements

As shown by Figure *@architecture@*, a graph is basically a collection of edges and a collection of nodes.
The nodes are entities that can contain some specific value from the domain but also for the algorithm execution.
This is why we get a rich hierarchy of nodes as shown below.

![Basic graph object-oriented representation: two collections of elements.](figures/architecture.pdf width=55&label=architecture)

### About nodes

The graph algorithms of this library use different nodes. All the nodes inherit from the same abstract class called `AIGraphNode`
and show below where indentation represents inheritance \(Figure *@NodeHierarchy@*\).

% ClassHierarchyPrinter new
%         forClass: AIGraphNode;
%         doNotShowState;
%         doNotShowSuperclasses;
%         print

```
AIGraphNode
  AIBFSNode
    AIDisjointSetNode
    AINodeWithPrevious
        AiHitsNode
            AIWeightedHitsNode
  AIPathDistanceNode
    AIReducedGraphNode
    AITarjanNode
```

![Node hierarchy.](figures/hierarchy.pdf width=55&label=NodeHierarchy)

We have different subclasses because a specific algorithm may need some store some special state.
But all the nodes share a common API.

The most important methods of the API are:

- `node from:`
- `node from:edge:`
- `node adjacentNodes`
- `node model`
- `node model:`
- `node to:`
- `node to:edge:`

### Graph algorithm inheritance tree

All of the algorithms are subclasses of `AIGraphAlgorithm` as shown below where indentation represents inheritance and shown in Figure *@AlgoHierarchy@*.

% ClassHierarchyPrinter new
%         forClass: AIGraphAlgorithm;
%         doNotShowState;
%         doNotShowSuperclasses;
%         print

```
AIGraphAlgorithm
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
```

![Algorithm hierarchy.](figures/hierarchyAlgo.pdf width=55&label=AlgoHierarchy)

As the nodes, all the graph algorithms of this library share a common API also.
The class `AIGraphAlgorithm` provides the common API to add nodes, edges, searching the nodes, etc.

Some of the methods of the API are:

- `algorithm nodes:`
- `algorithm nodes`
- `algorithm edges`
- `algorithm edges:from:to:`
- `algorithm edges:from:to:weight:`
- `algorithm findNode:`
- `algorithm run`

% ClassHierarchyPrinter new
%         forClass: AIGraphAlgorithm;
%         doNotShowState;
%         doNotShowSuperclasses;
%         print

### Visualizing a graph

There is a feature to visualize a graph and by the way inspect each node or edge model. It is based on *Roassal*, Pharo's visualization engine.

For example, to visualize the graph fixture structure answered by `AINonWeightedDAGFixture new moduleGraph2`, we inspect the answer to see Figure *@moduleGraph2@*.

![Visualized graph.](figures/modulegraph2.pdf width=55&label=moduleGraph2)

The method `moduleGraph2` answers an instance of `AIGraphNonWeightedFixtureStructure` as we see here:

```
AINonWeightedDAGFixture >> moduleGraph2
| nodes edges graph |
nodes := #( $u $w $v $z $a $b $c $d ).
edges := #( #( $u $w ) #( $w $a ) #( $w $b ) #( $w $c ) #( $w $d )
            #( $w $v ) #( $v $b ) #( $v $d ) #( $v $z ) #( $z $b )
            #( $a $d ) #( $c $v ) #( $c $z ) #( $d $b ) #( $d $z ) ).
graph:= AIGraphNonWeightedFixtureStructure new.

graph nodes: nodes.
graph edges: edges.
^graph
```

`AIGraphNonWeightedFixtureStructure` and `AIGraphWeightedFixtureStructure` are both subclasses of `AIGraphTestFixtureStructure`. 

Besides viewing the graph, we can interact with it. Thanks to Roassal's canvas controller, we can among other things:

- inspect the model of a selected node or edge

- rearrange the position of a node

- zoom in and out.

By the *Help* option at the top right, we can popup the shortcuts list. 

Of course, we can also use a subclass of `AIGraphTestFixtureStructure` to visualize the graph contained in a `AIGraphAlgorithm`. For example, doing the following code shows the inspector visualizing the graph defined in the Kruskal's algorithm instance:

```
| nodes edges structure kruskal |
    nodes := #( $s $a $b $t ).
    edges := #( #( $s $a 10 ) #( $s $b 8 ) #( $a $b 5 ) #( $a $t 10 ) #( $b $t 10 ) ).

kruskal := AIKruskal new.
kruskal nodes: nodes.
kruskal
    edges: edges
    from: [ :each | each first ]
    to: [ :each | each second ]
    weight: [ :each | each third ].

structure:= AIGraphWeightedFixtureStructure new.
structure edges: (kruskal edges collect: [:edge | edge asTuple ]).
structure nodes: (kruskal nodes collect: [:node | node model ]).
structure inspect
```

![Visualized weighted graph with edge labels.](figures/edgelabels.png width=80&label=edgelabels)

In case of visualizing a weighted graph, each edge weight is rendered in the inner middle of the edge arrow like in Figure *@edgelabels@*. You might want to move the edge labels for better readability, e.g. if you need the graph for a presentation. 
In such a case, you can export it to SVG like in this example:  

```
RSSVGCairoExporter new
    canvas: AIWeightedDAGFixture new weightedDAG buildGraphCanvas;
    exportToFile: 'c:/temp/foo.svg' asFileReference
```

Then you can keep working on it with your favorite SVG editor.

`RSSVGCairoExporter` and other Roassal exporters can be loaded by executing this code snippet:

```
Metacello new
    baseline: 'RoassalExporters';
    repository: 'github://pharo-graphics/RoassalExporters';
    load.
```

Furthermore, note that we can also use `AIGraphNonWeightedFixtureStructure`  for a weighted graph in case we do not want the edge weights to be drawn. 

### Conclusion

After this basic introduction we are ready to present and implement the algorithms.