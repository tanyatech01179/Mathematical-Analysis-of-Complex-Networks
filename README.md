# Project 2 — Mathematical Network Analysis Using Graph Theory

**Two case studies: Zachary's Karate Club (34 nodes, 78 friendships) and the American college football
network (115 teams, 613 games, 12 conferences).**

A complete, runnable network-analysis project that implements every graph algorithm **from scratch**
with Python + NumPy, then uses it to find influential nodes, detect communities, and stress-test the
network. NetworkX appears only in one verification cell, as an independent referee.

| | |
|---|---|
| **Notebook** | [`Project2_Network_Analysis_Graph_Theory.ipynb`](Project2_Network_Analysis_Graph_Theory.ipynb) |
| **Datasets** | Zachary's Karate Club (1977) and the college football network (2000), both embedded — no downloads |
| **Stack** | Python 3, NumPy, Matplotlib, pandas (NetworkX optional) |
| **Runtime** | under a minute on a free Colab CPU |
| **Outputs** | 10 figures in [`figures/`](figures), printed result tables |

---

## Problem statement

> Social networks, communication systems, and organisational structures contain complex relationships.
> Develop a mathematical network analysis system that identifies important nodes, communities, and
> relationships within large networks.

This project answers five questions:

1. **Who is important?** — degree, closeness and betweenness centrality.
2. **Who is influential in a random-walk sense?** — PageRank.
3. **Does the network split into communities?** — Girvan–Newman and two spectral-clustering variants.
4. **What breaks if those people leave?** — targeted-removal simulation, measured against both the
   remaining and the original network.
5. **Do the conclusions generalise?** — the whole pipeline is repeated on a larger football network.

The karate club is ideal for the first four questions because the answer is known: instructor
**Mr. Hi (node 0)** and administrator **Officer (node 33)** led rival factions of 16 and 18 members.
The football network provides an out-of-sample test with 12 known conferences, 3.4× the nodes and
7.9× the edges.

---

## Quick start

### Google Colab (recommended)

1. Open [Google Colab](https://colab.research.google.com/).
2. `File → Upload notebook` and choose `Project2_Network_Analysis_Graph_Theory.ipynb`.
3. `Runtime → Run all`.

Nothing to install. Both datasets are embedded, so the notebook works offline.

### Local

```bash
pip install -r requirements.txt
jupyter notebook Project2_Network_Analysis_Graph_Theory.ipynb
```

---

## Results at a glance

### Zachary's karate club

| Metric | Value |
|---|---|
| Nodes / edges | 34 / 78 |
| Average degree / density | 4.59 / 0.139 |
| Diameter / average path length | 5 / 2.408 |
| Connected components | 1 |
| Most connected node | **33** (degree 17, $C_D = 0.515$) |
| Strongest bridge | **0** (betweenness $C_B = 0.438$) |
| Highest PageRank | **33** ($PR = 0.1009$), then **0** ($0.0970$) |
| Girvan–Newman first split | 2 communities, $Q = 0.360$, **ARI = 0.882** |
| Girvan–Newman max $Q$ | 5 communities, $Q = 0.401$, ARI = 0.469 |
| Spectral, Fiedler variant ($k=2$) | 2 communities, $Q = 0.371$, **ARI = 1.000** |
| Spectral, literal NJW ($k=2$) | 2 communities, $Q = 0.360$, ARI = 0.882 |
| Spectral, Fiedler variant ($k=4$) | 4 communities, $Q = 0.401$, ARI = 0.523 |
| Spectral, literal NJW ($k=4$) | 4 communities, $Q = 0.420$, ARI = 0.541 |
| Remove top-5 betweenness nodes | giant component **34.5 % of remaining / 29.4 % of original** |
| Remove 5 random nodes | giant component **97.0 % of remaining / 82.7 % of original** |

### American college football network

| Metric | Value |
|---|---|
| Nodes / edges | 115 / 613 |
| Average degree / density | 10.66 / 0.0935 |
| Diameter / average path length | 4 / 2.508 |
| Connected components | 1 |
| Conferences | 12 |
| Top brokers (betweenness) | Notre Dame, Brigham Young, Navy, Louisiana Tech, Central Michigan |
| Best modularity in the sweep | $k = 11$, $Q = 0.603$, ARI = 0.857 |
| Known $k = 12$ | $Q = 0.582$, ARI = 0.824 (10 seeds: **0.835 ± 0.023**) |
| Literal NJW at $k = 12$ | $Q = 0.6005$, ARI = 0.897 |
| Remove 50 of 115 nodes | remaining graph still **100 % connected** |
| Remove 60 of 115 nodes | betweenness attack 100 % of remaining / 47.8 % of original |

### Pearson vs Spearman: all six pairs

Rank correlation is systematically higher than raw-score correlation when centrality is skewed. The
top-10 overlap is the share of nodes common to both shortlists.

| Pair | Karate Pearson | Karate Spearman | Karate top-10 | Football Pearson | Football Spearman | Football top-10 |
|---|---|---|---|---|---|---|
| degree / closeness | 0.772 | 0.895 | 90 % | 0.291 | 0.344 | 30 % |
| degree / betweenness | 0.915 | 0.905 | 80 % | 0.281 | 0.337 | 20 % |
| degree / PageRank | 0.998 | 0.979 | 100 % | 0.989 | 0.892 | 80 % |
| closeness / betweenness | 0.718 | 0.898 | 90 % | 0.817 | 0.842 | 50 % |
| closeness / PageRank | 0.742 | 0.825 | 90 % | 0.286 | 0.277 | 40 % |
| betweenness / PageRank | 0.923 | 0.879 | 80 % | 0.299 | 0.344 | 20 % |

### Top-10 by composite influence (karate)

Ranked by the average rank across degree, closeness, betweenness and PageRank (priority queue):

```
0, 33, 2, 32, 31, 1, 8, 13, 3, 23
```

Node **0** (the instructor) has the highest betweenness; node **33** (the administrator) has the
highest degree and PageRank. Both are bridges between the two factions — which is exactly why the club
split when their conflict escalated.

### Degree / betweenness centrality

![Centrality network and top-10 betweenness](figures/02_centrality_network.png)

### Detected communities

![Detected communities](figures/04_communities.png)

The Fiedler variant recovers the historical split **exactly**; Girvan–Newman's first cut gets 33 of 34
members right. The literal Ng–Jordan–Weiss variant, which includes the trivial eigenvector, gets ARI
0.882. Maximising modularity further produces 5 sub-communities — a useful reminder that the
maximum-$Q$ partition is not always the historically meaningful one.

### Robustness

![Fragmentation under targeted attacks](figures/05_robustness.png)

Removing the top brokers shatters the karate network; removing random members barely dents it. The two
panels give the same experiment with two denominators: the left asks whether the remaining graph stays
connected, the right how much of the original graph survives. This is the mathematical argument for
targeted immunisation and for protecting single points of failure.

### The second network

![Football network](figures/07_football_structure.png)

![Cross-dataset centrality comparison](figures/08_centrality_comparison.png)

![Football community structure](figures/09_football_communities.png)

![Cross-dataset robustness](figures/10_cross_dataset_robustness.png)

The football network is larger and far harder to fragment, but its centrality rankings diverge much
more than the karate club's. That is the main cross-dataset finding: the algorithms generalise, while
the identity of the important nodes and the resilience of the network do not.

---

## What is implemented (all from scratch)

| Algorithm | Mathematics | Complexity | Notebook section |
|---|---|---|---|
| Graph (adjacency list) | $G=(V,E)$, hash maps | $O(V+E)$ memory | 5 |
| BFS / DFS | queue / stack traversal | $O(V+E)$ | 6 |
| Connected components | repeated BFS | $O(V+E)$ | 6 |
| Union-Find | disjoint sets, path compression | $O(E\,\alpha(V))$ | 6 |
| Degree centrality | combinatorics | $O(V+E)$ | 7 |
| Closeness centrality | BFS all-pairs | $O(V(V+E))$ | 7 |
| Betweenness centrality | Brandes' algorithm | $O(VE)$ | 7 |
| PageRank | Markov chain + power iteration | $O(k(V+E))$ | 8 |
| Modularity $Q$ | observed vs expected edges | $O(V+E)$ per partition | 9 |
| Girvan–Newman | edge betweenness, divisive | $O(m^2V)$ | 9 |
| Spectral clustering | Laplacian eigenvectors + k-means++ | $O(V^3)$ | 9 |
| ARI | partition agreement | $O(V+k^2)$ | 9 |
| Pearson / Spearman | linear and rank correlation | $O(V)$ per pair | 12 |
| Force-directed layout | Fruchterman–Reingold | $O(V^2 + E)$ per iteration | 7 |
| Attack simulation | repeated BFS | $O(k(V+E))$ | 10, 13 |

**Math used:** graph theory, combinatorics (path counting), probability (random walk), linear algebra
(adjacency/Laplacian spectra), connectivity, modularity, rank statistics.
**DSA used:** adjacency lists, hash maps, queues, stacks, union-find, priority queue (`heapq`), binary
heaps, recursion-free graph traversal.

---

## Verification

Every score is compared against NetworkX on the same unweighted edge list:

| Quantity | Max absolute difference |
|---|---|
| Degree centrality | `0.00e+00` |
| Closeness centrality | `0.00e+00` |
| Betweenness centrality | `5.20e-18` |
| PageRank | `1.00e-11` |
| Modularity | `5.55e-17` |
| Number of components | `0` |

The PageRank difference is just a different stopping tolerance; both converge to the same vector
(the notebook also cross-checks against the exact linear solve, difference `2.67e-13`).

> **Note on the dataset:** NetworkX's built-in `karate_club_graph()` stores Zachary's interaction
> **weights** (and in recent versions `pagerank()` uses them by default). This project uses the
> standard **unweighted** projection (78 edges); the verification cell builds its referee graph from
> the same edge list, which is why the unweighted PageRank values here differ from a one-line
> `nx.pagerank(nx.karate_club_graph())` call.

---

## Notebook contents

| # | Section | What happens |
|---|---|---|
| 1–4 | Introduction, concepts, setup | problem, pipeline, imports |
| 5 | Graph representation | `Graph` class, embedded karate and football edge lists |
| 6 | Traversal | BFS, DFS, shortest path, components, Union-Find, diameter |
| 7 | Centrality | degree, closeness, Brandes betweenness + network plots |
| 8 | PageRank | power iteration, convergence, linear-solve check |
| 9 | Community detection | modularity, Girvan–Newman, Fiedler vs literal NJW, ARI |
| 10 | Robustness | targeted vs random removal, both denominators |
| 11–12 | Verification & correlation | NetworkX cross-check, Pearson + Spearman + top-10 overlap |
| 13 | Second network | football structure, centrality, spectral sweep, stability, robustness |
| 14–19 | Ranking, discussion, conclusion | composite top-10, complexity, experimental discussion, references |

---

## Project brief coverage

| Requirement | Where |
|---|---|
| **Graph theory** — represent relationships | `Graph` class, adjacency matrix |
| **BFS** — explore connected nodes | Section 6 |
| **DFS** — identify connected components | Section 6 |
| **Union-Find** — detect components | Section 6 |
| **Hash Maps** — store node information | `Graph.adj`, `Graph.meta`, algorithm state |
| **Priority Queue** — rank important nodes | Section 14 (`heapq`) |
| **Degree centrality** | Section 7 |
| **Betweenness centrality** | Section 7 (Brandes) |
| **Closeness centrality** | Section 7 |
| **PageRank** | Section 8 |
| **Community detection** | Section 9 (two methods) |
| **AI/ML clustering** | Spectral embedding + k-means++ |
| **Real-world applications** | Sections 10, 13, 17 |
| **Cross-dataset validation** | Section 13 |

---

## Figures

| File | Content |
|---|---|
| [`01_adjacency_and_degrees.png`](figures/01_adjacency_and_degrees.png) | adjacency matrix + degree distribution |
| [`02_centrality_network.png`](figures/02_centrality_network.png) | network coloured by betweenness + top-10 bar chart |
| [`03_pagerank.png`](figures/03_pagerank.png) | power-iteration convergence + PageRank ranking |
| [`04_communities.png`](figures/04_communities.png) | Fiedler vs literal NJW at $k=2$ and $k=4$ |
| [`05_robustness.png`](figures/05_robustness.png) | karate fragmentation with both denominators |
| [`06_centrality_correlation.png`](figures/06_centrality_correlation.png) | Pearson vs Spearman for karate |
| [`07_football_structure.png`](figures/07_football_structure.png) | 115-team network + top brokers |
| [`08_centrality_comparison.png`](figures/08_centrality_comparison.png) | Pearson/Spearman heatmaps for both networks |
| [`09_football_communities.png`](figures/09_football_communities.png) | spectral sweep + recovered vs true conferences |
| [`10_cross_dataset_robustness.png`](figures/10_cross_dataset_robustness.png) | karate vs football attack curves |

---

## Limitations and extensions

* **Betweenness** is $O(VE)$ — sample sources or use approximation for million-node graphs.
* **Girvan–Newman** is $O(m^2V)$ — replace with Louvain/Leiden for production scale.
* **Spectral clustering** needs an eigendecomposition — use randomised SVD, Nyström, or node2vec /
  GraphSAGE embeddings on large graphs.
* **k-means is stochastic.** At $k=12$ on football, ten seeds give ARI $0.83 \pm 0.02$; a single run is
  not *the* answer.
* **Modularity has a resolution limit**, which is one reason the best-$Q$ $k$ (10–11) differs from the
  known 12 conferences.
* **Conference labels are not a perfect partition** — teams play games outside their conference, so ARI
  cannot reach 1 even with a perfect algorithm.
* The graphs are **unweighted and undirected**; weighted shortest paths (Dijkstra) generalise the same
  code.
* Ground truth exists only for these classic datasets; on real data, validate with link prediction or
  partition stability.

Natural next steps: temporal networks, link prediction, influence maximisation (independent cascade),
edge-removal and cascading-failure models, and interactive visualisation with Plotly or PyVis.

---

## References

1. W. W. Zachary, *An information flow model for conflict and fission in small groups*, Journal of
   Anthropological Research, 1977.
2. U. Brandes, *A faster algorithm for betweenness centrality*, Journal of Mathematical Sociology, 2001.
3. L. Page, S. Brin, R. Motwani, T. Winograd, *The PageRank citation ranking*, Stanford, 1998.
4. M. E. J. Newman, M. Girvan, *Finding and evaluating community structure in networks*, Phys. Rev. E, 2004.
5. M. Girvan, M. E. J. Newman, *Community structure in social and biological networks*, PNAS, 2002.
6. A. Y. Ng, M. I. Jordan, Y. Weiss, *On spectral clustering: analysis and an algorithm*, NeurIPS, 2001.
7. S. Fortunato, *Community detection in graphs*, Physics Reports, 2010.
