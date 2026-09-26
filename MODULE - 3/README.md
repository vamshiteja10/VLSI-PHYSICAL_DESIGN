# Module 3: Standard Cell Design Using Magic and NGSpice

## 📌 Module Overview

This module explores the complete process of creating and characterizing a CMOS inverter standard cell. The work connects **transistor-level circuit design, SPICE simulation, physical layout, parasitic extraction, and timing analysis** into a single design flow.

The main tools used in this module are **NGSpice** for circuit simulation and **Magic** for physical layout. Along the way, the module also introduces important concepts such as CMOS fabrication, transistor sizing, switching threshold, propagation delay, rise/fall time, and layout parasitics.

The overall journey followed in this module is:

```text
CMOS Inverter
      ↓
SPICE Netlist
      ↓
NGSpice Simulation
      ↓
DC & Transient Analysis
      ↓
Transistor Sizing
      ↓
Magic Layout
      ↓
DRC Verification
      ↓
Parasitic Extraction
      ↓
Post-Layout Simulation
      ↓
Cell Characterization
      ↓
Standard Cell
```

---

# 📚 Table of Contents

1. [Introduction](#1-introduction)
2. [Objectives](#2-objectives)
3. [Software and Technology Used](#3-software-and-technology-used)
4. [Understanding the CMOS Inverter](#4-understanding-the-cmos-inverter)
5. [Creating a SPICE Description](#5-creating-a-spice-description)
6. [CMOS Inverter Netlist](#6-cmos-inverter-netlist)
7. [NGSpice Analysis](#7-ngspice-analysis)
8. [Voltage Transfer Characteristic](#8-voltage-transfer-characteristic)
9. [Switching Threshold](#9-switching-threshold)
10. [Transient Analysis](#10-transient-analysis)
11. [Static and Dynamic Behaviour](#11-static-and-dynamic-behaviour)
12. [Transistor Sizing](#12-transistor-sizing)
13. [Rise Time, Fall Time and Delay](#13-rise-time-fall-time-and-delay)
14. [Standard Cell Repository](#14-standard-cell-repository)
15. [Magic Layout](#15-magic-layout)
16. [Understanding CMOS Fabrication](#16-understanding-cmos-fabrication)
17. [Substrate and Active Region](#17-substrate-and-active-region)
18. [Well Formation](#18-well-formation)
19. [Gate Formation](#19-gate-formation)
20. [LDD Formation](#20-ldd-formation)
21. [Source and Drain Formation](#21-source-and-drain-formation)
22. [Contacts and Interconnects](#22-contacts-and-interconnects)
23. [Metal Formation](#23-metal-formation)
24. [Chemical Mechanical Polishing](#24-chemical-mechanical-polishing)
25. [Passivation](#25-passivation)
26. [Complete CMOS Fabrication Flow](#26-complete-cmos-fabrication-flow)
27. [Layout-to-Simulation Flow](#27-layout-to-simulation-flow)
28. [Standard Cell Characterization](#28-standard-cell-characterization)
29. [Important SPICE Parameters](#29-important-spice-parameters)
30. [Pre-Layout vs Post-Layout Simulation](#30-pre-layout-vs-post-layout-simulation)
31. [Key Observations](#31-key-observations)
32. [Why Physical Layout Matters](#32-why-physical-layout-matters)
33. [Overall Design Flow](#33-overall-design-flow)
34. [Key Takeaways](#34-key-takeaways)
35. [Conclusion](#35-conclusion)

---

# 1. Introduction

A digital circuit may look simple at the logic level, but its actual implementation starts much deeper at the transistor and layout levels.

In this module, a **CMOS inverter** is used as the basic circuit for understanding this complete process. The inverter is first described using a SPICE netlist and simulated with NGSpice. Its electrical characteristics are then studied through DC and transient simulations.

After verifying the circuit behaviour, the inverter is physically implemented using **Magic layout**. The layout is checked against design rules and then converted back into a circuit using parasitic extraction. This extracted circuit can be simulated again to observe the difference between ideal schematic behaviour and practical physical behaviour.

This makes the module useful for understanding how a transistor-level circuit eventually becomes a usable **standard-cell library element**.

---

# 2. Objectives

The major objectives of this module are:

* Understand the working principle of a CMOS inverter.
* Learn how to write a SPICE netlist.
* Understand transistor terminals and circuit connectivity.
* Perform DC analysis using NGSpice.
* Generate the inverter voltage transfer characteristic.
* Determine the switching threshold.
* Perform transient simulation.
* Measure rise time and fall time.
* Understand propagation delay.
* Study the effect of transistor sizing.
* Create a physical CMOS layout using Magic.
* Understand design-rule verification.
* Extract parasitic components from the layout.
* Perform post-layout SPICE simulation.
* Understand the major stages of CMOS fabrication.
* Learn how contacts and metal interconnections are formed.
* Understand the basic idea behind standard-cell characterization.

---

# 3. Software and Technology Used

| Tool / Technology | Purpose                                            |
| ----------------- | -------------------------------------------------- |
| **NGSpice**       | Circuit simulation and electrical characterization |
| **Magic**         | CMOS physical layout                               |
| **Git / GitHub**  | Obtaining and managing design files                |
| **SKY130**        | Open-source CMOS process technology                |
| **SPICE Models**  | Describing transistor electrical behaviour         |

---

# 4. Understanding the CMOS Inverter

The CMOS inverter is one of the simplest and most important digital circuits.

It contains two MOSFETs:

* One **PMOS**
* One **NMOS**

The PMOS provides the connection from the output towards the positive supply, while the NMOS provides the connection from the output towards ground.

```text
             VDD
              |
            PMOS
              |
              +-------- VOUT
              |
            NMOS
              |
             GND

              ↑
       Gates connected together
              |
             VIN
```

Both transistor gates receive the same input signal.

### Logic Operation

| VIN | PMOS | NMOS | VOUT |
| --- | ---- | ---- | ---- |
| 0   | ON   | OFF  | 1    |
| 1   | OFF  | ON   | 0    |

Therefore:

```text
VIN = 0  →  VOUT ≈ VDD

VIN = VDD → VOUT ≈ 0
```

This complementary switching action is what gives CMOS logic its low static power consumption.

---

# 5. Creating a SPICE Description

Before creating a physical layout, the circuit needs to be represented in a form that a simulator can understand.

A **SPICE deck** is simply a text-based description of the circuit.

It contains information such as:

* Transistor connections
* Node names
* Device dimensions
* Supply voltages
* Input sources
* Technology models
* Analysis commands
* Simulation conditions

A typical SPICE workflow is:

```text
Identify Circuit
      ↓
Define Connections
      ↓
Name the Nodes
      ↓
Specify Device Parameters
      ↓
Add Technology Models
      ↓
Define Input and Supply
      ↓
Select Simulation
      ↓
Run NGSpice
      ↓
Study Results
```

---

# 6. CMOS Inverter Netlist

A basic inverter can be represented using the following SPICE structure:

```spice
* CMOS Inverter

* PMOS transistor
M1 out in vdd vdd pmos W=0.375u L=0.25u

* NMOS transistor
M2 out in 0 0 nmos W=0.375u L=0.25u

* Power supply
Vdd vdd 0 2.5

* Input source
Vin in 0 0

* DC sweep
.dc Vin 0 2.5 0.05

* Technology model
.include tsmc_025_um_model.mod

.end
```

The exact model file and transistor syntax depend on the technology being used. For the SKY130 flow, the corresponding technology models and device names should be used.

---

# 7. NGSpice Analysis

NGSpice provides several types of circuit analysis. Two of the most useful analyses for an inverter are **DC analysis** and **transient analysis**.

## 7.1 Operating Point

The `.op` command calculates the DC operating point of the circuit.

```spice
.op
```

It provides the voltages and currents of the circuit for a particular bias condition.

---

## 7.2 DC Sweep

The `.dc` command changes a source over a specified range and calculates the circuit response.

```spice
.dc Vin 0 2.5 0.05
```

Here:

| Parameter | Meaning                    |
| --------- | -------------------------- |
| `Vin`     | Voltage source being swept |
| `0`       | Initial value              |
| `2.5`     | Final value                |
| `0.05`    | Increment                  |

The resulting graph gives the **Voltage Transfer Characteristic (VTC)** of the inverter.

---

# 8. Voltage Transfer Characteristic

The VTC shows how the output voltage changes as the input voltage is gradually increased.

A typical inverter characteristic looks like:

```text
VOUT
  |
VDD|───────
  |       \
  |        \
  |         \
  |          \
  |           \
  |            ───────
  |
  +---------------------- VIN
              ↑
             Vm
```

At low input voltage, the PMOS is conducting and the output remains HIGH.

As the input voltage increases, the NMOS begins conducting more strongly and the output starts falling.

Eventually, the NMOS dominates and the output approaches ground.

---

# 9. Switching Threshold

The **switching threshold**, commonly represented as `Vm`, is the point where the inverter changes its logic state.

For an idealized inverter, it is identified approximately from the point where:

```text
VIN = VOUT
```

The switching threshold is visible on the VTC curve.

### Why is Vm important?

It provides information about:

* Logic transition behaviour
* Noise margins
* Relative PMOS/NMOS strength
* Inverter symmetry
* Digital signal reliability

Changing the transistor sizes changes the relative drive strengths and therefore shifts the switching point.

---

# 10. Transient Analysis

DC analysis tells us how the circuit behaves at different steady-state input voltages.

However, a real digital circuit continuously switches between logic states. To study this behaviour, **transient analysis** is used.

A pulse can be applied as the input:

```spice
Vin in 0 PULSE(0 2.5 0 1n 1n 10n 20n)
```

A transient simulation can then be performed using:

```spice
.tran 0.1n 100n
```

The resulting waveform allows us to observe:

* Input transition
* Output transition
* Rise time
* Fall time
* Propagation delay
* Switching behaviour

---

# 11. Static and Dynamic Behaviour

## Static Behaviour

Static analysis considers situations where the input has settled to a fixed value.

### Input LOW

```text
VIN = 0

PMOS → ON
NMOS → OFF

VOUT ≈ VDD
```

### Input HIGH

```text
VIN = VDD

PMOS → OFF
NMOS → ON

VOUT ≈ 0
```

---

## Dynamic Behaviour

Dynamic analysis considers what happens while the input is changing.

```text
Input Pulse
     ↓
Transistor Switching
     ↓
Output Transition
     ↓
Delay Measurement
     ↓
Rise/Fall Time Measurement
```

This is important because real digital circuits are limited by how quickly their outputs can respond to changing inputs.

---

# 12. Transistor Sizing

The physical dimensions of MOSFETs strongly affect inverter performance.

The most important parameters are:

```text
W = Channel Width
L = Channel Length
```

The approximate strength of a transistor depends on:

```text
W/L
```

For example, consider:

```text
Wn = 0.375 µm
Ln = 0.25 µm

Wp = 0.375 µm
Lp = 0.25 µm
```

Then:

```text
Wn/Ln = 0.375/0.25 = 1.5

Wp/Lp = 0.375/0.25 = 1.5
```

Therefore:

```text
Wn/Ln ≈ Wp/Lp
```

The actual sizing used in a practical standard cell depends on the technology, mobility differences, load requirements, and desired timing characteristics.

---

# 13. Rise Time, Fall Time and Delay

An output signal does not switch instantaneously.

### Rise Time

Rise time represents the time required for the output to move from a LOW voltage level to a HIGH voltage level.

```text
LOW → HIGH
```

### Fall Time

Fall time represents the time required for the output to move from HIGH to LOW.

```text
HIGH → LOW
```

The transition behaviour is influenced by:

* PMOS width
* NMOS width
* Channel length
* Load capacitance
* Supply voltage
* Input transition time
* Interconnect resistance
* Parasitic capacitance

Changing the PMOS-to-NMOS sizing ratio can therefore change the balance between rising and falling transitions.

---

# 14. Standard Cell Repository

The practical layout work can be performed using an open-source standard-cell design environment.

A standard-cell repository can be obtained using Git:

```bash
git clone https://github.com/nickson-jose/vsdstdcelldesign.git
```

Move into the repository:

```bash
cd vsdstdcelldesign
```

The inverter layout can then be opened using Magic with the appropriate technology information.

For example:

```bash
magic -T sky130A.tech sky130_inv.mag
```

The exact command may vary depending on the installed SKY130 environment and the available technology files.

---

# 15. Magic Layout

**Magic** is an open-source layout editor used for designing and inspecting integrated circuits.

At the layout level, the schematic devices are represented physically using different layers.

A CMOS inverter layout typically contains:

* PMOS active region
* NMOS active region
* Polysilicon
* Metal
* Contacts
* Well regions
* Substrate connections

A simplified representation is:

```text
             VDD
              |
           PMOS Source
              |
            PMOS
              |
              +-------- OUT
              |
            NMOS
              |
           NMOS Source
              |
             GND

          POLYSILICON
              |
             VIN
```

The polysilicon gate crosses the active regions to form the transistor gates.

---

# 16. Understanding CMOS Fabrication

A physical layout is eventually converted into an actual silicon structure through a semiconductor manufacturing process.

CMOS fabrication consists of many carefully controlled steps, including:

* Wafer preparation
* Oxidation
* Lithography
* Etching
* Doping
* Ion implantation
* Annealing
* Gate formation
* Source/drain formation
* Contact formation
* Metal deposition
* Planarization
* Passivation

Understanding these steps helps explain why specific layout design rules exist.

---

# 17. Substrate and Active Region

The fabrication process begins with a semiconductor wafer.

A simplified flow is:

```text
Silicon Wafer
     ↓
Surface Preparation
     ↓
Oxide Formation
     ↓
Masking
     ↓
Photolithography
     ↓
Etching
     ↓
Active Region Definition
```

Photolithography is used to transfer patterns from masks onto the wafer.

A photoresist layer is deposited, exposed, developed, and then used to control which portions of the underlying material are removed or protected.

---

# 18. Well Formation

CMOS technology requires different regions for implementing NMOS and PMOS devices.

These regions are formed through controlled doping processes.

Typical dopants include:

```text
Boron       → P-type doping

Phosphorus  → N-type doping
```

The general process is:

```text
Well Mask
    ↓
Ion Implantation
    ↓
Annealing
    ↓
Dopant Activation
    ↓
Well Formation
```

Annealing repairs implantation damage and activates the implanted dopants.

---

# 19. Gate Formation

The gate is one of the most important parts of a MOS transistor.

The basic process involves:

1. Preparing the surface.
2. Forming the gate dielectric.
3. Depositing polysilicon.
4. Patterning the polysilicon.
5. Defining the gate structure.

A simplified flow is:

```text
Gate Oxide
    ↓
Polysilicon Deposition
    ↓
Photoresist
    ↓
Masking
    ↓
Pattern Development
    ↓
Etching
    ↓
Polysilicon Gate
```

The gate oxide electrically separates the gate from the semiconductor while allowing the electric field from the gate to control the channel.

---

# 20. LDD Formation

LDD stands for **Lightly Doped Drain**.

LDD structures are introduced to reduce the electric field near the drain region.

A high electric field can contribute to reliability problems such as hot-carrier effects.

A simplified relationship is:

```text
E = V/d
```

where:

* `E` = Electric field
* `V` = Voltage
* `d` = Distance

The LDD region provides a more gradual doping transition near the drain and helps improve device reliability.

---

# 21. Source and Drain Formation

After the gate has been defined, source and drain regions are formed through controlled implantation.

The general sequence is:

```text
Source/Drain Mask
       ↓
Ion Implantation
       ↓
Annealing
       ↓
Dopant Activation
       ↓
Source and Drain
```

The source and drain provide the terminals through which current enters and leaves the transistor.

A simplified transistor structure is:

```text
                 Gate
                  |
            Polysilicon
                  |
       ┌──────────┴──────────┐
       │      Gate Oxide     │
       └─────────────────────┘
          |              |
       Source          Drain
          |              |
       ─────────────────────
             Silicon
```

---

# 22. Contacts and Interconnects

Once the transistor regions are created, electrical contacts are required to connect them to the metal layers.

The contact process generally includes:

```text
Dielectric Layer
      ↓
Contact Opening
      ↓
Surface Cleaning
      ↓
Contact Material Deposition
      ↓
Annealing
      ↓
Electrical Connection
```

Contact structures provide a low-resistance path between the semiconductor regions, polysilicon, and metal interconnects.

---

# 23. Metal Formation

Metal layers are used to connect devices together and distribute signals and power across the chip.

A simplified process is:

```text
Dielectric Deposition
       ↓
Contact/Via Formation
       ↓
Metal Deposition
       ↓
Patterning
       ↓
Etching
       ↓
Metal Interconnect
```

Modern ICs use multiple metal layers so that large numbers of devices can be interconnected efficiently.

---

# 24. Chemical Mechanical Polishing

**CMP**, or Chemical Mechanical Polishing, is used to flatten the wafer surface.

Without planarization, every additional layer would make the surface increasingly uneven.

CMP combines:

* Chemical material removal
* Mechanical polishing

The basic idea is:

```text
Uneven Surface
      ↓
Chemical + Mechanical Action
      ↓
Excess Material Removed
      ↓
Flat Surface
```

A relatively flat surface is important for reliable fabrication of subsequent layers.

---

# 25. Passivation

After the required devices and interconnect layers have been completed, the chip is protected using a passivation layer.

Passivation helps protect the underlying circuitry from:

* Moisture
* Contamination
* Mechanical damage
* Environmental effects

Selected openings can be created where external electrical connections are required.

---

# 26. Complete CMOS Fabrication Flow

A simplified CMOS manufacturing sequence can be represented as:

```text
Silicon Wafer
      ↓
Surface Cleaning
      ↓
Oxidation
      ↓
Photolithography
      ↓
Active Region Definition
      ↓
Well Formation
      ↓
Gate Oxide Formation
      ↓
Polysilicon Gate Formation
      ↓
LDD Formation
      ↓
Source/Drain Implantation
      ↓
Annealing
      ↓
Contact Formation
      ↓
Metal Deposition
      ↓
Metal Patterning
      ↓
CMP / Planarization
      ↓
Additional Metal Layers
      ↓
Passivation
      ↓
Final Chip
```

---

# 27. Layout-to-Simulation Flow

Once the physical layout has been completed, it can be converted into an electrical representation.

The complete flow is:

```text
Schematic
    ↓
SPICE Simulation
    ↓
Magic Layout
    ↓
DRC
    ↓
Parasitic Extraction
    ↓
Extracted Netlist
    ↓
Post-Layout NGSpice Simulation
    ↓
Compare with Pre-Layout Results
```

The extracted circuit includes additional resistance and capacitance introduced by the physical implementation.

---

# 28. Standard Cell Characterization

A standard cell is not ready for use immediately after drawing the layout.

Its electrical and timing behaviour must be characterized.

The overall process is:

```text
CMOS Circuit
     ↓
SPICE Simulation
     ↓
Device Sizing
     ↓
Physical Layout
     ↓
DRC Verification
     ↓
Parasitic Extraction
     ↓
Post-Layout Simulation
     ↓
Timing Measurements
     ↓
Library Characterization
     ↓
.lib File
     ↓
Digital ASIC Flow
```

The resulting library information can contain parameters related to:

* Cell delay
* Transition time
* Input capacitance
* Output behaviour
* Timing arcs
* Power characteristics

These values allow digital design tools to understand how the physical cell behaves.

---

# 29. Important SPICE Parameters

| Parameter / Command | Description                             |
| ------------------- | --------------------------------------- |
| `W`                 | MOSFET channel width                    |
| `L`                 | MOSFET channel length                   |
| `VDD`               | Supply voltage                          |
| `VIN`               | Input voltage                           |
| `VOUT`              | Output voltage                          |
| `Vm`                | Switching threshold                     |
| `.op`               | Operating-point analysis                |
| `.dc`               | DC sweep                                |
| `.tran`             | Transient analysis                      |
| `.include`          | Includes an external model/library file |
| `.end`              | Marks the end of the SPICE deck         |

---

# 30. Pre-Layout vs Post-Layout Simulation

There is an important difference between simulating an ideal schematic and simulating an extracted physical layout.

| Feature                | Pre-Layout                    | Post-Layout                               |
| ---------------------- | ----------------------------- | ----------------------------------------- |
| Circuit representation | Schematic/netlist             | Extracted layout                          |
| Parasitics             | Usually ignored or simplified | Included                                  |
| Interconnect effects   | Minimal                       | Included                                  |
| Delay                  | More idealized                | More realistic                            |
| Purpose                | Functional verification       | Physical verification and timing analysis |

The post-layout result generally provides a more realistic picture of the circuit because physical effects are included.

---

# 31. Key Observations

## Observation 1 — Switching Threshold

The switching point can be obtained from the VTC around the location where:

```text
VIN ≈ VOUT
```

Its position depends on the relative strength of the PMOS and NMOS devices.

---

## Observation 2 — Effect of Width

Increasing transistor width increases the approximate drive capability:

```text
Larger W
   ↓
Higher W/L
   ↓
Greater Drive Strength
   ↓
Changed Switching Behaviour
```

However, larger devices can also introduce greater parasitic capacitance.

---

## Observation 3 — Timing Behaviour

Rise and fall times are influenced by:

* Transistor dimensions
* Load capacitance
* Supply voltage
* Input slew
* Interconnect resistance
* Parasitic capacitance

Therefore, transistor sizing involves a trade-off between drive strength, area,
