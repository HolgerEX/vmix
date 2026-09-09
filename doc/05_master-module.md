# Synkie / VMix — Master Module

## 1. Purpose

The Master module implements the digital control plane of the mixer.

It controls:

* bus routing
* output gain
* output enable/mute
* module configuration
* addressing
* presets/scenes
* user interface
* optional synchronization
* optional external communications

It does not carry the primary analog video signal.

---

## 2. Architecture

```text
                 MASTER
                   │
        ┌──────────┼──────────┐
        │          │          │
        ▼          ▼          ▼
    BUS ROUTER  OUTPUT 1   OUTPUT j
                OUTPUT 2
```

The analog signal remains physically separate:

```text
CHANNEL → ROUTER → BUS → OUTPUT
```

---

## 3. Control Responsibilities

### Bus routing

The Master determines which channel is assigned to each internal bus.

Example:

```text
BUS0 = CH1
BUS1 = CH2
BUS2 = CH3
BUS3 = CH4
```

### Output matrix

The Master controls the gain coefficient for every bus/output intersection.

For a 4 × 4 matrix:

```text
G00 ... G03
G10 ... G13
G20 ... G23
G30 ... G33
```

Total:

```text
16 independently controlled coefficients
```

---

## 4. Module Addressing

Modules should be addressable so that the system can be expanded without requiring a dedicated control line for every analog function.

The control architecture shall support:

* multiple bus-router devices
* multiple output modules
* independently addressed modules
* configuration during startup
* runtime parameter updates

---

## 5. Preferred Initial Control Interface

The initial implementation should use an:

**SPI-style serial control architecture with local module selection/addressing.**

The exact topology shall be finalized after selection of:

* analog MUX devices
* VCA/VGA devices
* digital control requirements
* module connector architecture

---

## 6. Control Separation

The Master should control analog devices but should not become part of their signal path.

Preferred:

```text
                    ┌─────────────┐
                    │   MASTER    │
                    └──────┬──────┘
                           │
                     DIGITAL CONTROL
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
       BUS MUX          OUTPUT 1         OUTPUT j
          │                │                │
          ▼                ▼                ▼
       ANALOG            ANALOG           ANALOG
       VIDEO             VIDEO            VIDEO
```

This provides a clean separation between:

```text
CONTROL PLANE
```

and:

```text
ANALOG VIDEO PLANE
```

---

## 7. User Interface

The Master is the natural location for:

* physical controls
* displays
* preset management
* scene recall
* configuration
* status indication

The exact user interface is outside the scope of the analog module architecture.

---

## 8. Presets / Scenes

A complete mixer state can be represented by:

```text
ROUTING STATE
+
OUTPUT MATRIX STATE
+
GLOBAL CONFIGURATION
```

For example:

```text
Routing:
BUS0 = CH1
BUS1 = CH2
BUS2 = CH3
BUS3 = CH4

Matrix:
OUT1 = [1, 0, 0, 0]
OUT2 = [0, 1, 0, 0]
OUT3 = [0, 0, 1, 0]
OUT4 = [0, 0, 0, 1]
```

The Master can store and recall such configurations as scenes.

---

## 9. Synchronization

Synchronization is optional at the initial architectural level.

Potential future functions include:

* coordinated parameter updates
* synchronized switching
* frame/line synchronization
* external control
* deterministic scene changes

These should not complicate the initial analog architecture unless required.

---

## 10. Design Principle

The Master is a **control-plane module**, not a video-processing module.

Its preferred role is:

```text
CONFIGURE
CONTROL
STORE
DISPLAY
COMMUNICATE
```

while the analog video path remains:

```text
CHANNEL → ROUTER → BUS → MIXER → OUTPUT
```
