# Sage Triples

Sage Triples represents a Knowledge Graph (KG) as a collection of subject, object
relation (s-r-o) triples. Nodes are a colleection of unique subject and object
while the edges are the relationship between each nodes.

## Triples

A *subject* node is connected to an *object* node via a *relation* edge. A triple
is a tuple of subject, relation and object (s-r-0). For example `(Simon, plays, tennis)`.
A series of this s-r-o triples make up a Knowledge Graph. Also within a KG, you
often have cases where the subjects have more than one connections to other objects.
For example `[(Simon, plays, tennis), (Simon, lives in, Melbourne)]`.
Here Simon shares connection with both tennis and Melbourne. This scenario where
a node has multiple connections is called an entity subgraph. Ideally, multiple
subgraphs make up a Knowledge Graph, but depending on the kind of KG, things can
get really complicated where a node has only one connection that’s disconnected
from the rest of the KG.

## Sage Triples & Graph Neural Networks

`sage-triples` aims to provide an easy way of converting your s-r-o triples into
a format that is easily loaded into Graph Neural Networks for Machine Learning
purposes. To achieve this, each subgraphs in the Knowledge Graph could return the
following as an [`ndarray`].

- **Node Features** `(n_nodes, n_nodes)`
  Since we have `n_nodes` nodes and each node is mapped to its corresponding
  embedding vector with vector size `embedding_dim`. We can stack all features
  in a matrix of shape `(n_nodes, embedding_dim)`. This matrix is represented
  using the [`ndarray` crate][ndarray-crate].

- **Adjacency Matrix** `(n_nodes, n_nodes)`
  A typical s-r-o consist of subjects & objects which represents the nodes and
  a relation which represents the edges describing the connection of a node with
  another node. If we unroll this an construct a matrix `a` of `(n_nodes, n_nodes)`,
  we can express this mathematically where each entry `a[i, j]` is non-zero if
  there exists an edge going from node `i` to node `j`, and zero otherwise.

  We can represent `a` as a dense 2-D `ndarray` or a [`sprs`] sparse matrix of
  shape `(n_nodes, n_nodes)`. For an undirected graph `a[i, j] == a[j, i]` while
  this might not be the case for directed graphs.

- **Edge Features** `(n_edges, n_edges)`
  Just as we've done for our adjacency matrix which expresses that a node is connected
  to another node. We can go a step further to describe the weight of the edge
  connections. By constructing a sparse matrix of `(n_edges, n_edges)` we can
  achieve this.

  However, we would end up with a matrix where most of the values are 0. This is
  not an ideal use of memory. Instead we could implement a sparse matrix by only
  storing the values of the non-zero entries in a list and asuuming that if a pair
  of indices is missing from the list then its corresponding value will be 0.
  This is called the *COOrdinate format* and it is used by many neural network
  libraries.

  For example, the edge feature for our above graph with 8 edges::

  ```sh
    [[   0  325  706    0    0    0    0    0]
     [ 325    0 2732    0    0    0    0    0]
     [ 706 2732    0 6691    0    0    0    0]
     [   0    0 6691    0    0    0    0    0]
     [   0    0    0    0    0    0    0    0]
     [   0    0    0    0    0    0    0    0]
     [   0    0    0    0    0    0    0    0]
     [   0    0    0    0    0    0    0    0]]
  ```

  can be represented in COOrdinate format as follows:

  ```sh
    R,  C,  V
    (0, 1)  325
    (0, 2)  706
    (1, 0)  325
    (1, 2)  2732
    (2, 0)  706
    (2, 1)  2732
    (2, 3)  6691
    (3, 2)  6691
  ```

  where `R` indicated the "row" indices, `C` the columns, and `V` the non-zero
  values `e[i, j]`. For example in the third line, we see there's an edge that
  goes **from node 1 to node 0** with the value of 325.

- **Edge Embeddings** `(n_edges, embedding_dim)`

  So far we've only been able to establish that node `i` is connected to node `j`
  (with some value). However, what we also need is a way to encode our edge
  relationships into an embedding space, just like we did for the nodes with the
  node features.

  Just like we did for the node features, we also construct a matrix of shape
  `(n_edges, embedding_dim)` where each edge is mapped to its corresponding
  embedding vector.

[`ndarray`]: https://github.com/rust-ndarray/ndarray
[ndarray-crate]: https://docs.rs/ndarray/latest/ndarray/index.html
[`sprs`]: https://docs.rs/sprs/latest/sprs/
<!-- [ndarray_npy]: https://docs.rs/ndarray-npy/latest/ndarray_npy/index.html -->