Social Network Analysis (SNA)

Summarize the key points in my `sna.Rmd` notes. These notes derive from the snaJSS.pdf paper, which can be downloaded from the schedule. The summary should be about two or three pages in length, i.e., be brief. You can view this assignment as assisting in your review for the final.

Add your discussion below. You can use R code as appropriate.

-----

This assignment aims at providing a summary of Social Network Analysis using the 'sna' package developed by Carter Butts.
sna is part of the statnet project, which provides software tools for the analysis, simulation, and visualization of network data.
The main idea is to illustrate the functions used in the analysis of relational data and their relative importance. The social newtork is a collecion of data that can be represented and phrased in the form of graphs, which is why "visualization" is an important concept in the network.

#Social network.

The basic units of a network are called actors or nodes. They can be people, or websites, or whatever “things” you are considering, and are often indicated as a single dot in a visualization. The relationships between the actors are referred to as relational ties or edges. For example, an instance of liking someone or being friends would be indicated by an edge. We refer to pairs of actors as dyads, and triplets of actors as triads. For example, if we have an edge between node A and node B, and an edge between node B and node C, then triadic closure would be the existence of an edge between node A and node C.

Social network we know, is a network of mostly relational data that consists of a fixed set of entities called vertices and a multiset of relationships among those entities called edges. Of several relationship type, most common is dyadic relationships, where edges consist of (possibly ordered) two-element multisets on the set of vertices. Furthermore we can define elements of an edge as as its endpoints, with the first element known as the tail or sender and the second known as the head (or receiver) in the ordered case. An edge whose endpoints are identical is called a loop.

As mentioned above, graphs are essential representation of the network, where a graph of social network relation can be defined as a combination of an edge set, $E$, with vertex set $V$(denoted by $G = (V,E)$). The size, or order of a graph is the number of elements in its vertex set (denoted by $|V|$, where $|·|$ is the cardinality operator).

Types of graph::

graph $G$ is said to be:  

* undirected: if edges are unordered multisets;

* directed or digraph: if edges are ordered multisets.

In either cases,  graph is simple if it has no loops and none of its edge has multiplicity greater than one. 

adjacency matrix; i.e., a square matrix, $A$, is a poplular method used to describe the relational data, whose elements are defined such that $a_{ij}$ is the value of the $(i,j)$ edge (or ${i,j}$ edge, in the undirected case) in the corresponding graph. Adjacency matrices are extremely convenient to work with, however they are inefficient for large, sparse graphs therefore here the use of network or sparse matrix SparseM objects may be preferred.

sna has a range of tools for random graph generation. Chief among these is rgraph, considered as a “workhorse” function for simulating deviates from both homogeneous and inhomogeneous Bernoulli graph distributions. Given a set of tie probabilities (which may be specified by graph or by edge), it generates one or more graphs whose edge states are independent Bernoulli trials conditional on the specified parameters.

The output in  sna is adjacency matrix or array even though iaccept network and related objects as input.

Along with 'rgraph' which generates graphs, `gden` is another important sna function which returns the density of one or more input graphs where the observed densities closely match their expectations. 


##eval.edgeperturbation for visualization

In social network analysis which is also called as 'relational analysis' due to relational data it works on, visualization and manipulation of relational data is a central task. The important function to note here is, eval.edgeperturbation that is a wrapper function which computes the difference in the value of a graph statistic resulting from forcing the selected edge or edges to be present, versus forcing them to be absent (holding all other edges constant). It is not only useful in producing extensive computation for simulation but also inference from exponential random graph processes. eval.edgeperturbation is extremely flexible, and can be used with any graph-level index function. 

> Drawback.

only drawback it possesses is that it is partially inefficiency meaning it is useful only for simple, exploratory applications, and does not replace the specialized (but less flexible) change-score functions used within packages such as `ergm`.

## egocentric network

The egocentric network of vertex $v$ in graph $G$ is defined as $G[v \cup N(v)]$, i.e., the subgraph of $G$ induced by $v$ and its neighborhood. This graph has `ego.extract` function which, for a given input graph (or set thereof) extracts the egocentric networks for one or more vertices. This is particularly handy for computing local structural properties, or for simulating the effects of ego net sampling. However in case of directed graphs, it requires modifications where it is further possible to specify the use of incoming, outgoing, or combined neighborhoods for generating the induced subgraphs. 

> Disadvantage

It provides the ability to assess the internal structure and properties however it does not provide for computation on attributes (i.e., exogenous covariates) of vertex neighbors. This functionality is supplied by `gapply`. For each vertex in its input set, `gapply` first identifies all members of its neighborhood; neighborhoods may be in, out, or combined, and higher-order neighborhoods may be selected (as discussed below). Once each neighborhood has been identified, `gapply` applies a user-specified function to the neighbors’ covariates (which may be supplied as a numeric vector). This provides a very quick and easy way to calculate properties such as the size of a given vertex’s 3rd-order neighborhood, the fraction of its alters with a given characteristic, the average value of its alters on a specified covariate, etc.

##Gplot
Network visualization is a fundamental aspect of social network analysis and is an important feature of `sna`. The primary "workhorse"" routine for graph visualization within `sna` is `gplot`, which displays an input network using a two-dimensional layout. Many options are available to `gplot`, including the ability to specify characteristics such as size, color, and shape for individual vertices, edges, and edge labels.


## classes of indices
social network analysis is rich with descriptive indices of various sorts, of which the most commonly used indices may be divided into two classes:  
* node-level indices (NLIs), which express properties of the positions of particular vertices;  
* graph-level indices (GLIs), which express properties of entire graphs.  


Here we wil discuss nodelevel indices.

### Node-level indices

The most well-developed node-level indices are the centrality indices.

####Types

Following are the most widely used centrality indices are:  

Degree: it is given by $c_d(v,G) = |N(v)|$ for undirected $G$. In the directed case, three notions of degree are generally encountered:   
Outdegree: $c_{d^+} (v, G) \equiv |N_+(v)|$;  
Indegree $c_{d^-} (v, G) \equiv |N_-(v)|$;  
Total or “Freeman” degree $c_{d^t} (v, G) = c_{d^+} (v, G) + c_{d^-} (v, G)$.

Betweenness-measures the extent to which a given vertex lies on non-redundant geodesics between third parties. The index is formally defined as $c_{b} (v, G) \equiv \sum_{(v^{\prime}, v^{\prime\prime}) \subset V | v} \frac{g^{\prime}(v^{\prime}, v, v^{\prime\prime}, G)}{g(v^{\prime}, v^{\prime\prime}, G)}$, where $g(v, v^\prime, G)$ is the number of $(v, v^{\prime})$ geodesics in $G$ and $g(v, v^{\prime}, v^{\prime\prime}, G)$ is the number of $(v, v^{\prime\prime})$ geodesics in $G$ containing $v^{\prime}$.
Closeness - is given by $c_c(v, G) \equiv \frac{n - 1}{\sum_{v^{\prime} \in V} d(v, v^{\prime})}$ where $d(v, v^{\prime})$ is the geodesic distance from vertex $v$ to vertex $v^{\prime}$. The geodesic distance between two vertices in a graph is the number of edges in a shortest path connecting them. Closeness is ill-defined on graphs which are not strongly connected.

These indices are implemented in `sna` via the `degree`, `betweenness`, and `closeness` functions. 

#Eignevector centrality
The eigenvector centralities is based on spectral properties of the graph adjacency matrix $A$. Eigenvector centrality (implemented in `sna` via `evcent`) is simply the absolute value of the principal eigenvector of $A$. This can be interpreted as a measure of "coreness" (or membership in the largest dense cluster), "recursive"  degree, i.e., v is central to the extent to which it has many ties to other central nodes, or of the ability of v to reach other vertices through a multiplicity of short walks.



