# Feature Specification: BGP Anycast Support

**Feature Branch**: `003-bgp-anycast`  
**Created**: 2025-11-15  
**Status**: Draft  
**Constitution Version**: 1.2.0  
**Priority**: CRITICAL (NON-NEGOTIABLE per Constitution Principle VII)  
**Input**: "BGP Anycast Support: Multiple routers advertise identical prefixes from different locations with proper AS_PATH handling, traffic steering, and failover simulation. Educational scenarios include CDN distribution, DNS root servers, and DDoS mitigation. Students observe closest-exit routing based on IGP metrics and can simulate node failures."

## User Scenarios & Testing

### User Story 1 - Basic Anycast Advertisement (Priority: P1)

A student configures 3 routers (New York, London, Tokyo) to advertise the same anycast prefix `1.1.1.1/32` and observes that client routers select the closest anycast node based on IGP cost.

**Why this priority**: This is the absolute MVP for anycast - multiple origins advertising the same prefix. Without this, anycast feature doesn't exist.

**Independent Test**: Configure 3 routers to announce `1.1.1.1/32`, query BGP table from client router, verify 3 paths exist with different next-hops, verify routing table selects closest based on IGP metric.

**Acceptance Scenarios**:

1. **Given** routers R1, R2, R3 each advertise `1.1.1.1/32` with origin `i`, **When** client router R4 runs `show ip bgp 1.1.1.1/32`, **Then** output shows 3 paths with next-hops pointing to R1, R2, R3
2. **Given** R4 has OSPF costs: R1=10, R2=50, R3=100, **When** R4 runs `show ip route 1.1.1.1/32`, **Then** routing table shows R1 as next-hop (lowest IGP cost)
3. **Given** R1 crashes, **When** R4 runs `show ip route 1.1.1.1/32` after convergence, **Then** routing table now shows R2 as next-hop (next-best path)
4. **Given** all 3 anycast nodes are up, **When** R4 sends traffic to `1.1.1.1`, **Then** traffic simulation shows packets forwarded to R1 (closest)

---

### User Story 2 - AS_PATH Handling and Loop Prevention (Priority: P1)

A student verifies that anycast prefixes maintain proper AS_PATH attributes when traversing multiple ASes, preventing routing loops.

**Why this priority**: Critical for correctness - BGP fundamentals require proper AS_PATH handling. Without this, anycast advertisements could create loops.

**Independent Test**: Configure anycast prefix advertised from AS 100 and AS 200, trace paths through AS 300, verify AS_PATH grows correctly and loops are prevented.

**Acceptance Scenarios**:

1. **Given** R1 (AS 100) and R2 (AS 200) both advertise `1.1.1.1/32`, **When** R3 (AS 300) receives both announcements, **Then** `show ip bgp 1.1.1.1/32` shows two paths with AS_PATH "100" and "200" respectively
2. **Given** R3 (AS 300) re-advertises best path to R4 (AS 400), **When** R4 checks AS_PATH, **Then** AS_PATH shows "300 100" (AS 300 prepended)
3. **Given** R1 (AS 100) tries to receive anycast prefix via R3 that includes AS 100 in path, **When** BGP processes update, **Then** route is rejected due to AS_PATH loop detection
4. **Given** student uses AS_PATH prepending `bgp as-path prepend 100 100` on R1, **When** R3 compares paths, **Then** R1's path has longer AS_PATH and R2's path is preferred (if IGP costs equal)

---

### User Story 3 - CDN Scenario - Content Delivery (Priority: P2)

A student builds a simulated CDN with anycast nodes in 5 geographic locations (US-East, US-West, EU, Asia, Australia) and observes traffic locality - clients connect to nearest PoP.

**Why this priority**: Primary educational use case for anycast - students learn real-world CDN architecture. Requires basic anycast (P1) working first.

**Independent Test**: Load TopologyZoo "Internet2" topology, configure 5 anycast nodes, generate traffic from 10 client locations, verify each client routes to geographically nearest node.

**Acceptance Scenarios**:

1. **Given** CDN anycast IP `203.0.113.1/32` advertised from 5 PoPs, **When** client in Boston queries route, **Then** next-hop points to US-East PoP
2. **Given** US-East PoP goes offline, **When** Boston client re-checks route after convergence (simulated 60s), **Then** next-hop switches to US-West PoP
3. **Given** student enables traffic visualization, **When** 100 simulated clients send requests, **Then** heatmap shows traffic distribution proportional to IGP distances
4. **Given** student adds 6th PoP in Africa, **When** African clients query route, **Then** they select new PoP instead of routing to EU

---

### User Story 4 - DNS Root Server Simulation (Priority: P2)

A student recreates the 13 DNS root server letter assignments (A-M) using anycast, where each "letter" is actually multiple physical servers advertising the same IP.

**Why this priority**: Iconic anycast use case - educational value high, helps students understand real internet infrastructure. Builds on CDN scenario patterns.

**Independent Test**: Configure 13 anycast prefixes (simulating A.root through M.root), assign multiple instances per letter, query from global clients, verify closest instance is selected per letter.

**Acceptance Scenarios**:

1. **Given** `198.41.0.4/32` (A.root) advertised from 5 locations worldwide, **When** client in Tokyo queries route, **Then** next-hop points to Tokyo A.root instance
2. **Given** student shuts down 1 of 5 A.root instances, **When** affected clients reconverge, **Then** they seamlessly fail over to next-closest A.root instance
3. **Given** student displays anycast summary with `show ip bgp anycast`, **Then** output shows all 13 root letters with instance counts and geographic distribution
4. **Given** DDoS attack simulated on L.root prefix, **When** traffic overwhelms nearest instance, **Then** scenario demonstrates how anycast absorbs attack locally (traffic doesn't transit to other instances)

---

### User Story 5 - Anycast Traffic Engineering with Communities (Priority: P3)

A student uses BGP communities to influence anycast path selection, steering traffic to prefer specific PoPs (e.g., load balancing or maintenance scenarios).

**Why this priority**: Advanced anycast technique - useful for real-world operations but not essential for basic anycast understanding.

**Independent Test**: Configure anycast nodes to tag advertisements with communities, set local preference based on community matching, verify traffic steering.

**Acceptance Scenarios**:

1. **Given** US-East PoP tags anycast routes with community `100:10`, **When** upstream router matches community and sets local-pref 150, **Then** US-East path is preferred over other PoPs (default local-pref 100)
2. **Given** student drains US-West PoP for maintenance by setting community `100:50`, **When** routers match community and lower local-pref to 50, **Then** traffic shifts away from US-West to other PoPs
3. **Given** student uses no-export community on anycast prefix, **When** prefix reaches eBGP peer, **Then** route is not advertised further (localized anycast)
4. **Given** student views BGP table with `show ip bgp 1.1.1.1/32`, **Then** output includes community column showing traffic engineering tags

---

### User Story 6 - Failover and Convergence Analysis (Priority: P2)

A student simulates anycast node failure and measures convergence time - how long before traffic reroutes to next-best anycast instance.

**Why this priority**: Critical operational concern for anycast - students need to understand failure scenarios and recovery timing. Differentiates from basic "it works" testing.

**Independent Test**: Start topology with 3 anycast nodes, generate continuous traffic from client, crash node 1, measure time until traffic switches to node 2 (should match BGP hold timer ~90s in real-time mode).

**Acceptance Scenarios**:

1. **Given** 3 anycast nodes with client routing to node 1, **When** node 1 interface fails, **Then** BGP session tears down and client reconverges to node 2 within 90 seconds (real-time mode) or immediately (event-driven mode)
2. **Given** student uses step-through simulation mode, **When** node failure occurs, **Then** student can step through BGP NOTIFICATION, route withdrawal, best-path recalculation events one-by-one
3. **Given** student queries `show ip bgp anycast 1.1.1.1/32 history`, **When** command executes, **Then** output shows timeline of path changes: initially 3 paths, then withdrawal from node 1, then 2 remaining paths
4. **Given** node 1 recovers and rejoins, **When** BGP session re-establishes, **Then** client routes back to node 1 (if IGP cost still lowest) after route advertisement propagates

---

### Edge Cases

- **Identical AS_PATH Tiebreaker**: When 2 anycast nodes have equal IGP cost and AS_PATH length, BGP must use router-ID tiebreaker consistently (document behavior)
- **Asymmetric Routing**: Client routes to anycast node A, but return traffic may use different path - scenario must clarify this is expected anycast behavior
- **Anycast Prefix Overlap**: If router advertises both `1.1.1.0/24` and anycast `1.1.1.1/32`, routing table must prefer /32 (longest prefix match)
- **Flapping Prevention**: If anycast node oscillates up/down rapidly, implement route dampening to prevent BGP churn (or document as future enhancement)
- **Empty Anycast Group**: If all instances of an anycast prefix withdraw, routing table must remove prefix entirely (no blackhole)
- **AS_PATH Prepending Limits**: Validate that excessive AS_PATH prepending (>255 ASNs) is handled per RFC limits
- **Anycast in iBGP**: Document special considerations for anycast within a single AS (next-hop-self, route-reflector implications)

## Requirements

### Functional Requirements

- **FR-001**: System MUST allow multiple routers to advertise the same prefix with different origin routers
- **FR-002**: System MUST maintain separate BGP paths for each anycast origin in RIB (Routing Information Base)
- **FR-003**: System MUST select best anycast path based on standard BGP decision process: local-pref > AS_PATH length > origin > MED > IGP cost > router-ID
- **FR-004**: System MUST prepend AS numbers correctly when anycast prefix traverses eBGP peering
- **FR-005**: System MUST prevent AS_PATH loops when anycast prefix is re-advertised through originating AS
- **FR-006**: System MUST handle anycast prefix withdrawal from one origin without affecting other origins
- **FR-007**: System MUST support CLI command `show ip bgp anycast <prefix>` to display all anycast origins with AS_PATH, next-hop, and status
- **FR-008**: System MUST support CLI command `show ip route anycast <prefix>` to show which anycast instance is selected for forwarding
- **FR-009**: System MUST support traffic simulation showing which anycast node receives traffic from each client
- **FR-010**: System MUST recalculate best path and update forwarding table when anycast node fails (BGP session down)
- **FR-011**: System MUST preserve BGP attributes (communities, local-pref, MED) consistently for anycast prefixes
- **FR-012**: System MUST support BGP communities on anycast announcements for traffic engineering
- **FR-013**: System MUST respect no-export community to prevent anycast leaking beyond specific scope
- **FR-014**: System MUST support scenario templates for CDN, DNS root server, and DDoS mitigation anycast patterns
- **FR-015**: System MUST log anycast-related events (new origin advertised, origin withdrawn, best path changed) for debugging
- **FR-016**: System MUST validate anycast configurations to prevent common mistakes (duplicate router-IDs, missing IGP)
- **FR-017**: System MUST measure and report convergence time when anycast origin fails in real-time mode
- **FR-018**: System MUST support anycast in both iBGP and eBGP contexts with proper next-hop handling

### Key Entities

- **AnycastPrefix**: A prefix advertised by 2+ routers; tracks all origin routers, current best path per observer router, historical path changes
- **AnycastOrigin**: Represents one router advertising an anycast prefix; includes router ID, AS number, BGP attributes (local-pref, communities)
- **AnycastPathSelection**: Logic implementing BGP decision process to select best anycast origin considering IGP cost, AS_PATH length, router-ID
- **TrafficSteering**: Simulator component that routes packets to selected anycast origin based on FIB (Forwarding Information Base)
- **AnycastScenario**: Pre-built educational scenario (CDN, DNS root, DDoS) with topology, anycast config, and learning objectives

## Success Criteria

1. **Correctness**: Anycast path selection matches RFC 4271 BGP decision process 100% (verified by contract tests comparing to reference implementation)
2. **Educational Clarity**: Students can explain "why did traffic go to node A instead of node B?" by examining IGP cost and AS_PATH (90% comprehension in post-lab survey)
3. **Failover Realism**: Anycast node failure triggers route reconvergence matching real-world BGP timers (hold time ~90s) in real-time simulation mode
4. **Scenario Richness**: 3 pre-built anycast scenarios (CDN, DNS root, DDoS) ship with release, each with learning objectives and auto-grading
5. **Performance**: Anycast path selection adds <10ms overhead to BGP route processing for topologies with up to 100 anycast origins
6. **Stability**: No infinite loops or routing blackholes introduced by anycast configuration errors

## Assumptions

- IGP (OSPF or static routes) is configured before anycast BGP - students must set up underlay first
- Anycast prefixes are typically /32 (host routes) but system supports any prefix length
- AS_PATH loop detection uses standard "my AS in path" check per RFC 4271
- Traffic simulation is control-plane only (data plane packet forwarding deferred to future phases)
- Real-time convergence timing uses default BGP timers (keepalive 30s, hold 90s) unless student configures otherwise
- Anycast scenarios assume IPv4; IPv6 anycast is functionally identical and can be added later
- Students understand basic BGP before attempting anycast scenarios (prerequisite: iBGP/eBGP fundamentals)

## Dependencies

- **Upstream bgpsim**: Provides BGP RIB (multiple paths per prefix), AS_PATH manipulation, community support
- **Phase 1 (Interactive CLI)**: Required to query anycast paths with `show ip bgp anycast <prefix>`
- **OSPF/IGP Metric API**: Must be able to query IGP cost from router A to router B for tie-breaking
- **Event Queue**: Must support event-driven and real-time simulation for convergence timing
- **Topology Loading**: Must support loading TopologyZoo datasets or custom topologies with geographic hints

## Out of Scope

- **Data Plane Simulation**: Actual packet forwarding with TTL decrement, fragmentation - Phase 9 (Advanced Features)
- **Anycast Load Balancing**: Active health checks or dynamic load distribution - beyond BGP scope
- **IPv6 Anycast**: Functionally identical to IPv4, deferred to avoid scope creep
- **BGP Flowspec for DDoS**: Advanced traffic filtering - separate feature
- **Anycast DNS Protocol**: Actual DNS query/response simulation - we simulate routing only
- **Cross-AS Anycast Optimization**: PEERING DB integration, IXP route servers - too advanced for Phase 3
- **Anycast Monitoring/Alerting**: Prometheus metrics, alerting rules - deferred to observability phase

## Open Questions

None. All ambiguities resolved with informed defaults documented in Assumptions.

---

**Constitution Compliance Check**

- ✅ **Anycast NON-NEGOTIABLE (Principle VII)**: This feature IS the constitutional mandate - first-class anycast support
- ✅ **RFC Compliance (Principle VI)**: AS_PATH handling follows RFC 4271; communities follow RFC 1997
- ✅ **Test-First (Principle I)**: Contract tests will verify path selection matches RFC behavior
- ✅ **Library-First (Principle II)**: Anycast logic added to `bgpsim` core library with stable API
- ✅ **Educational UX (Principle XI)**: CDN, DNS root, DDoS scenarios teach real-world anycast use cases
- ✅ **Reproducible Scenarios (Principle IX)**: Anycast scenarios are declarative, seed-driven, versionable
- ✅ **Observability (Principle V)**: Anycast events logged; `show ip bgp anycast` provides visibility
- ✅ **Performance Targets (Principle X)**: <10ms overhead for anycast path selection measured in benchmarks

---

**Next Steps**

1. Review specification with network operators to validate real-world scenarios
2. Run `/speckit.plan` to generate implementation plan
3. Extend `bgpsim` RIB to track multiple origins per prefix (if not already supported)
4. Implement `show ip bgp anycast <prefix>` command in CLI
5. Create 3 anycast scenario templates (CDN, DNS root, DDoS)
6. Write contract tests comparing anycast behavior to real BGP router (FRR or BIRD)
