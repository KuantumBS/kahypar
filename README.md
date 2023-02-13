  <p align="center"><img src="https://user-images.githubusercontent.com/484403/70459249-0fd0d080-1ab4-11ea-833b-17130ecafc0a.png" alt="KaHyPar - Karlsruhe Hypergraph Partitioning" width="60%" height="60%"></p>

License|Linux & macOS Build|Fossa|Zenodo
:--:|:--:|:--:|:--:
[![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](http://www.gnu.org/licenses/gpl-3.0)|[![Build Status](https://github.com/kahypar/kahypar/actions/workflows/kahypar_ci.yml/badge.svg)](https://github.com/kahypar/kahypar/actions/workflows/kahypar_ci.yml)|[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2FSebastianSchlag%2Fkahypar.svg?type=small)](https://app.fossa.com/projects/git%2Bgithub.com%2FSebastianSchlag%2Fkahypar?ref=badge_small)|[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.2555059.svg)](https://doi.org/10.5281/zenodo.2555059)

Code Coverage|Code Quality|Coverity Scan|SonarCloud|Issues
:--:|:--:|:--:|:--:|:--:
[![codecov](https://codecov.io/gh/kahypar/kahypar/branch/master/graph/badge.svg)](https://codecov.io/gh/kahypar/kahypar)|[![Codacy Badge](https://app.codacy.com/project/badge/Grade/93f46f4897c14ad280ed72d460f85c49)](https://www.codacy.com/gh/kahypar/kahypar/dashboard?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=kahypar/kahypar&amp;utm_campaign=Badge_Grade)|[![Coverity Status](https://scan.coverity.com/projects/11452/badge.svg)](https://scan.coverity.com/projects/11452/badge.svg)|[![Quality Gate](https://sonarcloud.io/api/project_badges/quality_gate?project=KaHyPar)](https://sonarcloud.io/dashboard?id=KaHyPar)|[![Average time to resolve an issue](http://isitmaintained.com/badge/resolution/sebastianschlag/kahypar.svg)](http://isitmaintained.com/project/sebastianschlag/kahypar "Average time to resolve an issue")

Table of Contents
-----------

   * [What is a Hypergraph? What is Hypergraph Partitioning?](#what-is-a-hypergraph-what-is-hypergraph-partitioning)
   * [What is KaHyPar?](#what-is-kahypar)
      * [Additional Features](#additional-features)
      * [Experimental Results](#experimental-results)
      * [Additional Resources](#additional-resources)
      * [Projects using KaHyPar](#projects-using-kahypar)
   * [Requirements](#requirements)
   * [Building KaHyPar](#building-kahypar)
   * [Testing and Profiling](#testing-and-profiling)
   * [Running KaHyPar](#running-kahypar)
   * [Using the Library Interfaces](#using-the-library-interfaces)
     * [The C-Style Interface](#the-c-style-interface)
     * [The Python Interface](#the-python-interface)
     * [The Julia Interface](#the-julia-interface)
     * [The Java Interface](#the-java-interface)
   * [Bug Reports](#bug-reports)
   * [Licensing](#licensing)
   * [Contributing](#contributing)



What is a Hypergraph? What is Hypergraph Partitioning?
-----------
[Hypergraphs][HYPERGRAPHWIKI] are a generalization of graphs, where each (hyper)edge (also called net) can
connect more than two vertices. The *k*-way hypergraph partitioning problem is the generalization of the well-known [graph partitioning][GraphPartition] problem: partition the vertex set into *k* disjoint
blocks of bounded size (at most 1 + ε times the average block size), while minimizing an
objective function defined on the nets.

The two most prominent objective functions are the cut-net and the connectivity (or λ − 1)
metrics. Cut-net is a straightforward generalization of the edge-cut objective in graph partitioning
(i.e., minimizing the sum of the weights of those nets that connect more than one block). The
connectivity metric additionally takes into account the actual number λ of blocks connected by a
net. By summing the (λ − 1)-values of all nets, one accurately models the total communication
volume of parallel sparse matrix-vector multiplication and once more gets a metric that reverts
to edge-cut for plain graphs.

<img src="https://cloud.githubusercontent.com/assets/484403/25314222/3a3bdbda-2840-11e7-9961-3bbc59b59177.png" alt="alt text" width="50%" height="50%"><img src="https://cloud.githubusercontent.com/assets/484403/25314225/3e061e42-2840-11e7-860c-028a345d1641.png" alt="alt text" width="50%" height="50%">

What is KaHyPar?
-----------
KaHyPar is a multilevel hypergraph partitioning framework for optimizing the cut- and the
(λ − 1)-metric. It supports both *recursive bisection* and *direct k-way* partitioning.
As a multilevel algorithm, it consist of three phases: In the *coarsening phase*, the
hypergraph is coarsened to obtain a hierarchy of smaller hypergraphs. After applying an
*initial partitioning* algorithm to the smallest hypergraph in the second phase, coarsening is
undone and, at each level, a *local search* method is used to improve the partition induced by
the coarser level. KaHyPar instantiates the multilevel approach in its most extreme version,
removing only a single vertex in every level of the hierarchy.
By using this very fine grained *n*-level approach combined with strong local search heuristics,
it computes solutions of very high quality.
Its algorithms and detailed experimental results are presented in several [research publications][KAHYPARLIT].

### Additional Features
 - **Hypergraph partitioning with variable block weights**

 	KaHyPar has support for variable block weights. If command line option `--use-individual-part-weights=true` is used, the partitioner tries to partition the hypergraph such that each block Vx has a weight of at most Bx, where Bx can be specified for each block individually using the command line parameter `--part-weights= B1 B2 B3 ... Bk-1`. Since the framework does not yet support perfectly balanced partitioning, upper bounds need to be slightly larger than the total weight of all vertices of the hypergraph. Note that this feature is still experimental.

 - **Hypergraph partitioning with fixed vertices**

    Hypergraph partitioning with fixed vertices is a variation of standard hypergraph partitioning. In this problem, there is an additional constraint on the block assignment of some vertices, i.e., some vertices are preassigned to specific blocks prior to partitioning with the condition that, after partitioning the remaining “free” vertices, the fixed vertices are still in the block that they were assigned to. The command line parameter `--fixed / -f` can be used to specify a fix file in [hMetis fix file format](http://glaros.dtc.umn.edu/gkhome/fetch/sw/hmetis/manual.pdf). For a hypergraph with V vertices, the fix file consists of V lines - one for each vertex. The *i*th line either contains `-1` to indicate that the vertex is free to move or `<part id>` to indicate that this vertex should be preassigned to block `<part id>`. Note that part ids start from 0.

    KaHyPar currently supports three different contraction policies for partitioning with fixed vertices:
    1. `free_vertex_only` allows all contractions in which the contraction partner is a *free* vertex, i.e., it allows contractions of vertex pairs where either both vertices are free, or one vertex is fixed and the other vertex is free.
    2. `fixed_vertex_allowed` additionally allows contractions of two fixed vertices provided that both are preassigned to the *same* block. Based on preliminary experiments, this is currently the default policy.
    3. `equivalent_vertices` only allows contractions of vertex pairs that consist of either two free vertices or two fixed vertices preassigned to the same block.

- **Evolutionary framework (KaHyPar-E)**

   KaHyPar-E enhances KaHyPar with an evolutionary framework as described in our [GECCO'18 publication][GECCO'18]. Given a fairly large amount of running time, this memetic multilevel algorithm performs better than repeated executions of nonevolutionary KaHyPar configurations, hMetis, and PaToH. The command line parameter `--time-limit=xxx` can be used to set the maximum running time (in seconds). Parameter `--partition-evolutionary=true` enables evolutionary partitioning.

- **Improving existing partitions**

   KaHyPar uses direct k-way V-cycles to try to improve an existing partition specified via parameter `--part-file=</path/to/file>`. The maximum number of V-cycles can be controlled via parameter `--vcycles=`.
   
- **Partitioning directed acyclic hypergraphs**

   While the code has not been merged into the main repository yet, there exists a [fork](https://github.com/DanielSeemaier/kahypar) that supports acyclic hypergraph partitioning. More details can be found in the corresponding [conference publication](https://epubs.siam.org/doi/abs/10.1137/1.9781611976472.1).


### Experimental Results
We use the [*performance profiles*](https://link.springer.com/article/10.1007/s101070100263) to compare KaHyPar to other partitioning algorithms in terms of solution quality.
  For a set of <img src="https://user-images.githubusercontent.com/484403/80751017-55f10400-8b29-11ea-9d73-be63c0727d39.jpg"/> algorithms and a benchmark set <img src="https://user-images.githubusercontent.com/484403/80751742-a452d280-8b2a-11ea-9f8d-47cbdd9cfccf.jpg"/> containing <img src="https://user-images.githubusercontent.com/484403/80751744-a452d280-8b2a-11ea-9049-5c3fb2d3cc76.jpg"/> instances, the *performance ratio* <img src="https://user-images.githubusercontent.com/484403/80751746-a4eb6900-8b2a-11ea-8413-ff75bccc2296.jpg"/> relates the cut computed by
  partitioner *p* for instance *i* to the smallest minimum cut of *all* algorithms, i.e.,
  <p align="center">
<img src="https://user-images.githubusercontent.com/484403/80750749-f09d1300-8b28-11ea-859d-e3b72f543ed1.png"/>. </p>

The *performance profile* <img src="https://user-images.githubusercontent.com/484403/80751752-a583ff80-8b2a-11ea-8f67-b88625b9e958.jpg"/> of algorithm *p* is then given by the function
  <p align="center">
<img src="https://user-images.githubusercontent.com/484403/80750914-35c14500-8b29-11ea-8e2c-3203b1776a96.jpg"/>.</p>

For connectivity optimization, the performance ratios are computed using the connectivity values <img src="https://user-images.githubusercontent.com/484403/80751741-a3ba3c00-8b2a-11ea-9509-6aafec2ca490.jpg"/> instead of the cut values.
    The value of <img src="https://user-images.githubusercontent.com/484403/80751750-a583ff80-8b2a-11ea-8782-3d44ee478d54.png"/> corresponds to the fraction of instances for which partitioner *p* computed the best solution, while  <img src="https://user-images.githubusercontent.com/484403/80751750-a583ff80-8b2a-11ea-8782-3d44ee478d54.png"/> is the probability
    that a performance ratio <img src="https://user-images.githubusercontent.com/484403/80751746-a4eb6900-8b2a-11ea-8413-ff75bccc2296.jpg"/> is within a factor of <img src="https://user-images.githubusercontent.com/484403/80752228-70c47800-8b2b-11ea-99e5-4524298c8620.jpg"/> of the best possible ratio.
    Note that since performance profiles only allow to assess the performance of each algorithm relative to the *best* algorithm, the <img src="https://user-images.githubusercontent.com/484403/80751747-a4eb6900-8b2a-11ea-846f-1265085ad086.jpg"/> values
    cannot be used to rank algorithms (i.e., to determine which algorithm is the second best etc.).

In our experimental analysis, the performance profile plots are based on the *best* solutions (i.e., *minimum* connectivity/cut) each algorithm found for each instance.
    Furthermore, we choose parameters <img src="https://user-images.githubusercontent.com/484403/80751754-a61c9600-8b2a-11ea-8fdb-3ba461bfe626.jpg"/> for all *p*, *i*, and <img src="https://user-images.githubusercontent.com/484403/80751762-a6b52c80-8b2a-11ea-9f6c-8fb9d00130d3.jpg"/> such that a performance ratio <img src="https://user-images.githubusercontent.com/484403/80751756-a61c9600-8b2a-11ea-88fe-22e2bdd511aa.jpg"/> if and only if algorithm *p* computed an infeasible solution
    for instance *i*, and <img src="https://user-images.githubusercontent.com/484403/80751759-a6b52c80-8b2a-11ea-9cbc-527965565037.jpg"/> if and only if the algorithm could not compute a solution for instance *i* within the given time limit. In our performance profile plots, performance ratios corresponding to *infeasible* solutions will be shown on the x-tick on the *x*-axis, while
instances that could not be partitioned within the time limit are shown implicitly by a line that exits the plot below <img src="https://user-images.githubusercontent.com/484403/80751768-a7e65980-8b2a-11ea-8a69-54dc47ea9eb7.jpg"/>.
    Since the performance ratios are heavily right-skewed, the performance profile plots are divided into three segments with different ranges for parameter <img src="https://user-images.githubusercontent.com/484403/80752228-70c47800-8b2b-11ea-99e5-4524298c8620.jpg"/> to reflect various areas of interest.
    The first segment highlights small values (<img src="https://user-images.githubusercontent.com/484403/80751766-a74dc300-8b2a-11ea-9b54-8e47cd726181.jpg"/>), while the second segment contains results for all instances
    that are up to a factor of <img src="https://user-images.githubusercontent.com/484403/80751763-a74dc300-8b2a-11ea-82f6-d1b644119430.jpg"/> worse than the best possible ratio. The last segment  contains all remaining ratios, i.e., instances for which
    some algorithms performed considerably worse than the best algorithm, instances for which algorithms produced infeasible solutions, and instances which could not be partitioned within
    the given time limit.

In the figures, we compare KaHyPar with PaToH in quality (PaToH-Q) and default mode (PaToH-D), the k-way (hMETIS-K) and the recursive bisection variant (hMETIS-R) of hMETIS 2.0 (p1), [Zoltan using algebraic distance-based coarsening](https://github.com/rsln-s/aggregative-coarsening-for-multilevel-hypergraph-partitioning) (Zoltan-AlgD), [Mondriaan v.4.2.1](http://www.staff.science.uu.nl/~bisse101/Mondriaan/) and the recently published [HYPE](https://arxiv.org/abs/1810.11319) [algorithm](https://github.com/mayerrn/HYPE).

  <p align="center">
	<b>Solution Quality</b>
<img src="https://user-images.githubusercontent.com/484403/67393292-65076000-f5a2-11e9-9605-1dcfd768b045.png" alt="Solution Quality" width="100%" height="50%">
	</p>

  <p align="center">
	<b>Running Time</b>
<img src="https://user-images.githubusercontent.com/484403/67393303-69cc1400-f5a2-11e9-8184-53cf8e5c7cda.png" alt="Running Time" width="100%" height="50%">
	</p>

### Additional Resources
We provide additional resources for all KaHyPar-related publications:

|kKaHyPar-SEA20 /<br/> rKaHyPar-SEA20 |SEA'20 | [Paper](https://drops.dagstuhl.de/opus/frontdoor.php?source_opus=12085) |[TR](https://arxiv.org/abs/2003.12110)| [Slides](http://www.sea2020.dmi.unict.it/SLIDES/Gottesburen.pdf) | [Experimental Results](https://github.com/kahypar/experimental-results/tree/master/sea20) |
|:--|:--|:--:|:--:|:--:|:--:|
|kKaHyPar /<br/> rKaHyPar | - | [Dissertation](https://publikationen.bibliothek.kit.edu/1000105953)| - |[Slides](http://algo2.iti.kit.edu/download/defense_schlag.pdf)|[Experimental Results](https://publikationen.bibliothek.kit.edu/1000105953)|
|KaHyPar-MF /<br/> KaHyPar-R-MF |SEA'18 /<br/> JEA'19|[SEA Paper](SEA'18) /<br/> [JEA Paper](https://dl.acm.org/citation.cfm?doid=3310279.3329872)|[TR](https://arxiv.org/abs/1802.03587)|[Slides](https://algo2.iti.kit.edu/download/sea18-schlag.pdf)|Experimental Results:<br/>[SEA][SEA'18bench] / [JEA][SEA'19bench]|
|KaHyPar-E (EvoHGP)|GECCO'18|[Paper][GECCO'18]|[TR](https://arxiv.org/abs/1710.01968)|[Slides](https://algo2.iti.kit.edu/3506.php)|[Experimental Results][GECCO'18bench]|
|KaHyPar-CA|SEA'17|[Paper][SEA'17]|\-|[Slides](http://algo2.iti.kit.edu/sea17schlag.php)|[Experimental Results][SEA'17bench]|
|KaHyPar-K|ALENEX'17|[Paper][ALENEX'17]|\-|[Slides](http://algo2.iti.kit.edu/3214.php)|[Experimental Results][ALENEX'17bench]|
|KaHyPar-R|ALENEX'16|[Paper][ALENEX'16]|[TR](https://arxiv.org/abs/1511.03137)|[Slides](http://algo2.iti.kit.edu/3034.php)|[Experimental Results][ALENEX'16bench]|

### Projects using KaHyPar

 - [**CoTenGra** - Hyper-optimized Contraction Trees for Large Tensor Networks](https://github.com/jcmgray/cotengra)
 - [**LSOracle** - The Logic Synthesis Oracle](https://github.com/LNIS-Projects/LSOracle)
 - [**Plasmo.jl** - Platform for Scalable Modeling and Optimization](https://github.com/zavalab/Plasmo.jl)
 - [**GraphDot** - A GPU-accelerated Python library for graph similarity computation](https://github.com/yhtang/GraphDot)
 - [**ACQDP** - Alibaba Cloud Quantum Development Platform](https://github.com/alibaba/acqdp)
 - [**DECODE** - A Computational Pipeline for discovering TCR Binding Rules](https://github.com/phineasng/DECODE)
 - [**HybridQ** - A Hybrid Simulator for Quantum Circuits](https://github.com/nasa/hybridq)
 - [**ClusterEnsembles** - Consensus Clustering](https://github.com/c0waro1/ClusterEnsembles)
 - [**TensorCircuit** - Quantum Circuit Simulator](https://github.com/tencent-quantum-lab/tensorcircuit)
 - [**simulate_symacore** - Simulation of the Sycamore quantum supremacy circuit](https://github.com/Fanerst/simulate_sycamore)
 - [**QML** - PennyLane Quantum Machine Learning Demos](https://github.com/PennyLaneAI/qml)

Requirements
-----------
The Karlsruhe Hypergraph Partitioning Framework requires:

 - A 64-bit operating system. Linux, Mac OS X and Windows (through the Windows Subsystem for Linux) are currently supported.
 - A modern, ![C++14](https://img.shields.io/badge/C++-17-blue.svg?style=flat)-ready compiler such as `g++` version 9 or higher or `clang` version 11.0.3 or higher.
 - The [cmake][cmake] build system.
 - The [Boost.Program_options][Boost.Program_options] library and the boost header files. If you don't want to install boost yourself, you can add the `-DKAHYPAR_USE_MINIMAL_BOOST=ON` flag to the cmake command to download, extract, and build the necessary dependencies automatically.
