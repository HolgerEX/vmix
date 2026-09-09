# Simplified Synkie / Anymix

## Living Engineering Baseline

**Project status:** Architecture / redesign phase
**Prototype target:** 4 inputs × 4 outputs
**Reference:** `mirdej/vmix` / `mirdej/synkie`
**Primary objective:** Simplify and modernize the electronic design while preserving the essential Synkie/Anymix signal architecture.

---

## 1. Project Definition

This project is a hardware redesign/fork derived from the **Synkie / Anymix21** ecosystem.

The objective is **not** to reproduce the original PCB implementation. The original design is treated as the functional and architectural reference, while the new design is free to substantially change:

* PCB partitioning
* component selection
* analogue topology
* digital architecture
* connector architecture
* power distribution
* control electronics
* mechanical integration

The redesign should preserve the useful characteristics of the original system while reducing unnecessary complexity.

### Primary design goals

1. **PCB simplification**
2. **Replacement of obsolete or unsuitable components**
3. **Reduction of component count**
4. **Reduction of inter-PCB connections**
5. **Improved manufacturability**
6. **Improved component availability**
7. **Improved serviceability**
8. **Preservation of the DC-coupled Synkie signal philosophy**

---

# 2. Prototype Scope

The first prototype shall implement:

* **4 input channels**
* **4 independent outputs**
* **1 master module**
* Four channel sections
* One master section
* Four mix/output buses
* Genlock / video timing
* Digital channel control where required
* DC-coupled signal processing

Conceptually:

```text
                    ┌──────────────────────────┐
 INPUT 1 ──────────►│                          │──────────► OUTPUT 1
 INPUT 2 ──────────►│     4 × CHANNEL CORE     │──────────► OUTPUT 2
 INPUT 3 ──────────►│                          │──────────► OUTPUT 3
 INPUT 4 ──────────►│                          │──────────► OUTPUT 4
                    └────────────┬─────────────┘
                                 │
                          MIX / CONTROL BUSES
                                 │
                    ┌────────────▼─────────────┐
                    │          MASTER           │
                    │                           │
                    │  Bus summing              │
                    │  Output processing        │
                    │  Genlock / sync           │
                    │  Control                  │
                    │  Timing                    │
                    └───────────────────────────┘
```

The first prototype is deliberately smaller than the original 12-channel Anymix21 system.

The 4-channel implementation should be considered a **development platform**, not necessarily the final product architecture.

---

# 3. Reference Architecture

The original Anymix21 architecture provides the starting point.

The original system is a DC-coupled analogue video mixer capable of handling Synkie-style signals rather than being limited to conventional AC-coupled composite video.

Important characteristics to preserve:

* DC coupling
* low-frequency / DC signal handling
* video-frequency signal handling
* approximately 10 MHz useful analogue bandwidth
* high-impedance inputs
* low-impedance outputs
* bipolar internal signals where required
* PAL/video timing compatibility
* genlock
* synchronised digital control

The original architecture also uses vertical blanking as an important timing boundary for digital operations.

---

# 4. Signal Philosophy

The new design should retain the Synkie philosophy:

> A channel signal is not merely a video waveform. It is a general-purpose DC-coupled analogue signal that may contain CV, audio-rate signals, ramps, modulation, or video-frequency content.

Therefore the analogue design must not accidentally introduce assumptions such as:

* mandatory AC coupling
* fixed video black level
* narrow-band video-only operation
* unnecessary DC restoration
* excessive filtering
* inappropriate termination

Unless explicitly required, signal paths should remain DC coupled.

---

# 5. Nominal Signal Requirements

Initial design assumptions:

| Parameter                  | Target                     |
| -------------------------- | -------------------------- |
| Signal type                | DC-coupled analogue        |
| Nominal signal range       | approximately 0–1 V        |
| Internal bipolar operation | permitted                  |
| Useful bandwidth           | DC to approximately 10 MHz |
| Input impedance            | high                       |
| Output impedance           | low                        |
| Video compatibility        | PAL/Synkie                 |
| Synchronisation            | genlocked                  |
| Channel count              | 4                          |
| Output count               | 4                          |

These values are **engineering targets**, not yet final specifications.

They must be confirmed against the actual prototype requirements and measured during validation.

---

# 6. Functional Architecture

Each channel is conceptually divided into:

```text
INPUT
  │
  ├── Input protection
  │
  ├── Input conditioning / clamp where required
  │
  ▼
ANALOG CHANNEL CORE
  │
  ├── Signal processing
  ├── Optional filtering
  ├── Comparator / threshold functions
  ├── Inversion
  ├── EQ / shaping
  └── Scale / bias
  │
  ▼
MIX LEVEL CONTROL
  │
  ├── BUS 1
  ├── BUS 2
  ├── BUS 3
  └── BUS 4 / AUX
  │
  ▼
PREVIEW / MONITOR
```

The exact implementation of each block is subject to redesign.

---

# 7. Channel Redesign Strategy

The original channel functionality is distributed over multiple PCBs.

The redesign should initially attempt to consolidate the channel into a single logical PCB.

## Preferred architecture

```text
┌──────────────────────────────────────────────┐
│                  CHANNEL PCB                 │
│                                              │
│ INPUT                                         │
│   │                                          │
│   ▼                                          │
│ Input protection / conditioning              │
│   │                                          │
│   ▼                                          │
│ Signal processing                            │
│   │                                          │
│   ├──── EQ / filtering                       │
│   ├──── comparator                           │
│   ├──── inverter                             │
│   └──── scale / bias                         │
│                                              │
│   ▼                                          │
│ Four mix-level controls                      │
│   │                                          │
│   ├──────── BUS 1                            │
│   ├──────── BUS 2                            │
│   ├──────── BUS 3                            │
│   └──────── BUS 4                            │
│                                              │
│ Preview / monitor                            │
│                                              │
│ Local control electronics                    │
└──────────────────────────────────────────────┘
```

### PCB consolidation principle

The default assumption is:

> **One physical PCB should contain one complete channel unless there is a compelling electrical, mechanical, thermal, or manufacturing reason to split it.**

The original PCB boundaries should not be treated as requirements.

---

# 8. Master Module

The master PCB should contain the functions common to all channels.

Initial functional partition:

```text
                   GENLOCK INPUT
                         │
                         ▼
                 SYNC EXTRACTION
                         │
                         ▼
                      TIMING
                         │
            ┌────────────┼────────────┐
            │            │            │
            ▼            ▼            ▼
         CONTROL      CHANNEL      OUTPUT
                       SYNC
            │            │            │
            └────────────┼────────────┘
                         │
              ┌──────────▼──────────┐
              │      BUS MIXING     │
              │                     │
              │ BUS 1               │
              │ BUS 2               │
              │ BUS 3               │
              │ BUS 4               │
              └──────────┬──────────┘
                         │
              ┌──────────▼──────────┐
              │   OUTPUT PROCESSING │
              └──────────┬──────────┘
                         │
                  OUTPUT 1–4
```

The master is also the preferred location for:

* system timing
* genlock
* global control
* bus summing
* output drivers
* system-level power distribution

---

# 9. Mix Bus Architecture

The first prototype shall support four independent output paths.

Each channel therefore has four controllable contributions:

```text
             CHANNEL
                │
       ┌────────┼────────┐
       │        │        │
       ▼        ▼        ▼
     LEVEL     LEVEL    LEVEL
       │        │        │
       ▼        ▼        ▼
     BUS 1    BUS 2    BUS 3
                          ...
                     BUS 4
```

The original design uses multiple analogue VCAs per channel.

This is a major redesign target.

## Investigation required

Determine whether the four-channel contribution can be implemented with:

* modern analogue VCAs
* integrated multi-channel VCA devices
* analogue multipliers
* digitally controlled analogue attenuators
* switched-gain amplifier structures
* alternative summing topologies

The objective is to reduce:

* IC count
* passive component count
* control circuitry
* PCB area
* power consumption

without compromising signal integrity.

---

# 10. VCA Redesign

The original four-VCA-per-channel architecture should **not automatically be retained**.

The redesign should investigate:

### Option A — Four analogue VCAs

```text
Signal
 ├── VCA ── BUS 1
 ├── VCA ── BUS 2
 ├── VCA ── BUS 3
 └── VCA ── BUS 4
```

Advantages:

* conceptually straightforward
* continuous analogue control
* close to original architecture

Disadvantages:

* high component count
* control circuitry
* power consumption
* PCB area

### Option B — Integrated multi-channel VCA

Use a modern IC containing several independent gain-control channels.

Potential advantage:

* dramatically reduced component count
* matched channels
* simplified PCB routing

### Option C — Alternative summing architecture

Investigate whether the required mathematical behaviour can be achieved by changing the location of attenuation/gain control.

For example:

```text
          CHANNEL SIGNAL
                │
                ▼
         GLOBAL PROCESSING
                │
                ▼
          BUS DISTRIBUTION
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
      BUS1     BUS2     BUS3 ...
```

This must be evaluated carefully because changing the topology may affect isolation and simultaneous output behaviour.

---

# 11. Digital Control Architecture

The original system uses dedicated microcontrollers and DAC/control electronics.

The redesign should not assume that the original MCU architecture is optimal.

First establish the actual control requirements.

## Control inventory

| Function             | Control type   | Resolution | Update rate | Automation |
| -------------------- | -------------- | ---------: | ----------: | ---------- |
| Channel level        | TBD            |        TBD |         TBD | Yes        |
| Bus 1 level          | TBD            |        TBD |         TBD | Yes        |
| Bus 2 level          | TBD            |        TBD |         TBD | Yes        |
| Bus 3 level          | TBD            |        TBD |         TBD | Yes        |
| Bus 4 level          | TBD            |        TBD |         TBD | Yes        |
| Scale                | TBD            |        TBD |         TBD | Yes        |
| Bias                 | TBD            |        TBD |         TBD | Yes        |
| Comparator threshold | TBD            |        TBD |         TBD | Maybe      |
| EQ                   | Analogue / TBD |          — |           — | Maybe      |
| Invert               | Digital        |          — |        Slow | Maybe      |
| Filter mode          | Digital        |          — |        Slow | Maybe      |

This table should be completed before selecting replacement MCUs/DACs.

---

# 12. MCU Strategy

The MCU architecture should be selected based on actual requirements rather than historical compatibility.

Potential architectures include:

### Distributed

```text
CHANNEL 1 MCU ─┐
CHANNEL 2 MCU ─┤
CHANNEL 3 MCU ─┼── MASTER
CHANNEL 4 MCU ─┤
                │
                └── Control network
```

### Centralised

```text
             MASTER MCU
                 │
       ┌─────────┼─────────┐
       ▼         ▼         ▼
   CHANNEL 1  CHANNEL 2  CHANNEL 3/4
```

### Hybrid

A master MCU handles system timing and communication while small local devices handle channel-specific analogue control.

The preferred solution is the one that minimizes:

* IC count
* firmware complexity
* PCB routing
* control latency
* noise injection
* maintenance burden

---

# 13. Digital Noise

Digital circuitry must be physically and electrically controlled because the analogue path extends into the MHz range.

The original architecture's use of vertical blanking as a timing boundary should therefore be retained as a design principle.

Preferred behaviour:

```text
ACTIVE VIDEO / SIGNAL
────────────────────────────────
Analogue circuitry quiet
No unnecessary digital activity

VERTICAL BLANKING
────────────────────────────────
Control updates
DAC updates
Communication
Housekeeping
```

The redesign should investigate whether all control updates can be synchronised to the same timing window.

---

# 14. Component Modernisation

The BOM should be rebuilt rather than mechanically copied.

Every inherited component should be classified:

| Classification | Meaning                                         |
| -------------- | ----------------------------------------------- |
| KEEP           | Current component remains technically suitable  |
| REPLACE        | Obsolete/EOL or unsuitable                      |
| CONSOLIDATE    | Function can be combined with another component |
| REMOVE         | Function is no longer necessary                 |
| INVESTIGATE    | Requires electrical comparison                  |

---

# 15. Component Selection Rules

Preferred components should have:

* active manufacturer support
* reasonable long-term availability
* multiple distribution channels where practical
* appropriate voltage range
* appropriate common-mode range
* sufficient GBW
* sufficient slew rate
* suitable input/output characteristics
* suitable noise performance
* suitable distortion
* suitable package
* reasonable thermal characteristics

For analogue video-frequency paths, component selection must consider more than DC accuracy.

Important parameters include:

* gain-bandwidth product
* slew rate
* settling time
* input capacitance
* output drive
* distortion
* noise
* phase behaviour
* capacitive-load stability

---

# 16. Op-Amp Replacement

Historical generic op-amp references should not be accepted automatically.

Each op-amp location should be classified according to function:

* precision DC amplifier
* high-speed signal amplifier
* buffer
* summing amplifier
* comparator-like function
* integrator
* filter
* bias generator
* output driver

A single modern op-amp family may be capable of replacing several historical part types, but this must be verified against:

* supply voltage
* input common-mode range
* output swing
* bandwidth
* slew rate
* stability
* loading

---

# 17. Component Count Reduction

Component reduction is a first-class engineering objective.

Potential sources:

### Passive components

* shared bias references
* shared termination networks
* simplified protection
* integrated filtering
* elimination of unnecessary pull-ups
* elimination of duplicated decoupling where electrically valid

### Analogue ICs

* multi-channel op-amps
* integrated VCA/control devices
* integrated DACs
* integrated analogue switches
* modern comparator arrays

### Digital ICs

* MCU peripherals replacing discrete glue logic
* integrated DAC/PWM where performance permits
* bus aggregation
* fewer level translators

### Connectors

* direct channel-to-master bus
* fewer intermediate PCBs
* reduced power connectors
* reduced control wiring

---

# 18. PCB Architecture

Initial preferred arrangement:

```text
┌───────────────────────────────┐
│       CHANNEL PCB             │
│                               │
│  CH1 │ CH2 │ CH3 │ CH4        │
│                               │
│  analogue + local control     │
└───────────────┬───────────────┘
                │
             BUS / CTRL
                │
┌───────────────▼───────────────┐
│          MASTER PCB           │
│                               │
│ Genlock / Timing              │
│ Bus summing                   │
│ Output processing             │
│ System control                │
│ Power distribution            │
└───────────────────────────────┘
```

However, two variants should be evaluated:

### Variant 1 — Four identical channel PCBs

```text
CH1 ─┐
CH2 ─┤
CH3 ─┼── MASTER
CH4 ─┘
```

Advantages:

* modular
* reusable PCB
* easy replacement
* natural future scaling

### Variant 2 — One 4-channel PCB

```text
┌──────────────────────────────┐
│ CH1 │ CH2 │ CH3 │ CH4 │ ...  │
└──────────────┬───────────────┘
               │
             MASTER
```

Advantages:

* fewer connectors
* fewer boards
* simpler system wiring
* potentially lower cost

Disadvantages:

* less modular
* larger PCB
* channel replacement less convenient

The final choice should be driven by mechanical constraints and expected scaling.

---

# 19. Interconnect Strategy

The original design uses multiple inter-PCB connections.

The redesign should minimise these.

Preferred logical buses:

```text
POWER
 ├── +V
 ├── -V
 ├── +5 V / digital supply
 └── GND

ANALOG
 ├── BUS 1
 ├── BUS 2
 ├── BUS 3
 └── BUS 4

TIMING
 ├── VSYNC
 └── optional timing signals

CONTROL
 ├── serial bus
 ├── reset
 └── optional interrupt
```

Exact connector type and physical topology remain TBD.

---

# 20. Grounding

Ground architecture is critical.

The redesign should explicitly define:

* analogue ground
* digital ground
* power ground
* chassis/shield ground
* connector shield
* genlock/video shield

The goal is to prevent digital return currents from contaminating sensitive analogue paths.

A grounding strategy must be designed before final PCB routing.

---

# 21. Power Architecture

The original system has substantial per-channel power requirements.

The redesign should target a significant reduction where possible.

Power architecture should be documented as:

```text
POWER INPUT
    │
    ├── Analogue supply
    │
    ├── Digital supply
    │
    └── Reference / auxiliary supply
             │
             ▼
       LOCAL REGULATION
             │
       ┌─────┴─────┐
       ▼           ▼
   ANALOG      DIGITAL
```

Each subsystem should have an explicit power budget.

Required measurements:

* idle consumption
* maximum analogue activity
* maximum digital activity
* startup current
* per-channel current
* master current
* total prototype consumption

---

# 22. Genlock / Synchronisation

Genlock is a core system function.

The master should provide:

1. external sync input
2. sync extraction
3. stable internal timing
4. distribution to channel/control electronics
5. synchronised control updates

The exact sync IC should be reconsidered during component modernisation.

Candidate approaches may include:

* dedicated sync separator
* comparator-based extraction
* modern video sync device
* MCU-assisted timing

The analogue quality and timing jitter must be evaluated before replacing the historical implementation.

---

# 23. Output Architecture

Each output should have:

```text
BUS
 │
 ▼
SUMMING / LEVEL
 │
 ▼
OUTPUT PROCESSING
 │
 ├── clipping / limiting
 ├── bias / level
 └── output buffer
 │
 ▼
OUTPUT CONNECTOR
```

The output stage must be capable of driving the intended external Synkie/video load without compromising bandwidth or waveform integrity.

---

# 24. Protection

Protection should be modernised but kept minimal.

Potential requirements:

* input overvoltage
* output short-circuit tolerance
* ESD
* connector abuse
* supply fault tolerance

Protection must not introduce excessive:

* capacitance
* leakage
* distortion
* signal-dependent behaviour

For high-frequency signal paths, every protection component must be evaluated as part of the analogue circuit.

---

# 25. Mechanical Design Principles

The PCB redesign should consider the final mechanical architecture early.

Requirements to establish:

* module width
* module height
* connector positions
* knob spacing
* PCB orientation
* mounting holes
* front-panel controls
* service access
* cable routing
* shielding

PCB simplification should not create an impractical mechanical assembly.

---

# 26. Design Rules

The following rules apply unless explicitly overridden by a documented engineering decision.

### Rule 1 — Function before historical implementation

The original circuit is a reference, not a constraint.

### Rule 2 — Prefer fewer parts

A simpler circuit is preferred when electrical performance is equivalent.

### Rule 3 — Do not sacrifice analogue performance for part count

Component reduction must not compromise:

* bandwidth
* DC accuracy
* noise
* distortion
* signal integrity
* stability

### Rule 4 — Avoid obsolete components

No new design dependency should be introduced on EOL components unless there is a compelling reason.

### Rule 5 — Prefer common components

Parts with broad availability should be preferred over unusual or single-source devices.

### Rule 6 — Minimise interconnects

Every connector and cable is considered a potential:

* failure point
* EMI path
* assembly cost
* impedance discontinuity

### Rule 7 — Keep analogue and digital domains disciplined

Digital convenience must not compromise the analogue signal path.

### Rule 8 — Measure rather than assume

Critical substitutions must be experimentally validated.

---

# 27. Validation Plan

The first prototype should be validated in stages.

## Stage 1 — Power

Verify:

* supply voltages
* startup behaviour
* current consumption
* regulator temperature
* ripple
* digital/analogue coupling

## Stage 2 — Basic analogue path

Verify:

* DC transfer
* gain
* offset
* input impedance
* output impedance
* bandwidth

## Stage 3 — Channel functions

Verify:

* channel level
* bus levels
* inversion
* filtering
* EQ
* comparator behaviour
* scale/bias

## Stage 4 — Four-channel operation

Test:

* one channel active
* four channels active
* all buses active
* worst-case simultaneous signals

## Stage 5 — Video

Test:

* PAL signal
* sync
* genlock
* colour/burst if applicable
* signal integrity
* output timing

## Stage 6 — Digital interaction

Test:

* control updates during blanking
* digital noise
* DAC settling
* control latency
* communication reliability

## Stage 7 — Stress

Test:

* overvoltage
* output loading
* rapid control changes
* maximum signal amplitude
* thermal conditions

---

# 28. Measurements to Establish

The following measurements should become part of the engineering record.

| Measurement           |   Target | Actual | Status |
| --------------------- | -------: | -----: | ------ |
| Input impedance       |      TBD |      — | TBD    |
| Output impedance      |      TBD |      — | TBD    |
| DC gain               |      TBD |      — | TBD    |
| DC offset             |      TBD |      — | TBD    |
| −3 dB bandwidth       | ≥ target |      — | TBD    |
| Slew rate requirement |      TBD |      — | TBD    |
| Noise                 |      TBD |      — | TBD    |
| THD                   |      TBD |      — | TBD    |
| Crosstalk             |      TBD |      — | TBD    |
| Bus isolation         |      TBD |      — | TBD    |
| DAC resolution        |      TBD |      — | TBD    |
| Control latency       |      TBD |      — | TBD    |
| Genlock jitter        |      TBD |      — | TBD    |
| Power/channel         |      TBD |      — | TBD    |
| Total power           |      TBD |      — | TBD    |

---

# 29. Component Replacement Workflow

For every inherited component:

```text
OLD COMPONENT
      │
      ▼
Determine function
      │
      ▼
Determine electrical requirements
      │
      ▼
Search modern alternatives
      │
      ▼
Compare:
  • bandwidth
  • voltage
  • noise
  • distortion
  • package
  • availability
      │
      ▼
Prototype / simulate
      │
      ▼
Measure
      │
      ▼
APPROVE / REJECT
```

A replacement should not be selected merely because it has a newer datasheet or lower nominal cost.

---

# 30. Engineering Decision Log

Decisions should be recorded in this format.

| ID      | Decision                     | Reason                            | Alternatives              | Status   |
| ------- | ---------------------------- | --------------------------------- | ------------------------- | -------- |
| DEC-001 | 4-channel first prototype    | Reduce development scope          | 12-channel                | Accepted |
| DEC-002 | Preserve DC coupling         | Core Synkie architecture          | AC coupling               | Accepted |
| DEC-003 | Redesign PCB partitioning    | Reduce complexity                 | Preserve original boards  | Accepted |
| DEC-004 | Re-evaluate VCA architecture | Major component-count opportunity | Original four VCA/channel | Open     |
| DEC-005 | Re-evaluate MCU architecture | Modernisation opportunity         | Original MCU topology     | Open     |
| DEC-006 | Rebuild BOM                  | Avoid obsolete dependencies       | Copy original BOM         | Accepted |

---

# 31. Open Engineering Questions

The following questions must be resolved during the redesign.

## Analogue

* What is the exact required signal amplitude?
* What is the maximum safe input voltage?
* What is the exact required bandwidth?
* What distortion level is acceptable?
* What noise floor is acceptable?
* Are all original EQ/filter functions required?
* Can several analogue stages be combined?
* Can the original VCA architecture be substantially simplified?

## Digital

* Is one MCU sufficient for the entire 4-channel system?
* Are local channel MCUs actually necessary?
* What DAC resolution is required?
* Can integrated MCU DACs be used?
* Is I²C still the best control bus?
* What data rate is required?
* Which operations must occur during vertical blanking?

## PCB

* Four channel PCBs or one 4-channel PCB?
* Where should the master/channel boundary be?
* How many connectors are actually required?
* Can the analogue buses run directly between channel and master?
* How should analogue and digital grounds be partitioned?

## Power

* What supply rails are actually required?
* Can the number of rails be reduced?
* Can several historical regulators be removed?
* What is the real current requirement per channel?

## Mechanical

* What is the intended module format?
* Should channel PCBs be replaceable?
* What connector system is preferred?
* How should front-panel controls connect to the PCB?

---

# 32. Prototype Milestones

## M0 — Architecture

* [x] Define 4×4 prototype
* [x] Establish simplification goals
* [x] Establish DC-coupled signal requirement
* [x] Establish master + channel architecture
* [ ] Freeze functional specification

## M1 — Circuit analysis

* [ ] Import current fork
* [ ] Analyse existing schematics
* [ ] Generate component inventory
* [ ] Identify obsolete components
* [ ] Identify duplicated circuitry
* [ ] Identify unnecessary circuitry
* [ ] Identify consolidation opportunities

## M2 — Analogue redesign

* [ ] Define channel signal path
* [ ] Select op-amps
* [ ] Redesign VCA architecture
* [ ] Redesign filters/EQ
* [ ] Define bus topology
* [ ] Define output stages
* [ ] Simulate critical analogue sections

## M3 — Digital redesign

* [ ] Define control architecture
* [ ] Select MCU
* [ ] Select DAC/control devices
* [ ] Define communications
* [ ] Define timing
* [ ] Define firmware architecture

## M4 — PCB

* [ ] Define PCB partition
* [ ] Define connectors
* [ ] Define power architecture
* [ ] Define grounding
* [ ] Complete schematic
* [ ] PCB layout
* [ ] Design review
* [ ] Manufacturing output

## M5 — Prototype

* [ ] PCB manufacture
* [ ] Assembly
* [ ] Power-up
* [ ] Analogue bring-up
* [ ] Digital bring-up
* [ ] Channel validation
* [ ] Master validation
* [ ] 4×4 integration

## M6 — Revision

* [ ] Record failures
* [ ] Record performance
* [ ] Identify unnecessary components
* [ ] Identify PCB improvements
* [ ] Revise BOM
* [ ] Revise schematic
* [ ] Revise PCB

---

# 33. Success Criteria

The first prototype is successful if it demonstrates the required 4×4 functionality while materially simplifying the original design.

Success should be evaluated quantitatively.

The redesign should aim for reductions in:

* PCB count
* component count
* connector count
* cable count
* power consumption
* assembly complexity
* obsolete component dependencies

while maintaining or improving:

* signal bandwidth
* DC performance
* noise
* distortion
* crosstalk
* stability
* synchronisation
* reliability

---

# 34. Current Project Position

The project is currently at:

**Architecture / component-analysis stage.**

The functional target is sufficiently defined to begin detailed circuit analysis, but the following should **not yet be considered frozen**:

* exact op-amps
* VCA topology
* DAC architecture
* MCU
* sync IC
* PCB partition
* connector system
* power rails
* exact component values

These should be determined from the actual fork's current schematics and BOM.

---

# 35. Next Engineering Action

The next step is to analyse the **actual fork**, rather than continuing to infer the design from the upstream repositories.

Required inputs:

1. Current schematic files
2. Current PCB files
3. Current BOM
4. Current project/repository tree
5. Any existing modifications relative to upstream

The analysis should then produce:

```text
CURRENT FORK
     │
     ├── Schematic inventory
     ├── PCB inventory
     ├── BOM inventory
     ├── Component status
     ├── Signal-path analysis
     ├── Power analysis
     ├── Control architecture
     │
     ▼
REDESIGN OPPORTUNITIES
     │
     ├── Remove
     ├── Combine
     ├── Replace
     ├── Re-route
     └── Re-architect
     │
     ▼
4×4 PROTOTYPE DESIGN
     │
     ├── Channel
     └── Master
```

This document should be maintained as the **single engineering baseline** for the redesign. New decisions should be added to the decision log rather than silently changing the architectural assumptions.

---

## Revision History

| Revision | Date       | Description                         |
| -------- | ---------- | ----------------------------------- |
| 0.1      | 2026-09-09 | Initial living engineering baseline |
|          |            |                                     |
|          |            |                                     |
|          |            |                                     |

---

## Working Principle

> **Preserve the behaviour and signal philosophy. Redesign the implementation.**

The objective is a simpler, more modern, lower-component-count Synkie/Anymix platform—not a cosmetically updated copy of the original hardware.
