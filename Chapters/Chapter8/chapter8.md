## Link analysisLink analysis is a technique used to evaluate relationships between nodes. Link analysis is used on several fields,such as search engines, fraud detection, among others. There is several algorithms of different kinds to perform link analysis.Here we are only going to focus on the Hyperlink-Induced Topic Search \(HITS\) algorithm.This algorithm was originally developed to rate web pages. But, nowadays modern search engines do not usethis algorithm since there is more advanced techniques. HITS has been also used to identify the important classes that should becommented in a large software system or the classes that a developer should read to get an insight of the key classes.### Hyperlink-Induced Topic Search \(HITS\) algorithmHyperlink-Induced Topic Search \(HITS\) algorithm, also knows as Hubs and Authorities,is an algorithm that rates every the nodes of a graph. Every node has a hub and a authority score. A hub is a node that may notbe relevant but references relevant nodes. An authority is a node that contains relevant information.The algorithm does the following:1. Assign to each node a hub and an authority score equal to 1.1. Run the authority update rule for each node.1. Run the hub update rule for each.1. Normalize the values by dividing each Hub score by the square root of the sum of the squares of all Hub scores, and dividing each Authority score by the square root of the sum of the squares of all Authority scores.1. Repeat from the second step as necessary.The update rules are simple:**Authority update rule**Update each node's authority score to be equal to the sum of the hub scores of each node that points to it.**Hub update rule**Update each node's hub score to be equal to the sum of the authority scores of each node that it points to.### HITS implementationThe Pharo implementation is as follows. The `k` number is the number of times that the scores are going to beupdated. The default value is `20` but it can also be set manually.```AIHits >> run

	self initializeNodes.
	k timesRepeat: [
		nodes do: [ :node | self computeAuthoritiesFor: node ].
		nodes do: [ :node | self computeHubsFor: node ].
		self normalizeScores ].
	^ nodes``````AIHits >> initializeNodes

	"Here we are using float instead of int because of the normalization."
	nodes do: [ :n |
		n auth: 1.0.
		n hub: 1.0 ]``````AIHits >> computeAuthoritiesFor: aNode

	aNode auth:
		(aNode incomingNodes
			inject: 0
			into: [ :sum :node | sum + node hub ])``````AIHits >> computeHubsFor: aNode

	aNode hub:
		(aNode adjacentNodes
			inject: 0
			into: [ :sum :node | sum + node auth ])``````AIHits >> normalizeScores

	| authNorm hubNorm |
	authNorm := 0.
	hubNorm := 0.

	nodes do: [ :node |
		authNorm := authNorm + node auth squared.
		hubNorm := hubNorm + node hub squared ].

	authNorm := authNorm sqrt.
	hubNorm := hubNorm sqrt.

	"To avoid dividing by 0"
	authNorm = 0 ifTrue: [ authNorm := 1.0 ].
	hubNorm = 0 ifTrue: [ hubNorm := 1.0 ].

	nodes do: [ :n |
		n auth: n auth / authNorm.
		n hub: n hub / hubNorm ]```### Case studyHere we calculate the hubs and authorities scores for all the nodes of the graph shown in Figure *@hits@* with 3 iterations.![A graph to play with the HITS algorithm.](figures/hits.pdf width=30&label=hits)```nodes := #( 'A' 'B' 'C' 'D' ).
edges := #( #( 'A' 'B' ) #( 'A' 'C' ) #( 'A' 'D' ) #( 'B' 'C' )
            #( 'B' 'D' ) #( 'C' 'A' ) #( 'C' 'D' ) #( 'D' 'D' ) ).
hits := AIHits new.
hits
	nodes: nodes;
	edges: edges from: #first to: #second;
  k: 3.
nodes := hits run```If we inspect the nodes, these are the scores calculated after 3 iterations.```('A' auth: 0.17 hub: 0.65)
('B' auth: 0.27 hub: 0.54)
('C' auth: 0.49 hub: 0.41)
('D' auth: 0.81 hub: 0.34)```### Weighted HITSThere are cases where the Hits algorithm does not behave as expected and sometimes the Hits algorithm puts 0 as valuesfor the hubs and authorities. Using weights in a graph helps in obtaining better results. Establishing the weights is aresponsibility of the user.For more information, you can read these papers:- _Modifications of Kleinberg's HITS Algorithm Using Matrix Exponentiation and Web Log Records_ by Miller et al. % ${cite:Mill01a}$- _An Improved Weighted HITS Algorithm Based on Similarity andPopularity_ by Zhang et al. % ${cite:Zhan07a}$In terms of implementation, it is only necessary to multiply the weights with the scores in each iteration.That means changing `computeAuthoritiesFor:` and `computeHubsFor:` methods.This is done in `AIWeightedHits` class.```AIWeightedHits >> computeAuthoritiesFor: aNode

	aNode auth: (aNode incomingEdges
			 inject: 0
			 into: [ :sum :edge | sum + (edge weight * edge from hub) ])``````AIWeightedHits >> computeHubsFor: aNode

	aNode hub: (aNode outgoingEdges
			 inject: 0
			 into: [ :sum :edge | sum + (edge weight * edge to auth) ])```### ConclusionEven if the HITS algorithm is not used anymore in the modern search engines, it is a very good algorithm forhaving a first look on how to classify links according to their relevance in the network.