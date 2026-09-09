# Synkie / VMix — Internal Video Bus

## 1. Purpose

The internal video bus is the analog distribution fabric between the centralized bus router and the output/mixer modules.

```text
BUS ROUTER
    │
    ▼
INTERNAL VIDEO BUS
    │
    ├──► OUTPUT MODULE 1
    ├──► OUTPUT MODULE 2
    ├──► ...
    └──► OUTPUT MODULE j
```

The bus shall be electrically designed as a high-bandwidth video interconnect.

---

## 2. Bus Width

The total number of lanes is:

```text
B = k × m
```

where:

* `m` = lanes per expansion group
* `k` = number of groups

Preferred:

```text
m = 4
```

Therefore:

```text
4 lanes
8 lanes
12 lanes
...
```

are natural system configurations.

---

## 3. First Prototype

```text
k = 1
m = 4
B = 4
```

The first prototype therefore has:

```text
BUS0
BUS1
BUS2
BUS3
```

---

## 4. Bus Capacity

For maximum routing flexibility:

```text
B ≥ n
```

is preferred.

For four physical inputs:

```text
n = 4
B = 4
```

so every channel can occupy its own bus simultaneously.

The architecture does not require `B ≥ n`; reduced bus capacity is possible when the application permits it.

---

## 5. Electrical Characteristics

If the internal signal remains conventional PAL/composite video, the bus shall be designed around the appropriate video impedance, typically:

```text
75 Ω
```

The actual architecture must account for:

* source termination
* destination termination
* trace impedance
* connector impedance
* capacitive loading
* reflections
* crosstalk
* attenuation
* fan-out
* bus length

A nominal signal frequency of 20 MHz does not make transmission-line effects irrelevant.

---

## 6. Bus Topology

The physical bus should be designed deliberately rather than treated as a generic digital backplane.

Preferred conceptual architecture:

```text
CHANNEL
   │
   ▼
BUS MUX
   │
   ▼
BUS DRIVER
   │
   ▼
VIDEO BACKPLANE
   │
   ├──► OUTPUT MODULE
   ├──► OUTPUT MODULE
   ├──► OUTPUT MODULE
   └──► OUTPUT MODULE
```

Whether buffering is required at the router, module inputs, or both shall be established during electrical design.

---

## 7. Backplane

The backplane should carry:

### Analog

* internal video bus lanes

### Digital

* module control bus

### Power

* required supply rails

The backplane should contain as little active analog circuitry as practical.

Its primary purpose is interconnection.

---

## 8. Expansion

The bus shall preferably expand in four-lane groups.

Example:

```text
GROUP 0
BUS0
BUS1
BUS2
BUS3
```

Add:

```text
GROUP 1
BUS4
BUS5
BUS6
BUS7
```

The channel PCB does not change when the bus is expanded.

Only the routing and output matrix capacity must increase.

---

## 9. Signal Integrity Requirements

The bus design shall be evaluated for:

* amplitude loss
* frequency response
* phase response
* reflections
* crosstalk
* ground bounce
* connector discontinuities
* switching artifacts
* module loading

The bus should be validated using the actual intended PCB stackup, connectors, module count, and termination scheme.

---

## 10. Design Principle

The internal bus is an analog-video infrastructure layer.

It should be:

* predictable
* low-loss
* low-crosstalk
* mechanically modular
* electrically standardized
* independent of channel and output module functionality
