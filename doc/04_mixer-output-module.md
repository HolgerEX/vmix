# Synkie / VMix — Mixer / Output Module

## 1. Purpose

Each mixer/output module produces one independent physical output from the internal video bus.

The module implements one row of the global output matrix.

```text
B INTERNAL BUSSES
       │
       ▼
VARIABLE GAIN
       │
       ▼
SUM
       │
       ▼
OUTPUT BUFFER
       │
       ▼
ONE PHYSICAL OUTPUT
```

---

## 2. Fundamental Architecture

For output `j`:

```text
BUS0 ──► G0 ──┐
BUS1 ──► G1 ──┤
BUS2 ──► G2 ──┼──► SUM ──► OUTPUT DRIVER ──► OUTj
...           │
BUSB ──► GB ──┘
```

There is one independently controlled gain path for every bus/output intersection.

---

## 3. Matrix

The complete output matrix is:

```text
                  BUS
             B0   B1   B2 ... BB
           ┌─────────────────────
OUT0       │ G00 G01 G02 ... G0B
OUT1       │ G10 G11 G12 ... G1B
OUT2       │ G20 G21 G22 ... G2B
...        │
OUTj       │ Gj0 Gj1 Gj2 ... GjB
```

Every output can independently receive every internal bus.

---

## 4. Gain Cell Count

The number of gain cells is:

```text
B × j
```

For the first prototype:

```text
B = 4
j = 4
```

therefore:

```text
4 × 4 = 16 gain cells
```

---

## 5. Preferred Analog Structure

The preferred implementation is:

```text
INTERNAL BUS
      │
      ├──► VARIABLE GAIN ──┐
      ├──► VARIABLE GAIN ──┤
      ├──► VARIABLE GAIN ──┤
      └──► VARIABLE GAIN ──┤
                           ▼
                          SUM
                           │
                           ▼
                    OUTPUT BUFFER
                           │
                           ▼
                          OUT
```

The exact VCA/VGA technology shall be selected based on:

* video bandwidth
* linearity
* noise
* control range
* control feedthrough
* distortion
* settling time
* availability
* component count

---

## 6. Matrix Mixing

Example:

```text
OUT1 = 1.0·BUS0 + 0.5·BUS1

OUT2 = 1.0·BUS1 + 1.0·BUS2

OUT3 = 0.5·BUS0 + 0.5·BUS2 + 1.0·BUS3

OUT4 = BUS0 + BUS1 + BUS2 + BUS3
```

The output modules do not need to know which physical channels are represented by the buses.

They only operate on the internal bus signals.

---

## 7. Pan / Balance

Pan and balance belong in the output matrix rather than on the channel PCB.

For a stereo output pair:

```text
BUS0 ──► G_L ──► OUT_L
      └─► G_R ──► OUT_R
```

The pan control changes the relative gain coefficients.

Centered:

```text
G_L ≈ G_R
```

Left:

```text
G_L > G_R
```

Right:

```text
G_R > G_L
```

No dedicated analog pan circuit is required on the channel module.

---

## 8. Output Scaling

The number of output modules is independent of the number of physical channels.

Examples:

```text
4 channels → 4 buses → 2 outputs
4 channels → 4 buses → 4 outputs
8 channels → 8 buses → 4 outputs
8 channels → 8 buses → 8 outputs
```

The channel PCB remains unchanged.

---

## 9. Output Responsibilities

The mixer/output module is responsible for:

* receiving internal bus signals
* applying independently controlled gain
* summing the selected/weighted signals
* maintaining signal integrity
* driving the physical output
* providing output termination/interface as required

It is not responsible for:

* selecting channel sources
* channel processing
* global routing
* system configuration

---

## 10. Preferred Module Definition

One output module represents:

```text
B bus inputs
      ↓
B independently controlled gain paths
      ↓
one analog summing node
      ↓
one physical output
```

This makes the output stage naturally scalable.

---

## 11. Design Principle

The mixer/output module should be treated as one independent matrix row:

```text
[B0 B1 B2 ... BB]
        │
        ▼
     OUTj
```

Adding outputs means adding matrix rows.

Increasing bus capacity means increasing the number of gain paths per row.

