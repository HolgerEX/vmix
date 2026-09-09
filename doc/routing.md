# Synkie / VMix — Final Bus Architecture

## 1. Architecture Objective

The mixer shall use a modular, extensible architecture separating:

1. **Input/channel processing**
2. **Internal video distribution/routing**
3. **Output/mixing**
4. **Global digital control**

The architecture shall support `n` independent input channels, an extensible internal video bus, and `j` independent output/mixer modules.

The preferred architecture is:

    INPUT CHANNELS
         │
         ▼
    Channel Processing
         │
         ▼
    Channel MUX
         │
         ▼
    INTERNAL VIDEO BUS
         │
         ▼
    Output / Mixer Modules
         │
         ▼
    Physical Outputs

A separate master module controls the digital routing/control plane.

---

# 2. Top-Level Architecture

    ┌─────────────────────────────────────────────────────────────┐
    │                         CHANNEL MODULES                      │
    │                                                             │
    │  CH1 ──► processing ──► MUX ──┐                             │
    │  CH2 ──► processing ──► MUX ──┤                             │
    │  CH3 ──► processing ──► MUX ──┤                             │
    │  ...                          │                             │
    │  CHn ──► processing ──► MUX ──┘                             │
    └───────────────────────────────┬─────────────────────────────┘
                                    │
                                    ▼
    ┌─────────────────────────────────────────────────────────────┐
    │                    INTERNAL VIDEO BUS                       │
    │                                                             │
    │  BUS 0   BUS 1   ...   BUS m-1                              │
    │                                                             │
    │  BUS m   BUS m+1 ...   BUS 2m-1                             │
    │                                                             │
    │  ...                                                        │
    │                                                             │
    │  BUS (k-1)m ...              BUS km-1                        │
    │                                                             │
    │              k × m total lanes                              │
    └───────────────────────────────┬─────────────────────────────┘
                                    │
                 ┌──────────────────┼──────────────────┐
                 ▼                  ▼                  ▼
          ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
          │ MIX/OUTPUT  │   │ MIX/OUTPUT  │   │ MIX/OUTPUT  │
          │ MODULE 1    │   │ MODULE 2    │   │ MODULE j    │
          │             │   │             │   │             │
          │ full matrix │   │ full matrix │   │ full matrix │
          │ mixer       │   │ mixer       │   │ mixer       │
          └──────┬──────┘   └──────┬──────┘   └──────┬──────┘
                 │                 │                 │
                OUT1              OUT2              OUTj


    ┌─────────────────────────────────────────────────────────────┐
    │                       MASTER MODULE                          │
    │                                                             │
    │  configuration / routing / gain control / synchronization   │
    │                                                             │
    │  Controls channel MUXes and output mixer modules            │
    └─────────────────────────────────────────────────────────────┘

---

# 3. Channel Architecture

Each physical input channel is an independent module.

Preferred channel signal path:

    INPUT
      │
      ▼
    Input protection / termination
      │
      ▼
    Input buffer
      │
      ▼
    Level / gain conditioning
      │
      ▼
    Channel MUX
      │
      ▼
    Internal video bus

The channel module should not contain the output mixer.

This keeps the channel PCB simple and makes the channel reusable regardless of the eventual number of outputs.

## Preferred choice

**One channel PCB per physical input channel.**

The channel PCB should contain:

- input termination
- input protection
- video buffer
- required level conditioning
- MUX/routing stage
- local control interface
- internal-bus output driver

The channel PCB should expose its selected signal onto the internal video bus.

---

# 4. Channel MUX

The MUX is primarily a **bus assignment mechanism**, not the output mixer.

Its purpose is:

    Channel signal
         │
         ▼
       MUX
      / | \
     /  |  \
   BUS BUS BUS ...

A channel can therefore select which internal bus lane carries its signal.

## Preferred topology

Use an analog video MUX/crosspoint device where practical.

The MUX should be:

- wideband beyond the 20 MHz system target
- low distortion
- low crosstalk
- low insertion loss
- suitable for composite/PAL video levels
- digitally controllable
- capable of high-impedance/off-state isolation

The digital control should come from the master module.

## Important architectural rule

The channel MUX does **not** perform arbitrary mixing.

It assigns a channel to an internal bus lane.

This makes the channel routing problem separate from the output mixing problem.

---

# 5. Internal Video Bus

The internal video bus is the central distribution fabric.

The bus shall consist of:

    k × m lanes

where:

- `m` = base bus width
- `k` = number of bus groups/expansions
- `k × m` = total number of internal video lanes

The base bus width shall preferably be:

    m = 2 or 4

The architecture shall permit additional groups of `m` lanes to be added.

Example:

    k = 1, m = 4

    BUS0
    BUS1
    BUS2
    BUS3

Expansion:

    k = 2, m = 4

    BUS0
    BUS1
    BUS2
    BUS3

    BUS4
    BUS5
    BUS6
    BUS7

Further expansion:

    k = 3, m = 4

    BUS0 ... BUS11

---

# 6. Bus Width Recommendation

The preferred base configuration is:

    m = 4

A four-lane bus is preferable to a two-lane bus for the initial prototype because it provides useful routing flexibility without making the backplane excessively large.

The architecture shall nevertheless remain compatible with:

    m = 2

where a smaller implementation is desired.

## Preferred choice

**4 lanes per bus group.**

The PCB/backplane should be designed so that additional four-lane groups can be added without redesigning the channel module.

---

# 7. Bus Scaling

The internal bus should scale independently of the physical channel count.

The basic scaling model is:

    Total bus lanes = k × m

For example:

    4 channels:
        k = 1
        m = 4
        → 4 internal lanes

    8 channels:
        k = 2
        m = 4
        → 8 internal lanes

    12 channels:
        k = 3
        m = 4
        → 12 internal lanes

    16 channels:
        k = 4
        m = 4
        → 16 internal lanes

This provides a natural modular scaling mechanism.

---

# 8. Preferred Bus Capacity

Ideally:

    Number of internal video lanes ≥ number of input channels

Therefore, the preferred fully populated configuration is:

    k × m ≥ n

where:

- `n` = number of input channels
- `m` = base bus width
- `k` = number of bus groups

For the first prototype:

    n = 4
    m = 4
    k = 1

therefore:

    4 input channels
    4 internal video lanes

This gives a **1:1 channel-to-bus capacity**.

That is the preferred starting point.

---

# 9. Why Bus Capacity Should Match Channel Count

If:

    bus lanes = channel count

then every channel can have a unique internal destination.

For `n` channels:

    CH1 → BUS1
    CH2 → BUS2
    CH3 → BUS3
    ...
    CHn → BUSn

This allows the output mixer to perform the complete routing/mixing operation.

It also avoids a fundamental bottleneck where several channels must share one internal bus before the output mixer.

The internal bus therefore acts as a **lossless routing pool**, while the output modules perform the actual mixing.

---

# 10. Channel MUX Routing Model

The channel MUX should preferably be able to select:

    one internal bus lane

for the channel's signal.

For example:

    CH1 → BUS3
    CH2 → BUS1
    CH3 → BUS4
    CH4 → BUS2

The master module stores and controls these assignments.

The MUX therefore establishes the internal signal topology.

The output mixer then operates on the available bus signals.

---

# 11. Output / Mixer Module

Each output/mixer module is independent.

A module consumes the internal video bus and produces one or more physical outputs according to its design.

For a single-output mixer:

    BUS0 ──► gain ──┐
    BUS1 ──► gain ──┤
    BUS2 ──► gain ──┼──► SUM ──► output driver ──► OUT
    BUS3 ──► gain ──┘

For a four-lane bus:

    OUT = G0·BUS0
        + G1·BUS1
        + G2·BUS2
        + G3·BUS3

Each contribution has an independently controllable gain.

---

# 12. Full Matrix Mixing

The output stage shall support **full matrix mixing**.

For `B = k × m` internal bus lanes and `j` output modules:

    OUT1 = G11·BUS1 + G12·BUS2 + ... + G1B·BUSB

    OUT2 = G21·BUS1 + G22·BUS2 + ... + G2B·BUSB

    ...

    OUTj = Gj1·BUS1 + Gj2·BUS2 + ... + Gjb·BUSB

The complete gain matrix is therefore:

                     Internal Bus
                  B1    B2    B3   ...   BB
               ┌────────────────────────────
    OUT1       │ G11  G12   G13  ...  G1B
    OUT2       │ G21  G22   G23  ...  G2B
    OUT3       │ G31  G32   G33  ...  G3B
    ...        │ ...
    OUTj       │ Gj1  Gj2   Gj3  ...  GjB
               └────────────────────────────

Every output shall be able to receive every internal bus lane.

This is the definition of the full matrix mixer.

---

# 13. Output Gain Cell

The preferred output architecture is:

    BUS
      │
      ▼
    Variable gain element
      │
      ▼
    Summing node
      │
      ▼
    Output buffer/driver

The gain element simultaneously provides:

- routing enable
- contribution level
- mixing coefficient

Therefore a separate MUX at every matrix intersection is not necessarily required.

Conceptually:

    Gij = 0
        → bus not contributing

    Gij = nominal
        → unity contribution

    0 < Gij < nominal
        → attenuated contribution

This is preferred over:

    crosspoint → separate VGA → mixer

if the selected VCA/VGA can provide sufficient OFF attenuation and signal isolation.

This reduces component count and PCB complexity.

---

# 14. Preferred Matrix Implementation

### Preferred

    INTERNAL BUS
         │
         ├──► VCA ──┐
         ├──► VCA ──┤
         ├──► VCA ──┤
         └──► VCA ──┘
                    │
                    ▼
                  SUM
                    │
                    ▼
                 OUTPUT

Each output module contains one gain cell per internal bus lane.

For `B` bus lanes:

    B gain cells / output

For `j` output modules:

    j × B gain cells

This gives a predictable scaling relationship.

---

# 15. Alternative Matrix Implementation

If the chosen VCA cannot provide sufficient OFF isolation, use:

    INTERNAL BUS
         │
         ▼
    Analog crosspoint
         │
         ▼
    VGA/VCA
         │
         ▼
       SUM

This provides explicit routing isolation but increases:

- component count
- PCB area
- control complexity
- signal-path loading
- potential insertion loss

Therefore it is the **secondary choice**, not the preferred architecture.

---

# 16. Output Module Scaling

Output modules shall be independent.

For example:

    4 internal buses
    1 output module

gives:

    4 → 1 mixer

With two output modules:

    4 → 1
    4 → 1

giving:

    4 → 2 full matrix

With four output modules:

    4 → 4 full matrix

Each output is independently controllable.

The number of output modules therefore does not need to equal the number of input channels.

---

# 17. Example: 4 × 4 Prototype

The recommended first prototype is:

    n = 4 input channels
    m = 4 lanes/group
    k = 1 bus group
    B = 4 total bus lanes
    j = 4 output modules

Architecture:

    CH1 ──► MUX ──┐
    CH2 ──► MUX ──┤
    CH3 ──► MUX ──┼──► BUS0..BUS3
    CH4 ──► MUX ──┘
                       │
             ┌─────────┼─────────┐
             ▼         ▼         ▼
           OUT1      OUT2      OUT3 ... OUT4
             │         │         │
          4:1 SUM    4:1 SUM   4:1 SUM
             │         │         │
            OUT1      OUT2      OUT3      OUT4

Each output has four independently controlled gain paths.

---

# 18. Example Routing

Suppose:

    CH1 → BUS0
    CH2 → BUS1
    CH3 → BUS2
    CH4 → BUS3

Then:

    OUT1 = 1.0·BUS0 + 0.5·BUS1 + 0·BUS2 + 0·BUS3

    OUT2 = 0·BUS0 + 1.0·BUS1 + 1.0·BUS2 + 0·BUS3

    OUT3 = 0.5·BUS0 + 0·BUS1 + 0.5·BUS2 + 1.0·BUS3

    OUT4 = 1.0·BUS0 + 1.0·BUS1 + 1.0·BUS2 + 1.0·BUS3

This demonstrates why a bus width equal to the channel count is desirable.

Every physical input can remain independently available to every output.

---

# 19. Pan / Balance

Stereo operation can be implemented in the output matrix.

For example, two output modules:

    OUT_L
    OUT_R

A channel can be assigned to a bus lane and then mixed into both outputs with different coefficients.

For channel `CH1`:

    BUS0 = CH1

Then:

    OUT_L = 0.707 × BUS0
    OUT_R = 0.707 × BUS0

for a center-panned signal.

Moving the pan changes the two matrix coefficients.

This avoids requiring dedicated analog pan circuitry on every input channel.

Pan is therefore preferably implemented as a **matrix gain relationship**.

---

# 20. Master Module

The master module is responsible for the digital control plane.

It shall control:

- channel MUX selection
- output mixer gain
- matrix configuration
- output enable/mute
- global configuration
- module identification
- optional presets/scenes
- synchronization of configuration changes

The master module should not carry the main analog video path.

Preferred architecture:

    MASTER
      │
      ├──── digital control ────► CHANNEL MODULES
      │
      ├──── digital control ────► MIXER MODULES
      │
      └──── configuration / UI

This keeps the high-speed analog video path physically separate from the digital control system.

---

# 21. Digital Control Architecture

The internal digital control bus should be designed as a modular control network.

Preferred characteristics:

- addressable modules
- deterministic configuration
- low pin count
- easy PCB expansion
- hot-plugging not required
- simple firmware implementation
- local decoding on each module

A serial control bus is preferred over dedicating individual MCU GPIO lines to every analog switch.

Possible implementations include:

- SPI with local chip-select/address decoding
- I²C where device speed/latency is adequate
- dedicated serial control bus
- shift-register architecture for simple static controls

### Preferred initial choice

**SPI-style local control with module addressing/selection.**

The final choice should be made after the selected MUX/VCA ICs are known.

---

# 22. Physical Bus Architecture

The internal video bus should preferably be implemented as a controlled backplane.

Recommended topology:

    Channel PCBs
        │
        │
        ▼
    ┌──────────────────────────┐
    │       VIDEO BACKPLANE     │
    │                           │
    │ BUS0 BUS1 BUS2 BUS3 ...   │
    └──────────────────────────┘
        │       │       │
        ▼       ▼       ▼
     MIX 1    MIX 2    MIX j

The backplane should be treated as a real high-frequency analog transmission structure.

At the 20 MHz system target, PCB layout remains important because PAL composite video contains significantly higher-frequency components than the nominal 20 MHz fundamental bandwidth target might suggest.

---

# 23. Bus Electrical Requirements

Each internal video lane should have:

- controlled source impedance
- defined termination strategy
- low capacitive loading
- adequate drive capability
- controlled trace geometry
- short stubs
- predictable loading

The bus should preferably be designed around a single defined video impedance, normally:

    75 Ω

if the internal signal remains conventional composite video.

The exact topology must be validated experimentally because a multi-drop 75 Ω analog bus can become heavily loaded.

The preferred implementation may therefore use:

    Channel driver
        │
        ▼
    buffered bus
        │
        ├──► Mixer 1
        ├──► Mixer 2
        └──► Mixer j

rather than directly connecting multiple high-capacitance MUX inputs to the same passive node.

---

# 24. Bus Driver

Each channel should preferably have a dedicated bus driver.

This isolates:

    channel circuitry

from:

    backplane loading

and provides predictable drive characteristics.

Preferred:

    Channel signal
         │
         ▼
    MUX
         │
         ▼
    Video buffer / line driver
         │
         ▼
    Internal bus

The buffer should be selected for:

- ≥20 MHz useful video bandwidth
- adequate slew rate
- low distortion
- low noise
- appropriate output drive
- 75 Ω video operation
- stable operation with expected capacitive loading

---

# 25. Component Selection Philosophy

Component selection should follow this order:

### 1. Signal integrity

The device must comfortably support the intended video bandwidth.

### 2. Video performance

Evaluate:

- differential gain
- differential phase
- crosstalk
- insertion loss
- return loss
- group delay
- settling/glitch behavior

### 3. Component count

Prefer devices combining multiple required functions.

### 4. Control simplicity

Prefer devices with straightforward digital configuration.

### 5. Availability

Prefer current-production components with multiple-source or stable supply availability where practical.

### 6. Cost

Cost optimization comes after achieving a robust signal path.

---

# 26. Recommended Functional Partition

## Channel PCB

Contains:

    Input
      ↓
    Protection / termination
      ↓
    Input buffer
      ↓
    Level conditioning
      ↓
    MUX
      ↓
    Bus driver
      ↓
    Internal bus

One PCB represents one independent input channel.

---

## Backplane

Contains:

- internal video bus lanes
- digital control bus
- power distribution
- module connectors

The backplane should contain as little active analog circuitry as possible.

---

## Mixer / Output PCB

Contains:

    BUS lanes
       │
       ├──► VCA/gain cell ─┐
       ├──► VCA/gain cell ─┤
       ├──► VCA/gain cell ─┤
       └──► VCA/gain cell ─┤
                           ▼
                         SUM
                           │
                       output buffer
                           │
                         OUTPUT

One output module corresponds to one independently controlled matrix output.

---

## Master PCB

Contains:

- MCU
- user interface
- digital control
- configuration storage
- module addressing
- optional communication interface

No primary analog video processing should be performed here.

---

# 27. Design Rules

The architecture shall follow these rules:

### Rule 1 — Channels are independent

Adding a channel should not require redesigning existing channel PCBs.

### Rule 2 — Channels do not directly mix

All mixing occurs in output/mixer modules.

### Rule 3 — Internal bus is a routing resource

The bus carries independently selected channel signals.

### Rule 4 — Output modules perform matrix mixing

Every output can access every internal bus lane.

### Rule 5 — Bus capacity should scale with channel count

Prefer:

    B ≥ n

for a fully flexible configuration.

### Rule 6 — Base expansion unit is fixed

Use:

    m = 4

as the preferred physical expansion unit.

### Rule 7 — Output count is independent

`j` may be smaller or larger than `n`.

### Rule 8 — Digital control is centralized

The master module owns the configuration.

### Rule 9 — Analog and digital domains remain separated

The digital control system should not unnecessarily enter the high-speed analog signal path.

---

# 28. Final Mathematical Model

Let:

    X = vector of physical input channels

    X = [X1, X2, ..., Xn]

Let:

    B = vector of internal video bus lanes

    B = [B1, B2, ..., BB]

The channel routing stage generates:

    B = R X

where `R` represents the channel-to-bus assignment.

The output matrix then generates:

    Y = G B

where:

- `G` is the output mixing matrix
- `Y` is the vector of physical outputs

Therefore:

    Y = G R X

This separation is intentional.

`R` controls **routing**.

`G` controls **mixing**.

The architecture can therefore be extended without changing the fundamental signal model.

---

# 29. Final Recommended Architecture

The preferred final architecture is:

    n INPUT CHANNELS
            │
            ▼
    ┌──────────────────┐
    │ Channel processing│
    │ + MUX             │
    │ + bus driver      │
    └────────┬─────────┘
             │
             ▼
    ┌────────────────────────────────┐
    │        INTERNAL VIDEO BUS       │
    │                                │
    │       k × m lanes              │
    │                                │
    │       m = 4 preferred          │
    │       k scalable               │
    │                                │
    │       B = k × m ≥ n preferred  │
    └───────────────┬────────────────┘
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
       MIX/OUT   MIX/OUT    MIX/OUT
       MODULE 1  MODULE 2   MODULE j
          │         │         │
          ▼         ▼         ▼
        OUT 1     OUT 2     OUT j

                   ▲
                   │
             MASTER MODULE
          digital control plane


## First Prototype

    n = 4 channels
    m = 4 lanes
    k = 1 bus group
    B = 4 internal lanes
    j = 4 output modules

    4 channels
       ↓
    4 channel MUXes
       ↓
    4-lane internal video bus
       ↓
    4 independent full-matrix mixer outputs


## Preferred implementation choices

| Section | Preferred choice |
|---|---|
| Input module | One PCB per input channel |
| Input buffer | Wideband video buffer |
| Channel routing | Analog video MUX |
| Channel → bus | Dedicated bus driver |
| Base bus width | **4 lanes** |
| Bus expansion | Additional 4-lane groups |
| Bus capacity | **Prefer ≥ input channel count** |
| Backplane | Controlled-impedance analog video bus |
| Output routing | Full matrix |
| Matrix gain | **VCA/gain cell per bus → output** |
| Separate output MUX | Avoid if VCA provides sufficient OFF isolation |
| Mixer | Analog summing amplifier |
| Output driver | Dedicated video output buffer |
| Output modules | Independent, scalable |
| Control | Central master module |
| Digital bus | SPI-style modular control initially |
| Pan/balance | Matrix coefficients |
| First prototype | **4 × 4 full matrix** |

---

# 30. Architectural Summary

The fundamental design decision is:

**Do not build a conventional channel mixer where each channel feeds a fixed number of mix buses.**

Instead, build:

    CHANNEL
       ↓
    MUX
       ↓
    INTERNAL VIDEO BUS
       ↓
    FULL OUTPUT MATRIX
       ↓
    OUTPUTS

The internal bus provides a scalable pool of independent video signals.

The output modules provide the actual mixing.

For maximum flexibility:

    internal bus lanes ≥ input channels

and the preferred physical expansion unit is:

    4 video lanes.

The first prototype therefore becomes a clean:

    4 INPUT
       ×
    4 INTERNAL BUS
       ×
    4 OUTPUT

matrix system.

The architecture can subsequently scale to:

    8 × 8
    12 × 12
    16 × 16
    ...

without fundamentally changing the channel, bus, or output-module concepts.
