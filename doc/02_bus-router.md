# Synkie / VMix — Central Bus Router

## 1. Purpose

The bus router converts the set of channel outputs into the internal video bus.

Its function is:

```text
n CHANNEL OUTPUTS
       │
       ▼
CENTRAL SOURCE SELECTION
       │
       ▼
B INTERNAL BUS LANES
```

Routing is centralized rather than distributed across the channel modules.

---

## 2. Fundamental Architecture

For each bus lane:

```text
BUSi = MUX(CH1 ... CHn)
```

Therefore:

```text
CH1 ─┬────► MUX1 ──► BUS0
CH2 ─┤
...  ┤
CHn ─┘

CH1 ─┬────► MUX2 ──► BUS1
CH2 ─┤
...  ┤
CHn ─┘

...

CH1 ─┬────► MUXB ──► BUSB
CH2 ─┤
...  ┤
CHn ─┘
```

Each bus lane independently selects one channel.

---

## 3. Routing Capability

The router supports independent source assignment.

Example:

```text
BUS0 = CH1
BUS1 = CH2
BUS2 = CH3
BUS3 = CH4
```

It can also duplicate a source:

```text
BUS0 = CH1
BUS1 = CH1
BUS2 = CH1
BUS3 = CH1
```

or route several sources to the same destination at different times/configurations.

Important:

**The router is a selector, not a mixer.**

One bus lane selects one source.

---

## 4. Fan-Out

A channel can be selected by multiple bus selectors.

```text
              ┌──► BUS0
              ├──► BUS1
CH1 ──────────┼──► BUS2
              └──► BUS3
```

This provides routing fan-out without requiring multiple outputs from the channel PCB.

---

## 5. Routing Matrix Concept

Conceptually the routing relationship can be represented as:

```text
             BUS
          B0 B1 B2 ... BB
       ┌─────────────────
CH1    │ x  x  x  ... x
CH2    │ x  x  x  ... x
CH3    │ x  x  x  ... x
...    │
CHn    │ x  x  x  ... x
```

However, the physical implementation does not require a conventional full crosspoint.

The preferred implementation is:

```text
one source-selection MUX per bus lane
```

This is simpler and directly matches the required routing function.

---

## 6. Electrical Architecture

The router must account for:

* MUX input capacitance
* channel-output drive capability
* MUX insertion loss
* crosstalk
* bandwidth
* switching transients
* source impedance
* bus loading

A suitable architecture may be:

```text
CHANNEL OUTPUT
      │
      ▼
MUX BANK
      │
      ▼
BUS DRIVER / BUFFER
      │
      ▼
INTERNAL BUS
```

The need for a post-MUX driver shall be determined by the selected analog switch topology and bus electrical design.

---

## 7. Channel Loading

Every channel output may connect to multiple MUX inputs.

Therefore the channel output must be designed for the aggregate load.

The router design shall determine:

* number of MUX inputs per channel
* MUX input impedance
* MUX input capacitance
* required channel output current
* required buffer topology

This is an important interface constraint between the channel and routing modules.

---

## 8. Bus Count

The router supports:

```text
B = k × m
```

bus lanes.

The preferred base expansion unit is:

```text
m = 4
```

For the first prototype:

```text
B = 4
```

---

## 9. Digital Control

Each bus selector shall have an independently controllable source selection.

Conceptually:

```text
MASTER
  │
  ├──► MUX0 source select
  ├──► MUX1 source select
  ├──► MUX2 source select
  └──► MUX3 source select
```

The actual serial-control implementation shall be finalized after the analog MUX devices have been selected.

---

## 10. Preferred Choice

**Centralized MUX bank with one independently controlled source selector per internal bus lane.**

No routing MUX is placed on the channel PCB.
