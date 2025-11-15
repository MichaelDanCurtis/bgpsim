# Feature Specification: Interactive CLI Foundation

**Feature Branch**: `002-interactive-cli`  
**Created**: 2025-11-15  
**Status**: Draft  
**Constitution Version**: 1.2.0  
**Input**: "Interactive CLI Foundation: Students can connect to individual routers and execute basic show commands like 'show ip bgp', 'show ip route', 'show interfaces' with vendor-neutral syntax. Each router has its own terminal session with prompt showing router name. Response time must be under 100ms for networks up to 100 routers. Supports help commands and colored output."

## User Scenarios & Testing

### User Story 1 - View BGP Routes on Single Router (Priority: P1)

A student connects to a simulated router and views its BGP routing table to understand which routes the router has learned and their attributes (next-hop, AS path, local preference).

**Why this priority**: This is the absolute minimum viable product - a single router, single command interaction. Without this, students cannot interact with the emulator at all.

**Independent Test**: Can be fully tested by launching emulator with 1 router pre-configured with BGP, connecting to its CLI, typing `show ip bgp`, and verifying output matches the router's current BGP state.

**Acceptance Scenarios**:

1. **Given** emulator is running with router R1 that has 3 BGP routes learned, **When** student types `show ip bgp` at R1's CLI, **Then** system displays table with 3 routes showing prefix, next-hop, AS path, and status
2. **Given** student is at R1's CLI prompt, **When** student types `show ip bgp 10.0.0.0/8`, **Then** system displays only routes matching that prefix
3. **Given** router R1 has no BGP routes, **When** student types `show ip bgp`, **Then** system displays "No BGP routes" message
4. **Given** router R1 receives a new BGP update, **When** student types `show ip bgp` again, **Then** system displays updated routing table with new route

---

### User Story 2 - View IP Routing Table (Priority: P1)

A student examines the complete routing table (including BGP, OSPF, static, and connected routes) to understand how packets would be forwarded.

**Why this priority**: Critical companion to BGP table - students need to see the "winning" routes that actually forward traffic, not just BGP candidates.

**Independent Test**: Can be fully tested by launching emulator with router having mixed route sources (BGP + OSPF + static), typing `show ip route`, and verifying all route types appear with correct protocol indicators.

**Acceptance Scenarios**:

1. **Given** router R1 has routes from BGP (B), OSPF (O), and static (S), **When** student types `show ip route`, **Then** system displays all routes with protocol codes and administrative distances
2. **Given** student is viewing routing table, **When** student types `show ip route ospf`, **Then** system displays only OSPF routes
3. **Given** student types `show ip route 10.1.1.0/24`, **Then** system displays the specific matching route with full details (next-hop, interface, metric)

---

### User Story 3 - Multi-Router Session Management (Priority: P2)

A student connects to multiple routers simultaneously to compare their routing tables and understand distributed routing behavior.

**Why this priority**: Essential for understanding BGP convergence and distributed protocols, but can be deferred until single-router interaction works perfectly.

**Independent Test**: Launch emulator with 3 routers, open 3 terminal sessions (one per router), execute commands in each, verify prompts show correct router names and outputs are independent.

**Acceptance Scenarios**:

1. **Given** emulator is running with routers R1, R2, R3, **When** student opens CLI session to R1, **Then** prompt shows `Router-R1#`
2. **Given** student has sessions open to R1 and R2, **When** student types `show ip bgp` in R1 session, **Then** only R1's routing table is displayed, R2 session is unaffected
3. **Given** student has 3 sessions open, **When** BGP convergence event occurs, **Then** student can refresh each router's view independently to observe propagation

---

### User Story 4 - Help and Command Discovery (Priority: P2)

A student unfamiliar with CLI syntax uses help commands to discover available commands and their syntax.

**Why this priority**: Improves usability for beginners, but experienced students can work without it initially.

**Independent Test**: At any CLI prompt, type `?` or `help`, verify list of available commands appears with brief descriptions.

**Acceptance Scenarios**:

1. **Given** student is at router CLI prompt, **When** student types `?`, **Then** system displays list of all available commands with one-line descriptions
2. **Given** student types `show ?`, **When** command executes, **Then** system displays subcommands available under `show` (ip, interfaces, etc.)
3. **Given** student types `show ip ?`, **When** command executes, **Then** system displays available options (bgp, route, ospf, etc.)
4. **Given** student types incomplete command `show i`, **When** student presses Tab, **Then** system auto-completes to `show ip` or shows options if ambiguous

---

### User Story 5 - Interface Status Monitoring (Priority: P3)

A student checks interface states to diagnose connectivity issues or understand topology.

**Why this priority**: Useful for troubleshooting scenarios, but less critical than routing table inspection for initial learning.

**Independent Test**: Launch emulator, type `show interfaces`, verify list shows interface names, IP addresses, and up/down status.

**Acceptance Scenarios**:

1. **Given** router R1 has 3 interfaces (eth0, eth1, eth2), **When** student types `show interfaces`, **Then** system displays table with interface name, IP address, status (up/down), and MTU
2. **Given** student types `show interfaces eth0`, **When** command executes, **Then** system displays detailed information for eth0 only
3. **Given** interface eth1 is administratively down, **When** student views interfaces, **Then** eth1 shows status "admin down" in red color
4. **Given** link failure occurs on eth0, **When** student types `show interfaces`, **Then** eth0 status changes to "down" immediately

---

### User Story 6 - BGP Neighbor Status (Priority: P3)

A student examines BGP session states to understand peering relationships and troubleshoot session failures.

**Why this priority**: Important for advanced BGP scenarios, but students can learn basic routing before tackling neighbor states.

**Independent Test**: Launch emulator with BGP neighbors configured, type `show ip bgp summary`, verify neighbor IPs, ASNs, and session states appear.

**Acceptance Scenarios**:

1. **Given** router R1 has 2 BGP neighbors (R2 and R3) in Established state, **When** student types `show ip bgp summary`, **Then** system displays neighbor table with IPs, remote AS, state "Established", and route counts
2. **Given** BGP session to R2 is down (Active state), **When** student views summary, **Then** R2's state shows "Active" in yellow/orange color
3. **Given** student types `show ip bgp neighbors 10.0.0.2`, **When** command executes, **Then** system displays detailed information for that specific neighbor (timers, capabilities, statistics)

---

### Edge Cases

- **Empty Topology**: When emulator has no routers configured, CLI should display friendly error: "No routers available. Load a topology first."
- **Router Not Found**: If student tries to connect to non-existent router name, display: "Router 'R99' not found. Available routers: R1, R2, R3"
- **Long Output**: When routing table exceeds terminal height (>100 routes), implement pagination with "Press SPACE for more, Q to quit"
- **Concurrent Updates**: If BGP update arrives while student is viewing `show ip bgp` output, ensure display completes without corruption (lock snapshot)
- **Malformed Commands**: If student types invalid syntax like `show bgp ip`, display helpful error: "Invalid command. Did you mean 'show ip bgp'? Type '?' for help."
- **Non-ASCII Input**: If student pastes special characters or emoji, sanitize input and display: "Invalid characters detected. CLI supports ASCII only."
- **Session Timeout**: If student is idle for >30 minutes, display warning before disconnect (or disable timeout for educational use)
- **Performance Degradation**: If command takes >100ms on network <=100 routers, log warning but still display output

## Requirements

### Functional Requirements

- **FR-001**: System MUST provide a CLI session per router with prompt format `Router-<NAME>#` where NAME is the router's identifier
- **FR-002**: System MUST support command `show ip bgp` to display BGP routing table with columns: Network, Next Hop, Metric, LocPrf, Weight, Path, Status
- **FR-003**: System MUST support command `show ip bgp <prefix>` to filter BGP table by specific prefix
- **FR-004**: System MUST support command `show ip route` to display full routing table with protocol indicators (B=BGP, O=OSPF, S=static, C=connected)
- **FR-005**: System MUST support command `show ip route <protocol>` to filter by protocol type (bgp, ospf, static, connected)
- **FR-006**: System MUST support command `show interfaces` to list all interfaces with name, IP, status, MTU
- **FR-007**: System MUST support command `show interfaces <name>` to display detailed interface information
- **FR-008**: System MUST support command `show ip bgp summary` to display BGP neighbor table with neighbor IP, remote AS, session state, route counts
- **FR-009**: System MUST support command `show ip bgp neighbors <ip>` to display detailed neighbor information
- **FR-010**: System MUST respond to help command `?` by listing available commands with descriptions
- **FR-011**: System MUST support context-sensitive help `<partial-command> ?` to show valid completions
- **FR-012**: System MUST support Tab completion for commands and arguments
- **FR-013**: System MUST display command output in <100ms for networks with ≤100 routers (95th percentile)
- **FR-014**: System MUST support colored output: green for "up/established", red for "down/failed", yellow for "transitional states"
- **FR-015**: System MUST preserve CLI history per session (up arrow recalls previous commands)
- **FR-016**: System MUST handle Ctrl+C to cancel command without terminating session
- **FR-017**: System MUST handle `exit` or `quit` command to close session gracefully
- **FR-018**: System MUST sanitize command input to prevent shell injection
- **FR-019**: System MUST display helpful error messages for invalid commands suggesting corrections when possible
- **FR-020**: System MUST snapshot router state at start of command to prevent mid-display updates

### Key Entities

- **CLISession**: Represents an active terminal connection to a specific router; tracks command history, current mode (show vs config), and router reference
- **Command**: Abstract representation of a CLI command; includes verb (show), noun (ip), modifiers (bgp, route), and filters (prefix, protocol)
- **OutputFormatter**: Formats routing/interface/neighbor data into human-readable tables or JSON; handles column alignment and color codes
- **RouterSnapshot**: Immutable point-in-time copy of router state (routing table, BGP table, interfaces) captured when command starts

## Success Criteria

1. **Usability**: A student with no prior CLI experience can type `?`, discover `show ip bgp`, execute it, and understand output within 5 minutes
2. **Performance**: 95% of show commands complete in <100ms on 100-router topology with 10k routes
3. **Correctness**: Output of `show ip bgp` exactly matches current BGP RIB state as verified by contract tests comparing CLI output to internal API state
4. **Concurrency**: 10 students can connect to 10 different routers simultaneously without session interference or state corruption
5. **Stability**: Invalid commands never crash the emulator; all errors display helpful messages and return to prompt
6. **Educational Value**: Students report they feel like they're using "real router equipment" (measured via survey: ≥80% agreement)

## Assumptions

- Initial implementation supports vendor-neutral syntax only; vendor modes (Cisco, Juniper) deferred to Phase 4
- CLI sessions are local to the machine running the emulator (no remote SSH/telnet in Phase 1)
- Output is text-based; graphical topology visualization deferred to Phase 8 (Web UI)
- Command set limited to show commands; configuration mode (`configure terminal`) deferred to Phase 2
- Performance target of <100ms assumes modern hardware (4-core CPU, 8GB RAM); slower systems may see degradation
- Colored output assumes terminal supports ANSI escape codes; fallback to plain text if not supported
- Router names are unique identifiers assigned during topology load; collision handling not required in Phase 1

## Dependencies

- Upstream `bgpsim` crate provides `Network`, `ForwardingState`, and BGP RIB APIs
- `Network::get_device()` method returns router reference by name
- BGP RIB can be queried for prefix, next-hop, AS path, communities, local pref
- OSPF and static routes accessible via `ForwardingState` API
- Interface status available via topology inspection

## Out of Scope

- Configuration commands (deferred to Phase 2: Configuration Mode)
- Vendor-specific syntax (deferred to Phase 4: Vendor Syntax Modes)
- Remote access via SSH/telnet (deferred to Phase 8: Web UI / Multi-User)
- Debug commands (`debug bgp updates`) - not needed for basic learning
- Scripting or command batching - students type interactively
- Output export to file - students can use terminal's copy/paste
- Real-time event streaming (`monitor bgp`) - deferred to Phase 5: Real-Time Mode

## Open Questions

None. All ambiguities resolved with informed defaults documented in Assumptions.

---

**Constitution Compliance Check**

- ✅ **Test-First (Principle I)**: Contract tests will verify CLI output matches internal API state; parser tests precede implementation
- ✅ **Library-First (Principle II)**: CLI implementation will be in new `bgpsim-cli` crate, independently testable
- ✅ **CLI-First (Principle III)**: Feature IS the CLI; stdin/stdout protocol enforced
- ✅ **Integration Testing (Principle IV)**: Contract tests between CLI and `bgpsim` Network API
- ✅ **Observability (Principle V)**: CLI commands logged with trace IDs; deterministic state snapshots
- ✅ **Educational UX (Principle XI)**: Interactive CLI is PRIMARY goal; help system supports beginners
- ✅ **Performance Targets (Principle X)**: <100ms response time explicitly required and testable
- ✅ **Crash Recovery (Additional Constraints)**: Invalid commands return errors, never crash emulator
- ✅ **Accessibility (Additional Constraints)**: Colored output with fallback to plain text

---

**Next Steps**

1. Review and validate this specification with maintainers/educators
2. Run `/speckit.clarify` if any requirements need user clarification (currently none)
3. Run `/speckit.plan` to generate technical implementation plan
4. Create test suite in `bgpsim-cli/tests/` before writing implementation
5. Implement TDD cycle: write failing test → implement CLI parser/formatter → pass test
