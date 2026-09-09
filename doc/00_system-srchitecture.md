# Synkie / VMix — System Architecture

## 1. Purpose

The mixer is designed as a modular analog-video processing system with a strict separation between:

1. Input/channel processing
2. Channel-to-bus routing
3. Internal video distribution
4. Output mixing
5. Global digital control

The architecture is intended to simplify the electronics, reduce component count, permit module reuse, and allow the number of inputs, internal buses, and outputs to scale independently.

The fundamental signal path is:

```text
PHYSICAL INPUT
      │
      ▼
CHANNEL MODULE
      │
      │ one standardized internal video output
      ▼
CENTRAL BUS ROUTER
      │
      ▼
INTERNAL VIDEO BUS
      │
      ▼
OUTPUT / MIXER MODULE
      │
      ▼
PHYSICAL OUTPUT
```

Digital control is a separate control plane:

```text
MASTER MODULE
      │
      ├──► Bus routing
      └──► Output mixer control
```

The Master module does not form part of the primary analog video path.

---

## 2. Core Architectural Principle

The system is divided into three independent signal layers.

### Channel layer

```text
n independent physical sources
          │
          ▼
INPUT PROCESSING
          │
          ▼
n standardized channel outputs
```

### Routing layer

```text
n channel outputs
          │
          ▼
centralized routing
          │
          ▼
B internal video buses
```

### Mixing layer

```text
B internal buses
          │
          ▼
output gain matrix
          │
          ▼
j independent outputs
```

The complete system can therefore be represented as:

```text
n CHANNELS → B BUSES → j OUTPUTS
```

---

## 3. Top-Level Architecture

```text
┌─────────────────────────────────────────────┐
│              CHANNEL MODULES                │
│                                             │
│ CH1 ──► processing ──► single output        │
│ CH2 ──► processing ──► single output        │
│ CH3 ──► processing ──► single output        │
│ ...                                         │
│ CHn ──► processing ──► single output        │
└──────────────────────┬──────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────┐
│             CENTRAL BUS ROUTER              │
│                                             │
│ MUX1 ──► BUS0                               │
│ MUX2 ──► BUS1                               │
│ MUX3 ──► BUS2                               │
│ ...                                         │
│ MUXB ──► BUSB                               │
└──────────────────────┬──────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────┐
│              INTERNAL VIDEO BUS             │
│                                             │
│ B = k × m lanes                             │
│ m = 4 preferred expansion unit              │
└──────────────────────┬──────────────────────┘
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
     OUTPUT 1     OUTPUT 2       OUTPUT j
     MATRIX       MATRIX         MATRIX
          │            │            │
          ▼            ▼            ▼
        OUT1         OUT2         OUTj
```

---

## 4. Channel Interface

Each physical channel has exactly one analog internal output.

```text
INPUT
  │
  ▼
CHANNEL PROCESSING
  │
  ▼
SINGLE INTERNAL OUTPUT
```

The channel module contains:

* input termination/protection
* input buffer
* level/gain conditioning
* required signal processing
* output buffer

The channel module does **not** contain:

* channel routing MUX
* crosspoint
* output matrix
* output mixer

This keeps the channel PCB independent of the number of internal buses and outputs.

---

## 5. Centralized Routing

Routing is performed after the channel modules.

For every internal bus lane:

```text
BUSi = MUX(CH1 ... CHn)
```

Thus each bus lane independently selects one channel.

A channel may be selected by multiple bus lanes:

```text
CH1 → BUS0
CH1 → BUS1
CH1 → BUS2
```

This provides fan-out at the routing stage without adding routing circuitry to the channel module.

The routing layer therefore separates:

```text
SOURCE GENERATION
```

from:

```text
SOURCE ASSIGNMENT
```

---

## 6. Internal Bus

The internal video bus is the distribution fabric between the routing and output stages.

The bus width is:

```text
B = k × m
```

where:

* `B` = total number of internal video lanes
* `m` = lanes per expansion group
* `k` = number of expansion groups

The preferred expansion unit is:

```text
m = 4
```

Examples:

```text
k = 1 → B = 4
k = 2 → B = 8
k = 3 → B = 12
```

---

## 7. Bus Capacity

For maximum routing flexibility, the preferred design target is:

```text
B ≥ n
```

This permits all physical channel signals to be simultaneously represented on independent internal buses.

However, this is **not a fundamental architectural requirement**.

A system may deliberately use:

```text
B < n
```

when reduced routing capacity is acceptable.

The bus count is therefore an architectural resource rather than a fixed function of input count.

---

## 8. Output Matrix

Each output is an independent weighted sum of the internal buses.

For output `j`:

```text
OUTj = Σ(Gji × BUSi)
```

where:

* `Gji` = gain applied to bus `i` for output `j`
* `BUSi` = internal video bus `i`

The complete system is:

```text
B internal buses
        │
        ▼
B × j gain matrix
        │
        ▼
j output sums
```

Every output can therefore independently mix every available bus.

---

## 9. Routing and Mixing Separation

Routing:

```text
CHANNEL → BUS
```

Mixing:

```text
BUS → OUTPUT
```

The complete analog architecture is:

```text
CHANNEL
   │
   ▼
PROCESSING
   │
   ▼
CENTRAL ROUTER
   │
   ▼
INTERNAL BUS
   │
   ▼
VARIABLE GAIN
   │
   ▼
SUM
   │
   ▼
OUTPUT DRIVER
```

This separation is fundamental to the design.

---

## 10. Scaling Model

The architecture is described by three independent parameters:

```text
n = number of physical input channels
B = number of internal video buses
j = number of output modules
```

The signal architecture is:

```text
n → B → j
```

The routing complexity is primarily determined by:

```text
n × B
```

The output mixing complexity is:

```text
B × j
```

The output matrix therefore scales with the internal bus width rather than directly with the physical input count.

---

## 11. Mathematical Model

Let:

```text
X = [X1, X2, ..., Xn]
```

represent the physical channel signals.

The router generates:

```text
B = R X
```

where `R` is the channel-to-bus routing matrix.

Each bus row contains one selected source.

The output matrix generates:

```text
Y = G B
```

where `G` is the output gain matrix.

Therefore:

```text
Y = G R X
```

The architecture consequently separates:

```text
R → source routing
G → signal mixing
```

---

## 12. First Prototype

The first prototype shall use:

```text
n = 4 input channels
B = 4 internal buses
j = 4 outputs
```

Therefore:

```text
4 channels
   ↓
4 independently routed buses
   ↓
4 independent full-matrix outputs
```

The output matrix contains:

```text
4 × 4 = 16
```

independent gain paths.

---

## 13. Preferred Module Partition

| Function                  | Module                 |
| ------------------------- | ---------------------- |
| Physical input processing | Channel module         |
| Channel-to-bus routing    | Bus router             |
| Signal distribution       | Internal bus/backplane |
| Bus-to-output gain        | Mixer/output module    |
| Output summing/drive      | Mixer/output module    |
| Digital configuration     | Master module          |
| User interface            | Master module          |

---

## 14. Preferred Choices

| Section                          | Preferred choice                                   |
| -------------------------------- | -------------------------------------------------- |
| Channel architecture             | One PCB per physical input                         |
| Channel outputs                  | Exactly one internal output                        |
| Channel routing                  | No MUX on channel PCB                              |
| Routing                          | Centralized bus router                             |
| Bus selector                     | One independently controlled selector per bus lane |
| Base bus width                   | 4 lanes                                            |
| Expansion unit                   | 4 lanes                                            |
| Output architecture              | Independent mixer/output modules                   |
| Output mixing                    | Full matrix                                        |
| Gain architecture                | One variable-gain path per bus/output intersection |
| Master                           | Central digital controller                         |
| Analog/video path through Master | None                                               |
| Initial system                   | 4 channels / 4 buses / 4 outputs                   |
