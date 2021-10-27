Ntoy graph model
================

ntoy uses disconnected graphs (NG, network graph) as input configuration
Each NG consists of network vertices (nv) and network edges (ne)
Each NV has following aspects:
* discovery: The way ntoy finds NV: it can be link with existing NV, name, mac address, interface type, etc. If few interfaces matches discovery, they may produce one or multiple 'siblings' based on quantifier: (only_one, may, any, all, at_least, at_most).
* converge: The desired state of the interface.
* (graph-like edges with other vertices, network edges, NE).

Graphs
======
Each graph has attribute with name 'mandatory'. If graph is mandatory, but wasn't fully discovered (conditions for discovery wasn't met), ntoy fails. If mandatory is false, undiscovered graph is discarded and ignored.

Vertices
========

The same network entity may participate in the network graph as few NV, when each vertice describe different set of properties. F.e. one NV described as 'interface with default gateway in the default routing table' with convergence properties like 'name', and the same interface participating as another NV discovered as 'ethernet interface' and converged as 'having gso disabled'. NV may be interface, network namespace, address, route, vlan, etc.
Each NV is classified as autonomous if it can exists as disconnected graph (f.e. eth0 may exists as autonomous device even if there is no link with any other NV), or as conditional (f.e. address may not exists without interface). Conditional NV may not exists without having a existential link with autonomous NV. F.e. configuration with NV with ip address '100.64.0.1' must include network interface as existential link, even if this interface is discovered through statement 'iface with ip 100.64.1'. Even if discovery is happening through IP address, interface become an existential
NV, and ip address is conditional. If ntoy configuration wants to move IP from (any) interface to the specific interface.

Edges
=====
Edges (NE) describes relations between NV, f.e. ipv4 address may be linked with interface, and inteface may be linked with namespace (to be in the namespace). NE may be existential (element without edges may not exits, f.e. ip adress without interface), may be 'causal' (f.e. if there is
veth0 with link device veth1, veth1 has causal link), or may be voluntary (interface eth0 has br0 as bridge master).

There is second classification for edges: each edge is either discovered or converged. NE may be discovered and converged at the same time (that means, configuration is exactly as we want it to be).

ntoy execution stages
=====================

* Syntax validation: ntoy checks if configuration syntax is correct.
* Graph validation stage: at this stage ntoy try to check if there are any unsolvable contraditions in the configuration (f.e. converged configuration is not discoverable).
* Discovery stage: all nodes are discovered according for discovery criteria. If some criteria fails (f.e. 'any ethernet', but no ethernet devices found), the whole graph with it is either discarded, or ntoy fails.
* Converge state: at this stage all applicable attributes are applied, converged links are established.
* Confirmation state: configuration is discovered again and matched against graph. If no changes is required, ntoy reports success, if there are some converged links or attributes missed, converge state is repeated (up to `max_iterations`, if come ediges/attributes are not converged yet, ntoy fails.)
