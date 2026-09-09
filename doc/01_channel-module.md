# Synkie / VMix — Channel Module

## 1. Purpose

The channel module converts one physical video input into one clean, standardized internal video signal.

The module is intentionally simple.

```text
PHYSICAL INPUT
      │
      ▼
INPUT INTERFACE
      │
      ▼
BUFFER / CONDITIONING
      │
      ▼
SIGNAL PROCESSING
      │
      ▼
STANDARDIZED INTERNAL OUTPUT
```

The channel PCB is independent of:

* number of buses
* number of outputs
* output matrix configuration
* global routing configuration

---

## 2. Fundamental Constraint

Each channel module has exactly:

```text
1 physical input
1 internal analog output
```

The channel PCB contains no routing MUX.

This is a deliberate architectural constraint.

---

## 3. Signal Path

Preferred generic structure:

```text
INPUT
  │
  ▼
Termination / Protection
  │
  ▼
Input Buffer
  │
  ▼
Level / Gain Conditioning
  │
  ▼
Channel Processing
  │
  ▼
Output Buffer
  │
  ▼
SINGLE INTERNAL OUTPUT
```

The exact processing blocks depend on the final channel functionality.

---

## 4. Channel Responsibilities

The channel module is responsible for:

* accepting the physical input
* providing correct input termination
* protecting the circuit where required
* buffering the input
* establishing the internal signal level
* performing channel-specific processing
* providing a low-impedance standardized output

The channel module is not responsible for:

* selecting which bus receives the signal
* mixing multiple sources
* selecting physical outputs
* output pan/balance
* global configuration

---

## 5. Channel-to-Bus Interface

The channel output connects to the centralized bus router.

```text
CH1 ─────────┐
CH2 ─────────┤
CH3 ─────────┼──► BUS ROUTER
CH4 ─────────┤
...          │
CHn ─────────┘
```

A channel does not need to know:

* how many bus lanes exist
* which bus has selected it
* how many outputs exist
* how its signal is eventually mixed

This is one of the main modularity benefits of the architecture.

---

## 6. Fan-Out

The same channel output may be selected by multiple bus lanes.

Example:

```text
             ┌──► BUS0
CH1 ─────────┼──► BUS1
             ├──► BUS2
             └──► BUS3
```

This fan-out is implemented by the bus-routing stage.

The channel output therefore remains a single physical signal.

---

## 7. Electrical Output Standard

All channel modules shall expose a common electrical interface.

The interface shall define:

* nominal signal amplitude
* DC operating point
* source impedance
* maximum output load
* bandwidth
* distortion
* noise
* connector/pinout
* power rails

The final values shall be established together with the bus-router electrical design.

---

## 8. Video Performance

The channel output shall preserve the required composite-video characteristics.

Design evaluation shall include:

* bandwidth
* amplitude accuracy
* DC offset
* differential gain
* differential phase
* noise
* distortion
* transient response
* source impedance

A nominal bandwidth target of approximately 20 MHz shall be treated as a system-level design target rather than as the only component-selection criterion.

---

## 9. Preferred Physical Implementation

**Preferred:**

```text
1 physical input
        │
        ▼
1 dedicated channel PCB
        │
        ▼
1 standardized output
```

This makes the channel PCB reusable across systems with different routing and output configurations.

---

## 10. Design Principle

The channel module should remain:

```text
INPUT → PROCESSING → SINGLE OUTPUT
```

and nothing more.

Complexity belongs downstream, where it can be shared by all channels.
