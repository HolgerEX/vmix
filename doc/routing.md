# Synkie / VMix — Final Bus Architecture

## 1. Architecture Objective

The mixer shall use a modular architecture separating:

1. **Input/channel processing**
2. **Internal video bus routing**
3. **Output/mixing**
4. **Global digital control**

The channel module has exactly **one internal video output**.

Routing from channels onto the internal video bus is performed by a centralized MUX/routing stage. The channel PCB contains **no routing MUX**.

The resulting architecture is:

```
INPUT CHANNELS
     │
     ▼
Channel Processing
     │
     ▼
Single Channel Output
     │
     ▼
INTERNAL BUS MUX / ROUTER
     │
     ▼
INTERNAL VIDEO BUS
     │
     ▼
Output / Mixer Modules
     │
     ▼
Physical Outputs
```

A separate master module controls the digital routing and configuration.

---

# 2. Top-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                       CHANNEL MODULES                        │
│                                                             │
│  CH1 ──► processing ───────────────┐                        │
│  CH2 ──► processing ───────────────┤                        │
│  CH3 ──► processing ───────────────┤                        │
│  ...                               │                        │
│  CHn ──► processing ───────────────┘                        │
│                                                             │
│  Each channel has ONE internal video output                 │
└───────────────────────────────┬─────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────┐
│                 INTERNAL BUS MUX / ROUTER                   │
│                                                             │
│  CH1 ──► MUX ──► BUS                                         │
│  CH2 ──► MUX ──► BUS                                         │
│  CH3 ──► MUX ──► BUS                                         │
│  ...                                                        │
│  CHn ──► MUX ──► BUS                                         │
│                                                             │
│  Routing is centralized at the bus level.                   │
└───────────────────────────────┬─────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────┐
│                    INTERNAL VIDEO BUS                        │
│                                                             │
│  BUS 0   BUS 1   ...   BUS m-1                              │
│                                                             │
│  BUS m   BUS m+1 ...   BUS 2m-1                             │
│                                                             │
│  ...                                                        │
│                                                             │
│  k × m total lanes                                           │
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
│  Controls bus MUXes and output mixer modules                │
└─────────────────────────────────────────────────────────────┘
```

---

# 3. Channel Module Architecture

Each physical input channel is an independent module.

The channel PCB has exactly **one internal video output**.

Preferred signal path:

```
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
SINGLE INTERNAL OUTPUT
  │
  ▼
Internal bus MUX
```

The channel module does **not** perform routing.

There is:

* no channel MUX
* no channel crosspoint
* no channel mixing
* no output selection

The channel module's sole responsibility is to turn its physical input into a clean, standardized internal video signal.

## Preferred choice

**One PCB per physical input channel with one standardized internal video output.**

This provides a very simple and reusable channel design.

---

# 4. Channel Output Standard

All channel modules should expose the same internal electrical interface.

The channel output should provide:

* standardized signal level
* standardized impedance
* defined DC operating point
* adequate bandwidth
* low distortion
* predictable source impedance

The channel output then connects to the centralized internal-bus routing stage.

Conceptually:

```
CH1 ─────────┐
CH2 ─────────┤
CH3 ─────────┼──► INTERNAL BUS MUX
CH4 ─────────┤
...          │
CHn ─────────┘
```

This makes the channel module independent from the number of internal bus lanes.

---

# 5. Internal Bus MUX / Router

The routing MUX is located centrally between the channel modules and the internal video bus.

Its purpose is:

```
CHANNEL OUTPUT
      │
      ▼
  BUS MUX
      │
      ▼
INTERNAL BUS LANE
```

For `n` channels and `B` internal bus lanes, the routing stage maps channel outputs onto the available bus lanes.

The conceptual routing matrix is:

```
             BUS
          B1 B2 B3 ... BB
       ┌───────────────────
CH1    │  x  x  x  ...  x
CH2    │  x  x  x  ...  x
CH3    │  x  x  x  ...  x
...    │
CHn    │  x  x  x  ...  x
       └───────────────────
```

The exact implementation does not necessarily need a full crosspoint.

The preferred initial implementation is a **MUX per internal bus lane**:

```
BUS1 = MUX(CH1 ... CHn)
BUS2 = MUX(CH1 ... CHn)
BUS3 = MUX(CH1 ... CHn)
...
BUSB = MUX(CH1 ... CHn)
```

This allows every bus lane to independently select a channel.

---

# 6. Bus MUX Architecture

For `B` internal lanes:

```
CH1 ─┬────────► MUX1 ──► BUS1
CH2 ─┤
CH3 ─┤
...  ┤
CHn ─┘

CH1 ─┬────────► MUX2 ──► BUS2
CH2 ─┤
...  ┤
CHn ─┘

...

CH1 ─┬────────► MUXB ──► BUSB
CH2 ─┤
...  ┤
CHn ─┘
```

This creates `B` independently selectable bus lanes.

Each bus lane can select one channel.

---

# 7. Bus Routing Capability

The bus routing stage therefore supports:

```
CH1 → BUS1
CH2 → BUS2
CH3 → BUS3
CH4 → BUS4
```

but can also create:

```
CH1 → BUS1
CH1 → BUS2
CH1 → BUS3
CH1 → BUS4
```

or:

```
CH1 → BUS1
CH2 → BUS1
CH3 → BUS1
CH4 → BUS1
```

depending on the MUX configuration.

This is an important distinction.

A channel still has only one physical output, but **that output may be selected by multiple bus MUXes**.

Therefore the architecture permits fan-out at the routing stage without adding a MUX to each channel.

---

# 8. Internal Video Bus

The internal video bus is the central distribution fabric.

The bus consists of:

```
k × m lanes
```

where:

* `m` = base bus width
* `k` = number of bus groups
* `B = k × m` = total number of internal video lanes

The preferred base configuration is:

```
m = 4
```

The architecture shall permit additional groups of four lanes to be added.

Example:

```
k = 1
m = 4

BUS0
BUS1
BUS2
BUS3
```

Expansion:

```
k = 2
m = 4

BUS0 ... BUS7
```

Further expansion:

```
k = 3
m = 4

BUS0 ... BUS11
```

---

# 9. Bus Width Recommendation

The preferred base bus width is:

```
m = 4
```

Four lanes provide useful routing flexibility while keeping the first implementation compact.

Two-lane variants may be supported where a smaller design is required:

```
m = 2
```

However, the standard expansion unit should preferably remain four lanes.

## Preferred choice

**4 internal video lanes per expansion group.**

---

# 10. Bus Capacity

The preferred relationship is:

```
B ≥ n
```

where:

* `B` = total internal video lanes
* `n` = number of physical input channels

This means the internal bus has at least one lane per input channel.

For the initial prototype:

```
n = 4
m = 4
k = 1
```

therefore:

```
B = 4
```

This gives:

```
4 channel outputs
    ↓
4 independent bus lanes
```

and allows a complete one-to-one channel assignment.

---

# 11. Why Match Bus Capacity to Channel Count

With four channels and four bus lanes:

```
CH1 → BUS0
CH2 → BUS1
CH3 → BUS2
CH4 → BUS3
```

all four source signals can simultaneously exist on independent internal lanes.

The output matrix can then independently combine them.

This avoids losing routing information before the mixer.

For a general system:

```
n channels
B ≥ n buses
```

provides sufficient capacity to expose every input independently to the output matrix.

This is the preferred architecture for maximum flexibility.

---

# 12. Output / Mixer Module

Each output/mixer module is independent.

A single-output mixer consumes all internal bus lanes:

```
BUS0 ──► gain ──┐
BUS1 ──► gain ──┤
BUS2 ──► gain ──┼──► SUM ──► output driver ──► OUT
BUS3 ──► gain ──┘
```

For `B` bus lanes:

```
OUTj = Σ(Gji × BUSi)
```

Each bus contribution has an independently controlled gain.

---

# 13. Full Matrix Mixing

The output stage shall support full matrix mixing.

For `B` internal bus lanes and `j` output modules:

```
OUT1 = G11·BUS1 + G12·BUS2 + ... + G1B·BUSB

OUT2 = G21·BUS1 + G22·BUS2 + ... + G2B·BUSB

...

OUTj = Gj1·BUS1 + Gj2·BUS2 + ... + GjB·BUSB
```

Every output can receive every internal bus lane.

The resulting matrix is:

```
                 INTERNAL BUS
              B1    B2    B3   ...   BB
           ┌────────────────────────────
OUT1       │ G11  G12   G13  ...  G1B
OUT2       │ G21  G22   G23  ...  G2B
OUT3       │ G31  G32   G33  ...  G3B
...        │ ...
OUTj       │ Gj1  Gj2   Gj3  ...  GjB
           └────────────────────────────
```

This is the full output mixing matrix.

---

# 14. Preferred Output Matrix Implementation

The preferred architecture is:

```
INTERNAL BUS
     │
     ├──► VCA / variable gain ──┐
     ├──► VCA / variable gain ──┤
     ├──► VCA / variable gain ──┤
     └──► VCA / variable gain ──┤
                                ▼
                               SUM
                                │
                                ▼
                          OUTPUT BUFFER
                                │
                                ▼
                              OUT
```

There is one gain cell for each:

```
bus lane × output
```

Therefore:

```
Number of gain cells = B × j
```

For the 4 × 4 prototype:

```
4 bus lanes × 4 outputs
= 16 gain cells
```

This provides the complete output matrix.

---

# 15. Why Routing and Mixing Are Separated

The architecture deliberately separates two functions.

## Routing

```
Channel → Internal Bus
```

controlled by the centralized MUX stage.

## Mixing

```
Internal Bus → Output
```

controlled by the output matrix.

Therefore:

```
CHANNEL
   │
   ▼
MUX / ROUTING
   │
   ▼
BUS
   │
   ▼
MATRIX GAIN
   │
   ▼
SUM
   │
   ▼
OUTPUT
```

This separation keeps each subsystem simple and makes the overall system scalable.

---

# 16. Pan / Balance

Pan and balance are naturally implemented in the output matrix.

For a stereo output pair:

```
BUS0 ──► G_L ──► OUT_L
    └──► G_R ──► OUT_R
```

The pan control changes:

```
G_L
G_R
```

For a centered signal:

```
G_L ≈ G_R
```

For a left-panned signal:

```
G_L → maximum
G_R → minimum
```

For a right-panned signal:

```
G_L → minimum
G_R → maximum
```

No dedicated pan circuitry is required on the channel module.

---

# 17. Output Module Scaling

The number of output modules is independent of the number of input channels.

Let:

```
n = input channels
B = internal bus lanes
j = output modules
```

Then the system provides:

```
n → B → j
```

with:

```
B ≥ n
```

preferred.

Examples:

```
4 channels → 4 buses → 2 outputs

4 channels → 4 buses → 4 outputs

8 channels → 8 buses → 4 outputs

8 channels → 8 buses → 8 outputs
```

The output matrix scales with:

```
B × j
```

rather than directly with:

```
n × j
```

because the bus isolates the channel layer from the output layer.

---

# 18. Master Module

The master module owns the digital control plane.

It shall control:

* channel-to-bus MUX selection
* output matrix gain
* output mute/enable
* module configuration
* module addressing
* presets/scenes
* optional synchronization
* user interface

The master module should not carry the main analog video signal.

Conceptually:

```
MASTER
  │
  ├── digital control ──► BUS MUX
  │
  ├── digital control ──► MIXER MODULES
  │
  └── user interface / configuration
```

---

# 19. Digital Control

The digital control architecture should be modular and addressable.

Preferred initial implementation:

**SPI-style serial control with local module selection/addressing.**

The control bus should allow:

* adding channel modules
* adding bus MUX modules
* adding output modules
* configuring modules independently

without requiring a dedicated control line for every analog device.

The exact interface shall be finalized after selecting the MUX and VCA components.

---

# 20. Physical Partition

## Channel PCB

Contains:

```
INPUT
  ↓
Protection / termination
  ↓
Input buffer
  ↓
Level conditioning
  ↓
SINGLE INTERNAL OUTPUT
```

No MUX.

No matrix circuitry.

No mixing.

---

## Bus / Routing PCB

Contains:

```
Channel inputs
     │
     ▼
MUX bank
     │
     ▼
Internal video bus
```

The MUX bank provides the channel-to-bus routing.

This module may be implemented as part of the backplane or as a dedicated routing PCB.

---

## Internal Video Backplane

Contains:

* internal video bus lanes
* digital control bus
* power distribution
* module connectors

The backplane should contain as little active analog circuitry as practical.

---

## Output / Mixer PCB

Contains:

```
BUS lanes
   │
   ├──► VCA ──┐
   ├──► VCA ──┤
   ├──► VCA ──┤
   └──► VCA ──┤
              ▼
             SUM
              │
         output driver
              │
            OUT
```

One output module represents one independent matrix output.

---

## Master PCB

Contains:

* MCU
* user interface
* digital control
* configuration storage
* module addressing
* optional communications

No primary video signal path.

---

# 21. Bus Electrical Architecture

If the internal signal remains conventional PAL/composite video, the internal bus should be designed around the appropriate video impedance, normally:

```
75 Ω
```

The bus architecture must account for:

* source impedance
* termination
* capacitive loading
* trace impedance
* reflections
* connector loading
* crosstalk
* signal attenuation

A passive multi-drop bus should not be assumed to behave correctly simply because its nominal frequency is only 20 MHz.

The preferred architecture is therefore:

```
Channel
   │
   ▼
Channel buffer
   │
   ▼
Bus MUX
   │
   ▼
Bus driver / isolation
   │
   ▼
Internal bus
```

where required.

---

# 22. Bus MUX Loading

Because every channel output may feed multiple MUX inputs, the channel output must be capable of driving the aggregate load.

The channel output stage should therefore be designed with the bus MUX bank in mind.

Preferred:

```
Channel
   │
   ▼
Wideband video buffer
   │
   ▼
Multiple MUX inputs
```

The buffer should provide adequate:

* output drive
* bandwidth
* stability
* isolation
* video linearity

The exact requirement depends on the selected MUX topology.

---

# 23. Component Selection Priorities

Component selection should follow:

1. Signal integrity
2. Video performance
3. Bandwidth
4. Low crosstalk
5. Low distortion
6. Low component count
7. Simple digital control
8. Availability
9. Cost

For the 20 MHz system target, candidate analog switches, buffers, VGAs/VCAs and summing amplifiers should be evaluated against the actual PAL signal requirements rather than bandwidth alone.

Important parameters include:

* bandwidth
* insertion loss
* crosstalk
* differential gain
* differential phase
* settling time
* glitch behavior
* output drive
* noise
* distortion

---

# 24. First Prototype

The preferred first prototype is:

```
n = 4 input channels
m = 4 bus lanes
k = 1 bus group
B = 4 internal video lanes
j = 4 output modules
```

Signal architecture:

```
CH1 ──► channel ──┐
CH2 ──► channel ──┤
CH3 ──► channel ──┼──► 4-channel MUX bank
CH4 ──► channel ──┘
                       │
                       ▼
                   BUS0..BUS3
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       MIX OUT1     MIX OUT2     MIX OUT3
          │            │            │
          └────────────┼────────────┘
                       ▼
                    MIX OUT4
```

Each output module receives all four internal bus lanes.

Each output module has four independently controlled gain paths.

Total matrix gain cells:

```
4 buses × 4 outputs = 16
```

---

# 25. Example Routing

The bus MUX can establish:

```
BUS0 = CH1
BUS1 = CH2
BUS2 = CH3
BUS3 = CH4
```

The output matrix can then create:

```
OUT1 = 1.0·BUS0 + 0.5·BUS1

OUT2 = 1.0·BUS1 + 1.0·BUS2

OUT3 = 0.5·BUS0 + 0.5·BUS2 + 1.0·BUS3

OUT4 = 1.0·BUS0 + 1.0·BUS1
     + 1.0·BUS2 + 1.0·BUS3
```

Thus every output can independently mix every available channel.

---

# 26. Scaling Example

For eight channels:

```
CH1 ──┐
CH2 ──┤
...   ├──► 8-channel routing
CH8 ──┘
          │
          ▼
      BUS0..BUS7
          │
    ┌─────┼─────┐
    ▼     ▼     ▼
   OUT1  OUT2  ... OUTj
```

Using four-lane expansion groups:

```
m = 4
k = 2
```

therefore:

```
B = 4 × 2
  = 8 lanes
```

The channel PCB remains unchanged.

Only the bus/routing capacity and required mixer matrix size increase.

---

# 27. Final Mathematical Model

Let:

```
X = [X1, X2, ..., Xn]
```

be the physical channel signals.

The centralized bus MUX generates:

```
B = R X
```

where:

```
R = channel-to-bus routing matrix
```

Each bus lane selects a channel.

The output matrix generates:

```
Y = G B
```

where:

```
G = output mixing matrix
```

Therefore:

```
Y = G R X
```

This gives a clean separation:

```
R → routing
G → mixing
```

The architecture can therefore scale independently in all three dimensions:

```
n = number of input channels
B = number of internal bus lanes
j = number of output modules
```

with the preferred constraint:

```
B ≥ n
```

---

# 28. Final Architecture

The final preferred architecture is:

```
┌──────────────────────────────────────┐
│           INPUT CHANNELS             │
│                                      │
│ CH1 ──► processing ──► single OUT    │
│ CH2 ──► processing ──► single OUT    │
│ CH3 ──► processing ──► single OUT    │
│ ...                                  │
│ CHn ──► processing ──► single OUT    │
└──────────────────┬───────────────────┘
                   │
                   ▼
┌──────────────────────────────────────┐
│         CENTRAL BUS MUX BANK         │
│                                      │
│ MUX1 ──► BUS0                        │
│ MUX2 ──► BUS1                        │
│ MUX3 ──► BUS2                        │
│ ...                                  │
│ MUXB ──► BUSB                        │
└──────────────────┬───────────────────┘
                   │
                   ▼
┌──────────────────────────────────────┐
│         INTERNAL VIDEO BUS           │
│                                      │
│       B = k × m lanes                │
│       m = 4 preferred                │
│       B ≥ n preferred                │
└──────────────────┬───────────────────┘
                   │
      ┌────────────┼────────────┐
      ▼            ▼            ▼
   OUTPUT 1     OUTPUT 2      OUTPUT j
      │            │            │
   full          full         full
   matrix        matrix       matrix
      │            │            │
      ▼            ▼            ▼
    OUT1         OUT2         OUTj


                ▲
                │
         MASTER MODULE
         digital control
```

---

# 29. Preferred Choices Summary

| Section             | Preferred choice                                   |
| ------------------- | -------------------------------------------------- |
| Input module        | One PCB per physical input                         |
| Channel output      | **Exactly one standardized internal output**       |
| Channel routing     | **No MUX on channel PCB**                          |
| Routing location    | **Centralized bus MUX bank**                       |
| Bus MUX topology    | One independently controlled MUX per bus lane      |
| Base bus width      | **4 lanes**                                        |
| Expansion unit      | 4 additional lanes                                 |
| Total bus width     | `B = k × 4`                                        |
| Bus capacity        | **Prefer `B ≥ n`**                                 |
| Internal bus        | Wideband video backplane                           |
| Output architecture | Independent mixer/output modules                   |
| Output mixing       | **Full matrix**                                    |
| Output gain         | VCA/variable-gain cell per bus/output intersection |
| Gain cells          | `B × j`                                            |
| Pan/balance         | Matrix gain coefficients                           |
| Master              | Central digital controller                         |
| Digital control     | SPI-style modular control initially                |
| First prototype     | **4 channels / 4 buses / 4 outputs**               |
| First matrix        | **4 × 4 full output matrix**                       |

---

# 30. Core Design Principle

The system shall be treated as three independent signal layers:

```
CHANNEL LAYER

n independent sources
      │
      ▼

ROUTING LAYER

n sources → B internal buses
      │
      ▼

MIXING LAYER

B internal buses → j independent outputs
```

The channel module therefore remains deliberately simple:

```
INPUT → PROCESSING → SINGLE OUTPUT
```

The centralized bus MUX performs source assignment:

```
CHANNEL OUTPUTS → INTERNAL BUS
```

The output modules perform arbitrary mixing:

```
INTERNAL BUS → FULL MATRIX → OUTPUTS
```

This separation is the fundamental architecture of the new mixer design.
