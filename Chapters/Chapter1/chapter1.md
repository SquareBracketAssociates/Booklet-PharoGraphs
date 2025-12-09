## Introduction

Graphs are everywhere and there is well-known algorithms to get the best out of them.
In this booklet the most common graph algorithms will be explained along with some case studies where these algorithms can be applied. These algorithms are available in the _Pharo AI_ _graph-algorithms_ library available at: [https://github.com/pharo-ai/graph-algorithms](https://github.com/pharo-ai/graph-algorithms).

This booklet will describe and implement thess algorithms.

### Paris metro as graph example

A typical example is a metro map as the one of Paris as shown in Figure *@paris@*.
What we see is that metro lines are connected by some stations for example \(Gare du Nord connects line 5 and 4\) and some (interchange) stations are a hub where multiple lines meet such as Chatelet.

![Newly designed Paris metro maps.](figures/Paris_Metro_map_1024.pdf width=100&label=paris)

When you visit Paris, you are often asking yourself:

- what are the different possibilities to go to that place?
- are they straight connections?
- finding the fastest one?
- finding the one with the least stops \(because RER is a fast kind of metro but stopping only in certain places\)
- finding the one with the least changes?
- what are all the places that I can reach in 5 stops?

All such questions can be answered by modeling the metro of Paris as a graph and applying algorithms to it.

Several graphs can be represented:

- we can have graph with only the interchange stations and eliminate the stations in between \(this would not be really useful for users because you do not want to only use interchange stations\).
- we can have a graph with the ride time between two stations.

### About tests

To ensure that the algorithms are working properly, we have implemented several tests for this library. We have a graph fixture in which we have implemented different types of graphs.
Each graph is implemented on a class side method and the method has a link to a picture to see the graph visually.
Then, in each of the tests for each of the algorithms, we construct a graph and check if the result after running the algorithm is the expected one.

#### Outline of the document

In this little book we will describe some algorithms that can be applied to graphs.
We will start with some basic definitions, then in subsequent chapters we will describe algorithms for several graph problem categories like topological sorting, shortest path, minimum spanning tree, strongly connected components, link analysis, graph matching or maximum flow. The algorithms are often named for its inventor or inventors (Kruskal, Tarjan, Bellman-Ford, ...) or use acronyms (BFS, DAG, HITS, ...).

All the algorithms of the library have the same API to set the nodes and the edges.
The edges can be both weighted or unweighted. A detailed explanation of how to use the API will be given on Chapter *@cha:repre@*.

### How to install

You can install the library executing the following code snippet{!footnote|EpMonitor serves to deactive Epicea, a Pharo code recovering mechanism, during the installation.!}:

```
EpMonitor disableDuring: [
    Metacello new
        repository: 'github://pharo-ai/graph-algorithms';
        baseline: 'AIGraphAlgorithms';
        load ]
```

However, the library is provided by a fresh Pharo image since Pharo 11.

You may also separately download from the `GraphGenerators` baseline group a tenner of library algorithms for generating regular and random graphs.