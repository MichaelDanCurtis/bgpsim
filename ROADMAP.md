# BGPemu2 Roadmap

**Vision:** Educational BGP/OSPF emulator where students learn networking by interacting with realistic router CLIs in hands-on scenarios.

**Status:** Fork established, constitution ratified (v1.2.0), CI foundation in place.

---

## Guiding Principles

1. **Education First:** Every feature serves the learning experience
2. **Anycast Non-Negotiable:** First-class BGP anycast support throughout
3. **Real Network Feel:** Students should feel like they're on production equipment
4. **Test-Driven:** All features built with TDD; scenarios have automated verification

---

## Phase 0: Foundation (Q1 2026) ✅ CURRENT

**Goal:** Establish project governance, CI/CD, and development workflows.

### Completed
- [x] Fork nsg-ethz/bgpsim
- [x] Ratify BGPemu2 Constitution v1.2.0
- [x] CI workflows (lint, test, bench)
- [x] Dependabot, CODEOWNERS
- [x] CONTRIBUTING.md, CHANGELOG.md

### In Progress
- [ ] Merge bootstrap PR to main
- [ ] Tag v0.21.0-edu.0 (first educational fork release)

### Deliverables
- Constitution governs development
- CI gates enforce quality
- Clear contribution path for educators/students

---

## Phase 1: Interactive CLI Foundation (Q2 2026) 🎯 NEXT

**Goal:** Students can connect to individual routers and execute basic show commands.

### Architecture
- Per-router CLI session manager
- Command parser (vendor-neutral base grammar)
- Output formatter (table, JSON, plain text)
- REPL loop with history and tab completion

### Features
- **Networking:** Spawn terminal per router (multiplexed stdin/stdout)
- **Commands (Tier 1):**
  - `show ip bgp` → display BGP table
  - `show ip bgp summary` → neighbor status
  - `show ip route` → routing table
  - `show interfaces` → interface list and status
  - `exit` / `quit`
- **User Experience:**
  - Prompt shows router name: `Router-R1#`
  - Help command (`?`) lists available commands
  - Colored output (green=up, red=down)
  - Response time <100ms for 100-router networks

### Technical Approach
1. Create `bgpsim-cli` crate (library-first)
2. Define `Command` trait and `OutputFormat` enum
3. Implement parser (nom or pest grammar)
4. Add session multiplexer using crossbeam channels
5. Hook into existing `Network` and `ForwardingState` APIs

### Success Criteria
- Student launches emulator, connects to router, runs `show ip bgp`
- Output matches expected BGP table state
- Contract tests verify command outputs
- Documentation: "Your First Router Session" tutorial

### Constitution Check
- ✅ Library-First Modularity: `bgpsim-cli` crate
- ✅ CLI-First: Every command has text I/O contract
- ✅ Test-First: Parser tests, output format tests, integration tests
- ✅ Educational UX: Realistic commands, helpful errors

---

## Phase 2: Configuration Mode (Q3 2026)

**Goal:** Students can configure BGP/OSPF settings and see changes propagate.

### Features
- **Config Mode:**
  - `configure terminal` enters config mode
  - `router bgp <ASN>` / `neighbor <IP> remote-as <ASN>`
  - `router ospf <process-id>` / `network <prefix> area <id>`
  - `ip route <prefix> <next-hop>` for static routes
  - `exit` / `end` to leave config mode
  - `no <command>` to undo config
- **Validation:**
  - Syntax checking with helpful error messages
  - Semantic validation (neighbor IP exists, ASN valid)
  - Preview mode: `configure terminal --dry-run`
- **Config Persistence:**
  - `show running-config` displays current config
  - `write memory` / `copy run start` saves config
  - `reload` restarts router with saved config

### Technical Approach
1. Extend parser with config mode grammar
2. Map CLI commands to `ConfigModifier` mutations
3. Add validation layer before applying changes
4. Implement config serialization/deserialization
5. Event propagation on config change

### Success Criteria
- Student changes BGP neighbor config, sees session re-establish
- Invalid config shows helpful error (e.g., "Neighbor 10.0.0.999 is not a valid IP")
- `show running-config` produces parseable output
- Scenario: "Configure iBGP Full Mesh" tutorial

### Constitution Check
- ✅ Crash Recovery: Invalid configs show errors, don't crash
- ✅ Real-World Fidelity: Config commands mirror Cisco/FRR syntax
- ✅ Reproducible: Config files are declarative, versionable

---

## Phase 3: BGP Anycast Support (Q4 2026) 🎯 CRITICAL

**Goal:** Multiple routers advertise identical prefixes; students observe anycast routing behavior.

### Features
- **Anycast Prefix Advertisement:**
  - Multiple routers can advertise same prefix with different origins
  - Proper AS_PATH handling (prepending, no-export communities)
  - Closest-exit routing based on IGP metrics
- **Traffic Steering:**
  - Visualize which anycast node serves which client
  - Simulate node failure and observe failover
  - BGP communities for traffic engineering (e.g., prefer specific PoP)
- **Educational Scenarios:**
  - CDN distribution (anycast for 1.1.1.1-style service)
  - DNS root server emulation (13 anycast roots)
  - DDoS mitigation (traffic absorption at nearest PoP)

### Technical Approach
1. Extend prefix advertisement to support multiple origins
2. Update BGP path selection to handle anycast prefixes
3. Implement traffic flow visualization (which router serves which prefix)
4. Add anycast-specific show commands:
   - `show ip bgp anycast <prefix>` → lists all advertisers
   - `show ip route anycast <prefix>` → shows closest exit per router
5. Create anycast scenario DSL templates

### Success Criteria
- 3 routers advertise 1.1.1.1/32; students see routing converge
- Shutdown 1 router; students observe traffic shift to next-closest
- Scenario: "Build a Global CDN with Anycast"
- Contract tests verify AS_PATH correctness for anycast

### Constitution Check
- ✅ RFC Compliance: Anycast adheres to BGP RFCs
- ✅ Anycast Non-Negotiable: First-class feature, not a hack
- ✅ Educational UX: Anycast scenarios teach real-world CDN/DNS

---

## Phase 4: Vendor Syntax Modes (Q1 2027)

**Goal:** Optional Cisco IOS, Juniper JunOS, and FRR syntax for certification prep.

### Features
- **Vendor Modes:**
  - `set syntax cisco` → Cisco IOS style
  - `set syntax juniper` → JunOS style (set/delete commands)
  - `set syntax frr` → FRRouting style
  - `set syntax neutral` → vendor-neutral (default)
- **Mode-Specific Behavior:**
  - Cisco: `#` prompt, `?` help, `Tab` completion
  - Juniper: `>` prompt, `set/delete`, `commit` required
  - FRR: `#` prompt, vtysh-style, `write` to save
- **Command Translation:**
  - Core uses neutral AST; vendor modes are facades
  - Same underlying network state

### Technical Approach
1. Define vendor-specific grammars (pest/nom)
2. Translate vendor commands to neutral AST
3. Format output per vendor style (tables, brackets, etc.)
4. Add mode-switching logic in session manager

### Success Criteria
- Student sets `syntax cisco`, sees `Router-R1#` prompt
- CCNP exam prep scenarios work in Cisco mode
- Juniper mode requires `commit` after config changes
- Documentation: "Vendor Mode Quick Reference"

### Constitution Check
- ✅ Multi-Vendor: Modes for Cisco, Juniper, FRR
- ✅ Educational UX: Certification prep alignment

---

## Phase 5: Real-Time & Step-Through Modes (Q2 2027)

**Goal:** Instructors can pause/step through convergence for teaching; real-time mode for SLA lessons.

### Features
- **Simulation Modes:**
  - Event-driven (default): Fast, deterministic
  - Real-time: Matches actual BGP/OSPF timers (30s keepalive, 90s hold)
  - Step-through: Pause after each event, press Enter to continue
  - Replay: Record session, replay with annotations
- **Visualization:**
  - `show simulation events` → pending event queue
  - `show simulation mode` → current mode
  - `simulation step` → advance one event
  - `simulation run` → run until convergence or breakpoint
- **Breakpoints:**
  - `simulation break on route-withdraw`
  - `simulation break on neighbor-down`
  - `simulation watch prefix 10.0.0.0/8`

### Technical Approach
1. Extend event queue with real-time timing wrapper
2. Add pause/resume/step API to Network
3. Implement event inspector and breakpoint logic
4. Create replay serialization format (JSON event log)

### Success Criteria
- Instructor demo: set real-time mode, shutdown link, students watch 90s BGP timeout
- Step-through mode: student advances event-by-event, sees state at each step
- Replay: record lab session, annotate, share with students

### Constitution Check
- ✅ Hybrid Simulation Modes: Event-driven, real-time, step-through, replay
- ✅ Observability: Events are visible and inspectable

---

## Phase 6: Packet Capture & Wireshark Integration (Q3 2027)

**Goal:** Students export BGP/OSPF messages to PCAP for Wireshark analysis.

### Features
- **PCAP Export:**
  - `capture start interface eth0` → begins packet capture
  - `capture stop` → writes to `.pcap` file
  - Includes BGP OPEN, UPDATE, KEEPALIVE, NOTIFICATION
  - Includes OSPF Hello, LSA, DBD packets
- **Wireshark Dissectors:**
  - BGP messages fully dissectable
  - OSPF messages fully dissectable
  - Color rules for route changes (green=announce, red=withdraw)
- **Scenarios:**
  - "Analyze BGP Path Selection in Wireshark"
  - "Observe OSPF LSA Flooding"

### Technical Approach
1. Implement message serialization to BGP/OSPF wire format
2. Write PCAP file format (libpcap/pcapng)
3. Add capture hooks to event queue
4. Test with Wireshark to ensure dissection works

### Success Criteria
- Student exports PCAP, opens in Wireshark, sees BGP UPDATE messages
- Wireshark displays AS_PATH, communities, next-hop correctly
- Scenario: "Trace BGP Convergence with PCAP"

### Constitution Check
- ✅ Real-World Fidelity: PCAP export for Wireshark analysis
- ✅ RFC Compliance: Messages match wire format

---

## Phase 7: Scenario Library & Auto-Grading (Q4 2027)

**Goal:** Instructors create labs with automated verification; students get instant feedback.

### Features
- **Scenario Format (YAML/TOML):**
  ```yaml
  scenario:
    title: "Configure iBGP Full Mesh"
    learning_objectives:
      - Understand iBGP peering requirements
      - Configure BGP neighbors
    topology: topologies/abilene.toml
    initial_state: configs/abilene-ospf.toml
    tasks:
      - instruction: "Configure iBGP sessions between all routers"
        verification:
          - command: "show ip bgp summary"
            expect: "all neighbors Established"
    hints:
      - "Remember to use the same AS number for iBGP"
    solution: solutions/ibgp-full-mesh.toml
  ```
- **Auto-Grading:**
  - `verify scenario ibgp-full-mesh.yaml` → runs checks
  - Students see pass/fail for each task
  - Instructor dashboard shows student progress
- **Scenario Repository:**
  - GitHub repo: `bgpemu2-scenarios`
  - Community-contributed labs
  - Tags: beginner, intermediate, advanced, CCNP, CCIE

### Technical Approach
1. Define scenario schema (serde YAML/TOML)
2. Implement verification engine (run commands, assert outputs)
3. Build scenario runner CLI
4. Create starter scenario pack (10 labs)

### Success Criteria
- Instructor creates scenario in 30 minutes
- Student completes lab, gets instant feedback
- 50+ community scenarios published
- Documentation: "Scenario Authoring Guide"

### Constitution Check
- ✅ Scenario Testing: Learning objectives, verification, instructor guides
- ✅ Reproducible: Scenarios are declarative, versioned

---

## Phase 8: Web UI & Multi-User Support (2028)

**Goal:** Browser-based interface for classrooms; instructor can monitor all students.

### Features
- **Web Interface:**
  - Topology visualization (D3.js or Cytoscape)
  - Per-router terminal in browser (xterm.js)
  - Drag-and-drop topology builder
  - Real-time event viewer
- **Multi-User:**
  - Students connect via browser
  - Instructor dashboard: see all student sessions
  - Snapshot student state for grading
  - Chat/help system
- **Deployment:**
  - Docker Compose for easy classroom setup
  - Kubernetes deployment for large classes

### Technical Approach
1. Build Rust backend (axum or actix-web)
2. WebSocket multiplexer for CLI sessions
3. React/Vue frontend with xterm.js
4. Add authentication and session management

### Success Criteria
- 30 students connect simultaneously
- Instructor sees live topology and student progress
- Deploy on university server in 1 hour
- Documentation: "Classroom Setup Guide"

### Constitution Check
- ✅ Safety & Isolation: Each student in isolated namespace
- ✅ Accessibility: Screen reader support in web UI

---

## Phase 9: Advanced Features (2028+)

**Future enhancements based on community feedback:**

- **Data Plane Simulation:**
  - Packet forwarding simulation
  - MTU, fragmentation, TTL handling
  - Traffic matrix generation
- **BGP Extensions:**
  - BGP Flowspec for DDoS mitigation
  - BGP-LS for SDN integration
  - EVPN for data center scenarios
- **Integration with Real Equipment:**
  - Connect to physical routers via ExaBGP
  - Hybrid virtual/physical topologies
- **AI-Powered Hints:**
  - Student stuck? AI suggests next steps
  - Analyze common mistakes, provide targeted help
- **Gamification:**
  - Leaderboards for fastest convergence
  - Badges for scenario completion
  - Capture-the-flag style network challenges

---

## Success Metrics

### Adoption
- **Year 1:** 10 universities adopt for network courses
- **Year 2:** 1,000 students complete scenarios
- **Year 3:** Community grows to 50+ contributors

### Educational Impact
- Students report increased confidence in BGP/OSPF
- Pass rates improve on CCNP/CCIE exams
- Industry feedback: new hires better prepared

### Technical
- 1k routers, 100k prefixes in <5s convergence
- 99.9% uptime for web platform
- <100ms CLI response time maintained

---

## Contributing to the Roadmap

This roadmap is a living document. Priorities may shift based on:
- Educator feedback and feature requests
- Student learning outcomes
- Community contributions
- Upstream bgpsim changes

**Want to influence the roadmap?**
- Open an issue with `[roadmap]` tag
- Join monthly community calls
- Contribute scenarios or features
- Share educational use cases

---

**Last Updated:** 2025-11-15  
**Constitution Version:** 1.2.0  
**Next Review:** 2026-Q1