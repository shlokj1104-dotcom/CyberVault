---
title: ROUTING ALGORITHM
date: 2026-07-06
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 5.2 Routing Algorithms

> **One-Line Summary:** A routing algorithm's job is to determine good (least-cost) paths from senders to receivers through a network modeled as a **graph** of nodes (routers) and weighted edges (physical links); the two dominant approaches take fundamentally opposite philosophies — the **Link-State (LS)** algorithm is centralized, requiring every node to first learn the entire network's topology via broadcast flooding before running Dijkstra's algorithm locally, while the **Distance-Vector (DV)** algorithm is decentralized, iterative, and asynchronous, with each node knowing only its own directly-attached link costs and gradually converging on correct paths purely through repeated gossip with its immediate neighbors — and neither approach is a clean winner: LS suffers from route oscillation under congestion-sensitive costs, while DV suffers from slow convergence and the notorious "count-to-infinity" problem when bad news (a link getting worse) has to propagate through the network.

---

## Core Idea: Routing as a Graph Problem

A routing algorithm's goal is to determine **good paths** (equivalently, **routes**) from senders to receivers, through the network of routers. Typically, a "good" path is one that has the **least cost** — though in practice, real-world concerns like policy issues also come into play (e.g., a rule such as "router x, belonging to organization Y, should not forward any packets originating from the network owned by organization Z").

Whether the network control plane adopts a per-router control approach or a logically centralized approach (Section 5.1), there must always be a well-defined sequence of routers that a packet will cross while traveling from sending host to receiving host. This is precisely why routing algorithms are of fundamental importance — they're one of the field's genuinely top-tier, foundational concepts.

### The Graph Abstraction

A **graph** `G = (N, E)` is a set `N` of nodes and a collection `E` of edges, where each edge is a pair of nodes from `N`. In network-layer routing:

- **Nodes** in the graph represent **routers** — the points at which packet-forwarding decisions are made.
- **Edges** connecting these nodes represent the **physical links** between routers.

> When BGP (the Internet's inter-domain routing protocol, Section 5.4) is studied, nodes will instead represent entire **networks**, and an edge will represent a direct connectivity relationship (known as **peering**) between two such networks — the same graph abstraction, reused at a different scale.

Each edge also has an associated **cost**. Typically, an edge's cost reflects some real-world property of the corresponding physical link — its physical length (e.g., a transoceanic link might cost more than a short-haul terrestrial link), the link speed, or the monetary cost associated with using it. For the purposes of studying the algorithms themselves, edge costs are simply taken as a given input; how they are determined in practice is a separate concern.

### Worked Example: The Reference Network (Figure 5.3)

![[Pasted image 20260706220613.png]]

Consider a 6-node network with nodes `u, v, w, x, y, z` and the following edge costs:

|Edge|Cost|
|---|---|
|u – v|2|
|u – x|1|
|u – w|5|
|v – x|2|
|v – w|3|
|x – w|3|
|x – y|1|
|y – w|1|
|y – z|2|

For any edge `(x, y)` in `E`, we denote `c(x, y)` as the cost of the edge between nodes `x` and `y`. If the pair `(x, y)` does not belong to `E`, we set `c(x, y) = ∞`. Since these graphs are undirected (no direction to an edge) in this discussion, `c(x, y) = c(y, x)`. A node `y` is said to be a **neighbor** of node `x` if `(x, y)` belongs to `E`.

### The Least-Cost Path Problem

A **path** in a graph `G = (N, E)` is a sequence of nodes `(x₁, x₂, ..., xₚ)` such that each of the pairs `(x₁,x₂), (x₂,x₃), ..., (xₚ₋₁,xₚ)` are edges in `E`. The **cost of a path** is simply the sum of all the edge costs along the path:

```
cost(x₁, x₂, ..., xₚ) = c(x₁,x₂) + c(x₂,x₃) + ... + c(xₚ₋₁,xₚ)
```

Given any two nodes, there are typically **many** paths between them, with each path having a cost. One (or more) of these paths is a **least-cost path**.

#### Worked Example: Finding the Least-Cost Path from u to w

In Figure 5.3, the least-cost path between source node `u` and destination node `w` is `(u, x, y, w)` with a path cost of:

```
c(u,x) + c(x,y) + c(y,w) = 1 + 1 + 1 = 3
```

Note that the _direct_ one-hop link from `u` to `w` costs `5` — meaningfully more expensive than the 3-hop route through `x` and `y`. This is the key insight the whole problem revolves around: **the shortest path in terms of hop count is not always the least-cost path**, and vice versa (if all edges have the same cost, then the least-cost path _is_ also the shortest path — the path with the fewest links — but that's a special case, not the general rule).

> **Why This Is Harder Than It Looks:** If you tried to find the least-cost path from `u` to `z` in Figure 5.3 just by inspection, you probably traced a few routes and convinced yourself your answer had the least cost among "all possible paths" — but did you actually check all 17 possible paths between `u` and `z`? Probably not! Such an intuitive calculation is really an example of a **centralized routing algorithm** — the algorithm ran in exactly one location (your brain), with complete information about the network.

### Classifying Routing Algorithms

Routing algorithms can be classified along three largely independent dimensions:

|Dimension|Option A|Option B|
|---|---|---|
|**Information scope**|**Centralized** — computes the least-cost path using complete, global knowledge of the network (all nodes and all link costs as inputs); often called a **link-state (LS)** algorithm|**Decentralized** — computation is carried out in an iterative, distributed manner; no node has complete information about the costs of all network links; called a **distance-vector (DV)** algorithm|
|**Update timing**|**Static** — routes change very slowly over time, typically only via human intervention (e.g., manually editing a link cost)|**Dynamic** — routing paths change as network traffic loads or topology change; can run periodically or in direct response to cost/topology changes; more responsive, but also more susceptible to routing loops and route oscillation|
|**Congestion awareness**|**Load-sensitive** — link costs vary dynamically to reflect the current level of congestion on the underlying link (early ARPAnet routing algorithms worked this way)|**Load-insensitive** — a link's cost does not explicitly reflect its current or recent level of congestion (today's Internet routing algorithms — RIP, OSPF, and BGP — are all load-insensitive)|

Note carefully: **centralized vs. decentralized is about where the computation happens** (Figure 5.1's per-router routing components each running the same algorithm locally, or Figure 5.2's single logically centralized controller, could both in principle run a _centralized_ algorithm like LS) — it is a separate question from whether that computation happens at one physical location.

---

## 5.2.1 The Link-State (LS) Routing Algorithm

### Getting Complete Topology Information

In a link-state algorithm, the network topology and all link costs are known — available as input to the LS algorithm. In practice, this is accomplished by having each node **broadcast link-state packets to all other nodes** in the network, with each link-state packet containing the identities and costs of its attached links. This broadcast mechanism is often implemented via a **link-state broadcast algorithm** (e.g., in the Internet's OSPF routing protocol, discussed in Section 5.3). The result: all nodes end up with an identical, complete view of the network, and each node can then independently run the LS algorithm to compute the same set of least-cost paths as every other node.

### Dijkstra's Algorithm

The link-state routing algorithm presented here is known as **Dijkstra's algorithm**, named after its inventor (a closely related algorithm is Prim's algorithm). Dijkstra's algorithm computes the least-cost path from one node (the **source**, denoted `u`) to all other nodes in the network.

The algorithm is **iterative**, and has the property that after the `k`th iteration, the least-cost paths are known to `k` destination nodes, and among the least-cost paths to _all_ destination nodes, these `k` paths will have the `k` smallest costs.

**Notation:**

|Symbol|Meaning|
|---|---|
|`D(v)`|Cost of the least-cost path from the source node to destination `v`, as of the current iteration|
|`p(v)`|Previous node (neighbor of `v`) along the current least-cost path from the source to `v`|
|`N'`|Subset of nodes; `v` is in `N'` if the least-cost path from the source to `v` is _definitively_ known|

The centralized routing algorithm consists of an **initialization step** followed by a **loop**. The number of times the loop executes equals the number of nodes in the network. Upon termination, the algorithm will have calculated the shortest paths from the source node `u` to every other node in the network.

### Link-State (LS) Algorithm Pseudocode

```
Link-State (LS) Algorithm for Source Node u

 1  Initialization:
 2  N' = {u}
 3  for all nodes v
 4    if v is a neighbor of u
 5      then D(v) = c(u,v)
 6    else D(v) = infinity
 7
 8  Loop
 9  find w not in N' such that D(w) is a minimum
10  add w to N'
11  update D(v) for each neighbor v of w and not in N':
12         D(v) = min(D(v), D(w) + c(w,v))
13         /* new cost to v is either old cost to v, or
14            least path cost to w PLUS cost from w to v */
15  until N' = N
```

### Worked Example: Running Dijkstra's Algorithm Step by Step

Let's compute the least-cost paths from source `u` to all possible destinations in the network from Figure 5.3.

**Initialization:** The currently known least-cost paths from `u` to its directly attached neighbors `v`, `x`, and `w` are initialized to their direct-link costs: `2`, `1`, and `5`, respectively. Note in particular that the cost to `w` is set to 5 (even though we'll soon see a lesser-cost path does indeed exist), since this is the cost of the direct (one-hop) link from `u` to `w`. The costs to `y` and `z` are set to infinity, because they are not directly connected to `u`.

**Iteration 1:** We look among nodes not yet in `N'` and find the node with the least cost. That node is `x`, with a cost of 1, and is thus added to `N'`.

#### Table 5.1 — Running the Link-State Algorithm on the Network in Figure 5.3

|step|N′|D(v), p(v)|D(w), p(w)|D(x), p(x)|D(y), p(y)|D(z), p(z)|
|---|---|---|---|---|---|---|
|0|u|2, u|5, u|1, u|∞|∞|
|1|ux|2, u|4, x|—|2, x|∞|
|2|uxy|2, u|3, y|—|—|4, y|
|3|uxyv|—|3, y|—|—|4, y|
|4|uxyvw|—|—|—|—|4, y|
|5|uxyvwz|—|—|—|—|—|

**Step-by-step walkthrough:**

- **Line 12** of the LS algorithm is then performed to update `D(v)` for all nodes `v`, yielding the results in the second row (Step 1) of Table 5.1. The cost of the path to `v` is unchanged. The cost of the path to `w` (which was 5 at the end of initialization) is found to have a cost of 4 through node `x` (`D(x) + c(x,w) = 1 + 3 = 4`). Hence this lower-cost path is selected, and `w`'s predecessor along the shortest path from `u` is set to `x`. Similarly, the cost to `y` (through `x`) is computed to be `1 + 1 = 2`, and the table is updated accordingly.
- **Iteration 2:** Nodes `v` and `y` are found to have the least-cost paths (both cost 2), and we break the tie arbitrarily, adding `y` to the set `N'`, so `N'` now contains `u`, `x`, and `y`. The cost to the remaining nodes not yet in `N'` — namely `v`, `w`, and `z` — is updated via Line 12, yielding the results in the third row of Table 5.1: `D(w)` improves from 4 to `D(y) + c(y,w) = 2 + 1 = 3`, and `D(z)` becomes reachable for the first time at `D(y) + c(y,z) = 2 + 2 = 4`.
- **And so on...** — the algorithm continues adding one node per iteration until `N' = N`.

**When the LS algorithm terminates**, we have, for each node, its predecessor along the least-cost path from the source node. For each predecessor, we also have _its_ predecessor, and so in this manner we can construct the entire path from the source to all destinations. The **forwarding table** in a node — say node `u` — can then be constructed from this information by storing, for each destination, the **next-hop node** on the least-cost path from `u` to that destination.

![[Pasted image 20260706220749.png]]

#### The Resulting Forwarding Table at Node u

|Destination|Link (next hop)|
|---|---|
|v|(u, v)|
|w|(u, x)|
|x|(u, x)|
|y|(u, x)|
|z|(u, x)|

Notice something striking: **every single destination except `v` is reached via the exact same next hop, `x`** — even though the actual least-cost _paths_ to `w`, `y`, and `z` are all different downstream of `x`. This is precisely the forwarding-table compression benefit already seen in Section 4.2: a router only needs to remember the _immediate next hop_, not the entire remaining path, because every downstream router along that least-cost path will, in turn, have independently computed the correct next hop for its own portion of the journey.

### Computational Complexity of the LS Algorithm

What is the computational complexity of this algorithm? That is, given `n` nodes (not counting the source), how much computation must be done in the worst case to find the least-cost paths from the source to all destinations?

```
Iteration 1: search through n nodes to determine the minimum-cost node
Iteration 2: search through n − 1 remaining nodes
Iteration 3: search through n − 2 remaining nodes
   ...
Total nodes searched across ALL iterations = n(n+1)/2
```

We therefore say that this straightforward implementation of the LS algorithm has **worst-case complexity of order n², written O(n²)**. (A more sophisticated implementation, using a data structure known as a **heap**, can find the minimum in Line 9 in **logarithmic** rather than linear time, reducing the overall complexity.)

### The LS Oscillation Pathology

Before completing our discussion of the LS algorithm, let's consider a **pathology** that can arise. Figure 5.5 shows a simple network topology where link costs are equal to the **load** carried on the link — reflecting the delay that would be experienced. In this example, link costs are **not symmetric**: `c(u,v)` equals `c(v,u)` only if the load carried in both directions on the link `(u,v)` is the same.

#### Worked Example: How Route Oscillation Develops

Setup: node `z` originates a unit of traffic destined for `w`; node `x` also originates a unit of traffic destined for `w`; node `y` injects an amount of traffic equal to `e`, also destined for `w`. The initial routing is shown with link costs corresponding to the amount of traffic carried.

![[Pasted image 20260706221124.png]]

**When the LS algorithm is next run:**

1. Node `y` determines (based on the link costs in the initial routing) that the **clockwise** path to `w` has a cost of 1, while the **counterclockwise** path to `w` (which it had been using) has a cost of `1 + e`. Hence `y`'s new least-cost path to `w` is now **clockwise**.
2. Similarly, `x` determines that its new least-cost path to `w` is _also_ now clockwise, resulting in the costs shown in the second routing configuration (`b`).
3. When the LS algorithm is run **next**, nodes `x`, `y`, and `z` **all** detect a zero-cost path to `w` in the **counterclockwise** direction, and all route their traffic to the counterclockwise routes.
4. The next time the LS algorithm is run, `x`, `y`, and `z` **all** again route their traffic to the **clockwise** routes.

This back-and-forth is the **oscillation**: because all three nodes reassess and switch direction _simultaneously_ every time the algorithm runs, they perpetually overshoot, flip-flopping in lockstep forever.

#### What Prevents This Oscillation?

This pathology can occur in _any_ algorithm (not just LS) that uses a congestion- or delay-based link-cost metric. Possible solutions:

|Solution|Verdict|
|---|---|
|Mandate that link costs not depend on the amount of traffic carried|Unacceptable — this defeats one of routing's actual goals, avoiding highly congested (high-delay) links|
|Ensure not all routers run the LS algorithm at the exact same time|More reasonable — but researchers have found that **routers in the Internet can self-synchronize among themselves** [Floyd Synchronization 1994], even when they initially execute the algorithm with the same period but at different starting instants, converging to run in lockstep over time|

**The practical fix:** have each router **randomize** the time at which it sends out a link advertisement — directly attacking the self-synchronization tendency rather than trying to prevent congestion-based costing altogether.

---

## 5.2.2 The Distance-Vector (DV) Routing Algorithm

### A Fundamentally Different Philosophy

Whereas the LS algorithm is an algorithm using **global** information, the **distance-vector (DV)** algorithm is **iterative, asynchronous, and distributed**:

|Property|What It Means|
|---|---|
|**Distributed**|Each node receives information from one or more of its **directly attached** neighbors, performs a calculation, and distributes the results of its calculation back to its neighbors|
|**Iterative**|This process continues on until no more information is exchanged between neighbors (interestingly, the algorithm is also **self-terminating** — there is no signal that computation should stop; it just stops)|
|**Asynchronous**|The algorithm does not require all nodes to operate in lockstep with each other|

> **Why This Combination Is Genuinely More Interesting:** An asynchronous, iterative, self-terminating, distributed algorithm is much more interesting — and, arguably, more elegant — than a centralized algorithm that requires complete global knowledge up front.

### The Bellman-Ford Equation

Let `dₓ(y)` be the cost of the least-cost path from node `x` to node `y`. The least costs are related by the celebrated **Bellman-Ford equation**:

```
dₓ(y) = min_v { c(x,v) + d_v(y) }        (5.1)
```

where the `min` is taken over all of `x`'s neighbors. The intuition: after traveling from `x` to `v`, if we then take the least-cost path from `v` to `y`, the path cost will be `c(x,v) + d_v(y)`. Since we must begin by traveling to some neighbor `v`, the least cost from `x` to `y` is the minimum of `c(x,v) + d_v(y)` taken over all neighbors of `x`.

#### Worked Example: Verifying Bellman-Ford Against the Dijkstra Result

Let's check the Bellman-Ford equation for source node `u` and destination node `z` in the reference network. Node `u` has three neighbors: `v`, `x`, and `w`. From the graph:

```
d_v(z) = 5      d_x(z) = 3      d_w(z) = 3

Plugging into Equation 5.1, along with c(u,v)=2, c(u,x)=1, c(u,w)=5:

  d_u(z) = min{ 2+5, 1+3, 5+3 }
         = min{ 7, 4, 8 }
         = 4
```

This matches exactly what the Dijkstra/LS algorithm gave us for the same network (Table 5.1: `D(z) = 4` via `y`). This quick cross-check should relieve any skepticism about the Bellman-Ford equation's validity.

### From Equation to Algorithm

The Bellman-Ford equation is not just an intellectual curiosity — it actually has significant practical importance: **the solution to the Bellman-Ford equation provides the entries in node x's forwarding table.** To see this, let `v*` be any neighboring node that achieves the minimum in the equation. Then, if node `x` wants to send a packet to node `y` along a least-cost path, it should first forward the packet to `v*`. Thus, node `x`'s forwarding table would specify `v*` as the **next-hop router** for the ultimate destination `y`.

### The Distance Vector

The basic idea is as follows. Each node `x` begins with `Dₓ(y)`, an estimate of the cost of the least-cost path from itself to node `y`, for all nodes `y` in `N`. Let `Dₓ = [Dₓ(y) : y in N]` be node `x`'s **distance vector**, which is the vector of cost estimates from `x` to all other nodes in `N`.

With the DV algorithm, each node `x` maintains the following routing information:

- For each neighbor `v`, the cost `c(x,v)` from `x` to directly attached neighbor `v`
- Node `x`'s own distance vector, `Dₓ = [Dₓ(y) : y in N]`, containing `x`'s estimate of its cost to all destinations `y` in `N`
- The distance vectors of each of its neighbors, that is, `D_v = [D_v(y) : y in N]` for each neighbor `v` of `x`

### The Distributed, Asynchronous Update Process

From time to time, each node sends a copy of its distance vector to each of its neighbors. When a node `x` receives a new distance vector from any of its neighbors `w`, it saves `w`'s distance vector, and then uses the Bellman-Ford equation to update its own distance vector as follows:

```
Dₓ(y) = min_v { c(x,v) + D_v(y) }        for each node y in N
```

If node `x`'s distance vector has changed as a result of this update step, node `x` will then send its updated distance vector to each of its neighbors — which can, in turn, update their own distance vectors. Miraculously enough, as long as **all** the nodes continue to exchange their distance vectors in an asynchronous fashion, each cost estimate `Dₓ(y)` **converges** to `dₓ(y)`, the actual cost of the least-cost path from node `x` to node `y` [Bertsekas 1991]!

### Distance-Vector (DV) Algorithm Pseudocode

```
Distance-Vector (DV) Algorithm — at each node x:

 1  Initialization:
 2  for all destinations y in N:
 3    Dx(y) = c(x,y)     /* if y is not a neighbor then c(x,y) = infinity */
 4  for each neighbor w
 5    Dw(y) = ? for all destinations y in N
 6  for each neighbor w
 7    send distance vector Dx = [Dx(y): y in N] to w
 8
 9  loop
10  wait (until I see a link cost change to some neighbor w
       or I receive a distance vector from some neighbor w)
11
12  for each y in N:
13    Dx(y) = minv{ c(x,v) + Dv(y) }
14
15  if Dx(y) changed for any destination y
16    send distance vector Dx = [Dx(y): y in N] to all neighbors
17
18  forever
```

**How a node determines its own forwarding table from this process:** In the DV algorithm, a node `x` updates its distance-vector estimate when it either sees a cost change in one of its directly attached links or receives a distance-vector update from some neighbor. But to update its own forwarding table for a given destination `y`, what node `x` really needs to know is not the shortest-path _distance_ to `y`, but instead the neighboring node `v*(y)` that is the **next-hop router** along the shortest path to `y`. As you might expect, the next-hop router `v*(y)` is the neighbor `v` that achieves the minimum in Line 14 of the DV algorithm. (If multiple neighbors `v` achieve the minimum, then `v*(y)` can be any of the minimizing neighbors.) Thus, in Lines 13–14, for each destination `y`, node `x` _also_ determines `v*(y)` and updates its forwarding table for destination `y` accordingly.

**Recall the key contrast with LS:** the LS algorithm is centralized in the sense that it requires each node to first obtain a complete map of the network before running the Dijkstra calculation. The DV algorithm is **decentralized** and does not use such global information at all — the _only_ information a node will ever have is the costs of the links to its directly attached neighbors, and the information it receives from those neighbors.

### Worked Example: DV Algorithm in Operation (Figure 5.6)

![[Pasted image 20260706221624.png]]

Consider a simple 3-node network with nodes `x`, `y`, `z`, and link costs `c(x,y) = 2`, `c(y,z) = 7`, `c(x,z) = 1` (with `x` connected directly to `z` at cost 1 and to `y` at cost 2, and `y`-`z` connected at cost 7). The operation is illustrated in a synchronous manner — all nodes simultaneously receive distance vectors from their neighbors, compute new distance vectors, and inform their neighbors if anything changed.

#### Step-by-Step: Node x's Table Evolution

```
INITIALIZATION (leftmost column):
  Node x's initial routing table (row 1 = its own distance vector):
    D_x = [D_x(x), D_x(y), D_x(z)] = [0, 2, 7]
  Rows 2 and 3 (most recent vectors received FROM y and z) are
  initialized to infinity, since x has not yet received anything.

AFTER FIRST EXCHANGE (middle column):
  x sends D_x = [0, 2, 7] to both y and z.
  x receives D_y and D_z from its neighbors, and recomputes:

    D_x(x) = 0
    D_x(y) = min{ c(x,y) + D_y(y),  c(x,z) + D_z(y) }
           = min{ 2 + 0,  1 + 7 }
           = 2
    D_x(z) = min{ c(x,y) + D_y(z),  c(x,z) + D_z(z) }
           = min{ 2 + 7,  1 + 0 }
           = 1

  Node x's estimate for the least cost to node z has CHANGED
  from 7 to 1 -- because x now knows the direct x-z link exists
  at cost 1, which is cheaper than what it had assumed before.
  At this point x determines: v*(y) = y (direct), v*(z) = z (direct).
```

The process of receiving updated distance vectors from neighbors, recomputing routing-table entries, and informing neighbors of changed costs continues **until no update messages are sent**. At that point, all nodes enter a **quiescent state** — everyone is simply waiting (Lines 10–11 of the DV algorithm) — and the algorithm remains quiescent until a link cost changes.

### Distance-Vector Algorithm: Link-Cost Changes and Link Failure

#### Good News Travels Fast

![[Pasted image 20260706222133.png]]

```
       y
      / \
    1     4->1
    /       \
   z----50----x

Link cost c(y,x) DROPS from 4 to 1 at time t0.
y detects this immediately (direct neighbor) and
updates D_y(x) = 1 right away. GOOD NEWS: fast.
```

When a node running the DV algorithm detects a change in the link cost from itself to a neighbor, it updates its distance vector, and, if there's a change in the cost of the least-cost path, informs its neighbors of its new distance vector. Suppose the link cost from `y` to `x` changes from `4` to `1`. Focusing only on `y` and `z`'s distance table entries to destination `x`, the sequence of events unfolds:

1. At time `t₀`, `y` detects the link-cost change (cost changed from 4 to 1), updates its distance vector, and informs its neighbors of this change since its distance vector has changed.
2. At time `t₁`, `z` receives the update from `y` and updates its table. It computes a new least cost to `x` (decreased from a cost of 5 to a cost of 2) and sends its new distance vector to its neighbors.
3. At time `t₂`, `y` receives `z`'s update and updates its distance table. `y`'s least costs do not change, and hence `y` does not send any message to `z`. The algorithm comes to a quiescent state.

**Only two iterations were required** for the DV algorithm to reach a quiescent state — the good news about the decreased cost between `x` and `y` propagated quickly through the network.

#### Bad News Travels Slowly: The Count-to-Infinity Problem

![[Pasted image 20260706222437.png]]

```
       y
      / \
    4     4->60
    /       \
   z----50----x

Link cost c(y,x) RISES from 4 to 60 at time t0.
y wrongly believes it can still reach x via z at
cost 5 (z's old advertised cost). Routing LOOP
forms; cost estimates creep up 1 at a time across
44 iterations before finally reaching x directly.
BAD NEWS: slow -- the count-to-infinity problem.
```

Now suppose the link cost between `x` and `y` **increases** from `4` to `60`. Before this change: `D_y(x) = 4`, `D_z(y) = 1`, and `D_z(x) = 5`.

#### Worked Example: Tracing the Count-to-Infinity Problem

```
t0: y detects the cost change (4 -> 60). y computes its new
    minimum-cost path to x:

    D_y(x) = min{ c(y,x)+D_x(x), c(y,z)+D_z(x) }
           = min{ 60+0, 1+5 }
           = 6

    Of course, with our GLOBAL view we know this new cost via z
    is WRONG. But y only knows that z LAST told it z could get to
    x at a cost of 5. So y (wrongly) expects that z will be able
    to reach x with a cost of 5. As of t1 we have a ROUTING LOOP:
    to get to x, y routes through z, and z routes through y.

    (A routing loop is like a black hole -- a packet destined for
    x arriving at y or z will bounce back and forth between these
    two nodes forever, or until the forwarding tables change.)

t1: y informs z of its new (wrong) distance vector: D_y(x) = 6.

t2: z receives y's update, which indicates y's new min cost to x
    is 6. z computes:
    D_z(x) = min{ 50+0, 1+6 } = 7
    z's cost to x has increased, so z informs y of ITS new vector.

t3: y receives z's update (z's cost = 7), computes:
    D_y(x) = min{ 60+0, 1+7 } = 8
    y sends its new distance vector.

... and so on. Each round, the estimate creeps up by exactly 1.
This continues for 44 ITERATIONS -- until z eventually computes
the cost of its path via y to be greater than 50 (the cost of
z's DIRECT connection to x). At THAT point, z finally determines
that its least-cost path to x is via its direct link, and the
bad news has finally, slowly, propagated through.
```

**The scenario's real lesson:** what would have happened if the link cost `c(y,x)` had changed from 4 to 10,000, and `c(z,x)` had been 9,999? The two nodes would have looped for **thousands** of iterations before converging. This is exactly why the problem is called the **count-to-infinity problem** — bad news about an increased cost propagates only one small increment per exchange round, potentially taking an enormous number of iterations to finally settle.

### Distance-Vector Algorithm: Adding Poisoned Reverse

The specific looping scenario just described can be avoided using a technique known as **poisoned reverse**. The idea is simple: **if `z` routes through `y` to get to destination `x`, then `z` will advertise to `y` that its distance to `x` is infinity** — that is, `z` will advertise to `y` that `D_z(x) = ∞` (even though `z` knows `D_z(x) = 5` in truth). `z` continues telling this "little white lie" to `y` as long as it routes to `x` via `y` (and will only stop lying about it once it stops routing to `x` via `y`).

#### Worked Example: How Poisoned Reverse Breaks This Specific Loop

```
Before change: y's distance table indicates D_y(x) = 4.
Because of poisoned reverse, z has been telling y (falsely) that
D_z(x) = infinity, since z routes to x via the direct z-x link,
NOT via y.

t0: c(y,x) changes from 4 to 60. y updates its table, but continues
    to route directly to x (albeit at a higher cost of 60), and
    informs z of its new cost to x: D_y(x) = 60.

t1: z receives the update. Since z is now routing via the direct
    (z,x) link at a cost of 50, z shifts to a new least-cost path
    to x via the DIRECT (z,x) link at cost 50 (no change needed,
    it was already using this route). Since the path no longer
    passes through y, z now informs y that its distance is 50.
    Also, since z is now routing directly (not via y), z poisons
    the reverse path FROM y by informing y that D_z(x) = infinity
    (even though z knows D_z(x) = 50 in truth).

t2: After receiving z's update, y updates its table with D_y(x) = 51,
    also poisoning the reverse path from z by informing z that
    D_y(x) = infinity (even though y knows D_y(x) = 51 in truth).
```

**Does poisoned reverse solve the general count-to-infinity problem?** **No, it does not.** You should convince yourself that loops involving **three or more** nodes (rather than simply two immediately neighboring nodes) will **not** be detected by the poisoned reverse technique — the fix only handles the specific two-node mutual-loop case.

---

## A Comparison of LS and DV Routing Algorithms

The DV and LS algorithms take complementary approaches toward computing routing. In the DV algorithm, each node talks to **only** its directly connected neighbors, but it provides its neighbors with least-cost estimates _from itself to all_ the nodes it knows about in the network. The LS algorithm requires **global information**: consequently, each node would need to communicate with **all** other nodes (via broadcast), but it tells them only the costs of its directly connected links.

|Attribute|Link-State (LS)|Distance-Vector (DV)|
|---|---|---|
|**Message complexity**|Requires each node to know the cost of every link in the network — this requires `O(\|N\|·\|E\|)` messages to be sent in total. Also, whenever a link cost changes, the new link cost must be sent to all nodes|Requires message exchanges between directly connected neighbors only. Convergence time depends on many factors — when link costs change, the DV algorithm will propagate the results of the changed link cost only if the new link cost results in a changed least-cost path for one of the nodes attached to that link|
|**Speed of convergence**|Our implementation is an `O(\|N\|²)` algorithm requiring `O(\|N\|·\|E\|)` messages. LS is not itself prone to routing loops during convergence|Can converge slowly, **and can have routing loops while the algorithm is converging**. DV also suffers from the count-to-infinity problem|
|**Robustness**|If a router fails, misbehaves, or is sabotaged — under LS, a router could broadcast an _incorrect_ cost for one of its attached links (but no others). A node computes only its own forwarding table; other nodes are performing similar, separately-derived calculations for themselves. This means route calculations are somewhat separated under LS, providing a **degree of robustness**|Under DV, a node can advertise incorrect least-cost paths to **any or all** destinations. (Indeed, in 1997, a malfunctioning router in a small ISP provided national backbone routers with erroneous routing information. This caused other routers to flood the malfunctioning router with traffic and caused large portions of the Internet to become **disconnected for up to several hours** [Neumann 1997].) More generally, at each iteration, a node's incorrect calculation in DV is passed on to its neighbor, and then indirectly to its neighbor's neighbor on the next iteration — an incorrect node calculation can be **diffused through the entire network** under DV|

**In the end, neither algorithm is an obvious winner over the other** — indeed, **both algorithms are used in the Internet** (OSPF and IS-IS use link-state; RIP and the original ARPAnet used distance-vector; BGP, studied in Section 5.4, uses a related "path-vector" approach).

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**LS's broadcast-flooding of link-state packets assumes honest participants**|A single compromised or misconfigured router broadcasting a false link cost for one of its own attached links can distort routing decisions network-wide, even though LS's "degree of robustness" limits the damage to that router's _own_ advertised links only|Link-state protocols like OSPF typically require some form of authentication on link-state advertisements to prevent unauthorized nodes from injecting false topology information|
|**DV's neighbor-to-neighbor gossip has essentially no built-in error containment**|As the 1997 real-world incident demonstrates, a single malfunctioning or malicious DV-speaking router can inject wildly incorrect distance-vector information that diffuses through the entire network, potentially disconnecting large portions of the Internet for hours|Route filtering, sanity-checking of advertised costs against plausible bounds, and prefix/route validation are essential DV-protocol hardening measures — the underlying algorithm alone provides no defense|
|**The count-to-infinity problem is itself a denial-of-service vector**|An attacker who can manipulate a link cost upward (or simulate a link failure) can potentially trigger prolonged routing instability and temporary unreachability while the network "counts" its way to the correct value|Poisoned reverse and, more robustly, path-vector protocols (BGP, Section 5.4) that carry the _entire_ path rather than just a distance estimate are specifically designed to detect and prevent this class of loop far more generally|
|**Routing loops act as informal denial-of-service / traffic black holes**|Deliberately inducing a routing loop (e.g., via link-cost manipulation) causes legitimate traffic destined for the looped nodes to be silently dropped once TTL/hop-limit expires (Section 4.3.1) — an attacker doesn't need to touch the destination at all to deny service to it|Detecting anomalous TTL-expiration rates or unexpected routing-table churn can serve as an early warning signal for an active routing-loop-inducing attack|

---

## Questions I Still Have

- [ ] The heap-based optimization reduces LS's minimum-finding step (Line 9) from linear to logarithmic time — does this actually change the overall complexity class from O(n²) down to O(n log n), or does the O(|E|) update step (Line 11-12) end up dominating regardless of how fast the minimum-search itself becomes?
- [ ] For LS route oscillation — randomizing link-advertisement timing helps prevent self-synchronization, but could two routers' randomized timers still occasionally realign by chance in a large enough network, causing intermittent (rather than permanent) oscillation?
- [ ] In the count-to-infinity worked example, the "44 iterations" figure came from the specific cost values chosen (4 vs 60, with a 50-cost alternate path) — is there a general formula relating the size of a cost jump to the number of iterations required to converge, so this could be predicted for other cost combinations?
- [ ] Poisoned reverse only fixes 2-node loops — what's the actual production-network mechanism (in RIP or elsewhere) that handles 3+-node loops, given the text says poisoned reverse doesn't solve the general case? Is it simply "wait long enough" (accepting the slow convergence), or is there a real fix deployed?
- [ ] Given that OSPF (LS) and RIP (DV) both exist in real networks and neither is a clean winner — in practice, what factors determine which one an organization actually chooses for its internal (intra-domain) routing today, if load-insensitivity and robustness tradeoffs are roughly a wash?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Graph** `G = (N, E)`|An abstraction of a computer network for routing purposes: `N` is the set of nodes (routers), `E` is the set of edges (physical links)|
|**Path cost**|The sum of all edge costs along a sequence of nodes forming a path between two nodes|
|**Least-cost path**|Among potentially many paths between two nodes, the one with the smallest total path cost|
|**Centralized routing algorithm**|Computes the least-cost path using complete, global knowledge of network connectivity and all link costs; also called a link-state (LS) algorithm|
|**Decentralized routing algorithm**|Computation carried out iteratively and distributedly; no single node has complete information about all network link costs; also called a distance-vector (DV) algorithm|
|**Static routing algorithm**|Routes change slowly, typically only via human/manual intervention|
|**Dynamic routing algorithm**|Routes change automatically and more responsively as traffic loads or topology change; more susceptible to loops and oscillation|
|**Load-sensitive algorithm**|Link costs vary dynamically to reflect current congestion levels on the underlying link|
|**Load-insensitive algorithm**|Link cost does not reflect current/recent congestion (RIP, OSPF, BGP are all load-insensitive)|
|**Link-State (LS) algorithm / Dijkstra's algorithm**|A centralized algorithm requiring complete network topology as input (obtained via link-state broadcast), computing least-cost paths from one source to all other nodes|
|**Link-state broadcast**|The mechanism (e.g. used by OSPF) by which every node floods its directly-attached link costs to all other nodes, giving everyone an identical, complete network view|
|**N′ (LS algorithm)**|The subset of nodes for which the least-cost path from the source is definitively known at a given point in the LS algorithm|
|**D(v), p(v)**|In the LS algorithm: the current known least cost to node `v`, and the predecessor node along that least-cost path|
|**Route oscillation**|A pathology (possible in any load-sensitive routing algorithm) where routes flip back and forth repeatedly because all nodes reassess and switch paths simultaneously|
|**Self-synchronization**|The tendency of independently-timed periodic routers to gradually drift into running their routing algorithm at the same instants, exacerbating oscillation risk|
|**Bellman-Ford equation**|`dₓ(y) = min_v{c(x,v) + d_v(y)}` — relates the least cost from `x` to `y` to the least costs from `x`'s neighbors to `y`|
|**Distance-Vector (DV) algorithm**|An iterative, asynchronous, distributed algorithm where each node maintains and exchanges a "distance vector" of cost estimates with only its directly attached neighbors|
|**Distance vector**|A node `x`'s vector `Dₓ = [Dₓ(y): y in N]` of its own current cost estimates to every other node in the network|
|**Quiescent state**|The state a DV algorithm settles into once no more update messages are being exchanged between neighbors — the algorithm has converged and simply waits|
|**Count-to-infinity problem**|The pathology where "bad news" (an increased link cost) propagates through a DV network only one small increment per exchange round, potentially requiring an enormous number of iterations to converge|
|**Routing loop**|A cycle in the forwarding path (e.g., x routes to y, y routes to x) causing packets destined for the looped destination to bounce indefinitely until TTL expiration or table correction|
|**Poisoned reverse**|A DV technique where a node advertises a cost of infinity, back to the neighbor it is currently routing through, for any destination reached via that neighbor — prevents simple 2-node routing loops, but not longer loops|

---

## Related Concepts

---

→ Next: [[OSPF]]