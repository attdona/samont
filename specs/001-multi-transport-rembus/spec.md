# Feature Specification: Multi-Transport Rembus Firmware

**Feature Branch**: `001-multi-transport-rembus`

**Created**: 2026-10-29

**Status**: Draft

**Input**: User description: "The samont firmware is composed of a set of components that manages a different transport: WiFi, BLE, LoRaWAN and serial connection. Each component communicate using the Rembus specification. The set of functions related to pub/sub topics and rpc methods are a library of functions to be shared between all the wireless and wired protocols. samont provide a REPL feature for interacting from serial terminal."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Talk to a samont device over the serial connection (Priority: P1)

A technician connects a terminal to the samont serial port and, using the REPL, inspects the device and exercises its Rembus functions (publish to a topic, subscribe to a topic, call an RPC method exposed by the device) without needing any wireless link.

**Why this priority**: the serial connection is the simplest, always-available transport. It is the bench and field tool for commissioning and debugging, and it is the first place the shared Rembus function library can be proven end to end. It delivers a usable MVP on its own.

**Independent Test**: connect a serial terminal to a device, run REPL commands to call a device RPC method, publish a message and subscribe to a topic, and verify the printed results.

**Acceptance Scenarios**:

1. **Given** a powered device with a serial terminal attached, **When** the technician types a REPL command to call an exposed RPC method, **Then** the result (or an explicit error) is printed in the terminal.
2. **Given** the REPL is open, **When** the technician subscribes to a topic and a message is published on it, **Then** the message is shown in the terminal as it arrives.
3. **Given** the REPL is open, **When** the technician types an unknown or malformed command, **Then** the REPL prints an error and a usage hint and stays responsive.
4. **Given** the REPL is open, **When** the technician asks for help, **Then** the list of available commands is shown.

---

### User Story 2 - Reach the device over Wi-Fi and BLE with the same behavior (Priority: P2)

A remote Rembus peer (for example a gateway or an operator application) connects to the device over Wi-Fi or over BLE and uses the same publish/subscribe and RPC capabilities available on the serial connection, with identical semantics.

**Why this priority**: Wi-Fi and BLE are the main interactive wireless links. Identical behavior across them is the core value of sharing one function library.

**Independent Test**: from a remote peer, call the same RPC method and publish/subscribe to the same topic over each transport in turn, and verify the results are the same as those observed over the serial connection.

**Acceptance Scenarios**:

1. **Given** a device joined to a Wi-Fi network, **When** a remote peer calls an exposed RPC method, **Then** it receives the same result it would get over the serial connection.
2. **Given** a device advertising over BLE, **When** a remote peer connects and calls an exposed RPC method, **Then** it receives the same result.
3. **Given** a remote peer subscribed to a topic over any transport, **When** the device publishes on that topic, **Then** the peer receives the message.
4. **Given** a transport link is lost, **When** the link is later restored, **Then** the device accepts new Rembus sessions on it without a reboot.

---

### User Story 3 - Exchange messages over LoRaWAN (Priority: P3)

The device sends telemetry (publishes) and receives commands over a LoRaWAN link using the same Rembus functions, within the constraints of a low-bandwidth, low-duty-cycle radio.

**Why this priority**: LoRaWAN extends reach to places without Wi-Fi/BLE coverage but has the most restrictive capacity, so it builds on the shared library proven by the other transports.

**Independent Test**: with a LoRaWAN network available, publish a topic message from the device and send a downlink command, and verify both are delivered through the same functions used on the other transports.

**Acceptance Scenarios**:

1. **Given** a device joined to a LoRaWAN network, **When** it publishes a topic message, **Then** the message reaches the network application intact.
2. **Given** a device joined to a LoRaWAN network, **When** a command is received in downlink, **Then** the device processes it with the same behavior as on the other transports.
3. **Given** a message larger than the link allows, **When** the device tries to send it, **Then** the sender is told explicitly that it was not sent and why.

---

### User Story 4 - Add or change a transport without touching message semantics (Priority: P3)

A firmware developer adds a new topic or RPC method, or a new transport, by changing only one side: new topics/methods become available on every transport automatically, and a new transport reuses the existing function library.

**Why this priority**: it protects long-term maintainability and consistency, but delivers no direct end-user value until the transports exist.

**Independent Test**: register a new RPC method once and verify it is callable over every enabled transport; check that a transport component contains no per-method logic.

**Acceptance Scenarios**:

1. **Given** a new RPC method is registered once, **When** it is called over each enabled transport, **Then** it behaves identically on all of them.
2. **Given** one transport is disabled at build time, **When** the firmware runs, **Then** the remaining transports and the REPL keep working.

---

### Edge Cases

- A transport link drops in the middle of an RPC call: the caller gets a timeout/error and the device stays operational.
- Two transports receive requests at the same time: both are served without corrupting each other.
- A message is malformed, truncated or of an unsupported protocol version: it is rejected with an error and does not crash the device.
- A message exceeds the maximum size a transport can carry: it is refused with an explicit error instead of being silently truncated.
- Serial input contains partial lines, overlong lines or non-printable bytes: the REPL discards them and recovers.
- A topic has no subscriber, or an RPC method is unknown: the publisher gets no failure; the caller gets an explicit "not found" error.
- Radio resource contention (Wi-Fi, BLE and LoRaWAN active together): each transport degrades gracefully and none blocks the serial REPL.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The firmware MUST provide four transports: Wi-Fi, BLE, LoRaWAN and serial, each managed by its own independent component.
- **FR-002**: Every transport MUST carry Rembus messages and MUST NOT define a transport-specific application protocol; the message semantics MUST be identical across transports.
- **FR-003**: The publish/subscribe (topic) and RPC (method) functions MUST live in a single library shared by all transports; transport components MUST NOT contain topic- or method-specific logic.
- **FR-004**: A topic or RPC method registered once MUST be available on every enabled transport without further per-transport work.
- **FR-005**: The device MUST allow a remote peer to publish to topics, subscribe to topics and call RPC methods over each transport.
- **FR-006**: The device MUST be able to publish messages and expose RPC methods of its own, not only consume them.
- **FR-007**: Each transport MUST detect link loss and recover automatically when the link returns, without a device reboot.
- **FR-008**: A failure or disconnection in one transport MUST NOT stop the other transports or the REPL.
- **FR-009**: Messages that are malformed, truncated, oversized or of an unsupported protocol version MUST be rejected with an explicit error and MUST NOT affect device stability.
- **FR-010**: The protocol framing, error behavior and versioning used by each transport MUST be explicitly documented and testable.
- **FR-011**: A message that cannot be carried by a transport because of size or capacity limits MUST be reported to the sender as not sent; it MUST NOT be silently truncated or dropped.
- **FR-012**: The device MUST provide a REPL on the serial connection that lets a user list commands, call RPC methods, publish to a topic and subscribe to a topic.
- **FR-013**: The REPL MUST report an error and remain usable on unknown, malformed or incomplete commands.
- **FR-014**: The REPL MUST show subscribed-topic messages and command results asynchronously without corrupting the line being typed.
- **FR-015**: Each transport MUST be individually enableable or disableable at build time, with the rest of the firmware working unchanged.
- **FR-016**: The shared function library MUST be testable without hardware or radio, using a simulated transport.
- **FR-017**: Operation of each transport (connect, disconnect, rejected message, error) MUST be observable through device logs.

### Key Entities

- **Transport**: a communication channel (Wi-Fi, BLE, LoRaWAN, serial) with its own link state, capacity limits and framing, which carries Rembus messages.
- **Rembus message**: the common application-level unit exchanged on every transport (publish, subscribe, RPC request, RPC response, error).
- **Topic**: a named channel to which peers publish and subscribe.
- **RPC method**: a named operation exposed by the device that remote peers (or the REPL) can call and that returns a result or an error.
- **Session / peer**: a remote party connected through one transport and using Rembus with the device.
- **REPL command**: a text command typed on the serial terminal that maps to a Rembus function or a device inspection action.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of the RPC methods and topics defined for the device are available and give identical results on every enabled transport, verified by one shared test suite run against each transport.
- **SC-002**: A technician can open the serial terminal and complete a first RPC call from the REPL within 1 minute, using only on-device help.
- **SC-003**: After a link loss, every transport accepts a new session without a reboot in 100% of tested cases.
- **SC-004**: Disabling any single transport at build time leaves the other transports and the REPL passing their tests in 100% of tested cases.
- **SC-005**: 100% of malformed, truncated, oversized and wrong-version messages in the test set are rejected with an explicit error and the device keeps running (no reset, no hang) during a 24-hour soak with such traffic.
- **SC-006**: Adding a new RPC method requires changes in exactly one place and no changes in any transport component.
- **SC-007**: The REPL stays responsive (a command echoes within 200 ms) while Wi-Fi, BLE and LoRaWAN traffic is active.

## Assumptions

- Samont is an ESP32-S3 device; the RAM, flash and radio-coexistence budgets that bound this feature are to be fixed in the plan, and must be measurable per the project constitution.
- Rembus is the application protocol on all transports with CBOR as its wire encoding, as required by the project constitution; the exact Rembus version supported is set at planning time.
- Interactive Rembus sessions on Wi-Fi, BLE and serial are assumed; LoRaWAN is assumed to carry the same messages in a connectionless, low-rate fashion limited by the network's maximum payload size and duty cycle.
- Splitting a large Rembus message across several LoRaWAN frames is out of scope for this feature; oversized messages are refused (FR-011).
- Security and authentication of peers (pairing, credentials, encryption beyond what each transport natively provides) are not defined by this feature and need a separate specification before production use.
- Provisioning of transport credentials (Wi-Fi network, LoRaWAN keys) is outside this feature; the transports use credentials already present in the device configuration.
- The REPL is a local maintenance interface and relies on physical access to the serial port; no REPL login is assumed.
- The specific topics and RPC methods exposed by the device are defined by separate application specifications; this feature provides the mechanism, plus a minimal built-in set for testing (for example a ping/echo).
- Ethernet and other transports named in the constitution are future work and out of scope here.
