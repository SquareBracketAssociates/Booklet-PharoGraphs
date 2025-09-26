## Matchings (Independent Edge Sets)
In graph theory, a matching or independent edge set in an undirected graph is a set of edges without common vertices. 
The term matching is the more popular one, but it shouldn't be confused with another meaning of graph matching, namely computing the similarity of graphs (graph isomorphism).

Matchings are used in various applications such as network design, job assignments, and scheduling. 
More specifically, matching strategies are very useful in flow network algorithms such as the Edmonds-Karp algorithm that is also in our library.

### Definitions and examples
The **cardinality** of a matching is the number of its edges. Figure 9-1 shows 3 matchings of cardinality 1, 2 and 2 (from left to right).

Figure 9-1: <img width="338" height="70" alt="image" src="https://github.com/user-attachments/assets/b7e53772-9506-4116-832d-7c0607fff34b" />

A matching is **maximal** if it cannot be expanded to another matching by addition of any edge in the graph.
The matchings in figure 9-1 are maximal.

Moreover, a **maximum**(-cardinality) matching has the largest possible cardinality among all matchings in a graph.
For each of the matchings in figure 9-1, figure 9-2 shows a maximum matching example with cardinality 2, 3 and 2.

Figure 9-2: <img width="312" height="71" alt="image" src="https://github.com/user-attachments/assets/53485f3d-28e5-4f17-b072-790cbaba5552" />

We note from these examples that a maximum matching is always maximal, but the converse does not always hold.

A **maximum-weight** matching in a weighted graph is a matching with the largest sum of weights. 
Similarly, a **minimum-weight** matching has the smallest sum of weights.

### A greedy algorithm for matching
The algorithm for graph matching implemented in this library is a simple greedy algorithm with time complexity of $O(|E|log(|V|))$.
The class ```AIGraphMatchingAlgorithm``` can be instantiated to find an **approximation** either of the maximum-weight, the minimum-weight or the maximum-cardinality matching.

For instance, the pseudocode to greedily find a matching of large weight is the following one:
```
M ← Empty list that will contain the matching edges
E ← Set of all edges
remove all loops (edges joining a vertex to itself) from E

while E is not empty do
    let e be the edge in E with the highest weight
    add e to M
    remove e and all edges incident to e from E
```
In any case, the greedy graph matching algorithm finds a maximal matching but it doesn't always find the optimal solution, in contrast to more expensive matching algorithms like the Hungarian Maximum Matching Algorithm, the Blossom Algorithm or the Hopcroft–Karp Algorithm.
Nevertheless, it can be proven that it is a 2-approximation (greedy result >= 1/2 optimal result).

To see that the result is not always optimal, consider the graph in figure 9-3.

Figure 9-3: { (A, B, 1), (B, C, 1+epsilon), (C, D, 1) }

The greedy algorithm for maximum-weight would first take weight 1+epsilon and then stop, missing the optimal weight sum 1+1.

### Stable matchings
Maximality and minimality are not the only interesting optimal matching problems.
Another goal for optimal matching is stability, based on mutual preferences between two groups such as men and women or students and colleges.

Our library implements the classical Gale-Shapely algorithm to solve the famous Stable Matching Problem (aka Stable Marriage Problem).
To simplify, the problem considers two groups of equal size n, modelled as complete bipartite graph.
Each group member has defined a strict preference ordering over all the members of the other group.
A resulting matching would contain n edges, each one relating a different pair.

A matching is stable when there does not exist any pair which both prefer each other to their current partner under the matching.
This leads to a sense of harmony and fairness in the outcome.
It can be proved that... even n,m, ...
### Conclusion
