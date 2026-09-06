# Floorplanning and Placement – PicoRV32A

## 📌 Introduction

Floorplanning and placement are important stages in the **physical design flow of a digital VLSI circuit**. After synthesis converts RTL into a gate-level netlist, the next step is to transform that logical representation into a physical arrangement of cells on silicon.

In this module, the **PicoRV32A** design is taken through the initial physical design stages using the OpenLane flow. The focus is on understanding how the chip area is defined, how power and I/O are organized, and how standard cells are positioned and optimized.

---

## 🎯 Objectives

The major objectives of this module are to understand:

* Core and die dimensions
* Aspect ratio and core utilization
* Floorplan creation
* OpenLane floorplan configuration
* Power distribution planning
* I/O pin placement
* Pre-placed cells and placement blockages
* Standard-cell placement
* Placement optimization
* Decoupling capacitors
* Placement statistics and physical metrics

---

# 1. Physical Design Flow

The synthesized netlist is gradually converted into a physical implementation through several stages.

```text
RTL Design
    │
    ▼
Logic Synthesis
    │
    ▼
Gate-Level Netlist
    │
    ▼
Floorplanning
    │
    ▼
Power Planning
    │
    ▼
I/O Pin Placement
    │
    ▼
Standard-Cell Placement
    │
    ▼
Placement Optimization
```

Each stage prepares the design for the following physical implementation step.

---

# 2. Understanding the Core and Die

## Core

The **core** is the main region of the chip where standard cells and other logic elements are placed.

It contains the functional portion of the design and provides the area required for implementing the synthesized logic.

## Die

The **die** represents the complete physical chip area. It includes the core as well as the surrounding regions required for I/O, power distribution, and other physical structures.

```text
+--------------------------------+
|              DIE               |
|                                |
|       +----------------+       |
|       |                |       |
|       |      CORE      |       |
|       |                |       |
|       +----------------+       |
|                                |
+--------------------------------+
```

The relationship between the die and core is an important part of floorplan design because insufficient area can lead to congestion and timing problems.

---

# 3. Aspect Ratio and Core Utilization

## Aspect Ratio

The **aspect ratio** describes the shape of the core.

It is calculated as:

```text
Aspect Ratio = Core Height / Core Width
```

For example:

```text
Aspect Ratio = 1
```

indicates a square-shaped core.

Values greater or less than 1 result in a rectangular floorplan.

Choosing an appropriate aspect ratio is important because the physical shape of the design can influence routing, congestion, timing, and overall chip area.

---

## Core Utilization

Core utilization indicates how much of the available core area is occupied by logic cells.

```text
                  Area occupied by cells
Utilization = ------------------------------- × 100
                     Total core area
```

A higher utilization packs more cells into the available area. However, excessive utilization can make routing difficult and increase congestion.

<img width="955" height="970" alt="floorplan" src="https://github.com/user-attachments/assets/f0983f6c-cd40-484b-a736-d99432cf236c" />


---

# 4. Floorplanning

Floorplanning is one of the first major steps in physical design.

It establishes the physical framework in which the circuit will be implemented.

During floorplanning, several important decisions are made:

* Core dimensions
* Die dimensions
* Aspect ratio
* Core utilization
* I/O locations
* Placement of large blocks
* Power distribution requirements
* Regions reserved for special cells

A carefully designed floorplan can reduce routing congestion and improve the timing and reliability of the final design.



---

# 5. Floorplan Parameters in OpenLane

OpenLane provides several configuration variables that control the physical dimensions and arrangement of the design.

Some important parameters include:

```text
FP_CORE_UTIL
FP_ASPECT_RATIO
FP_SIZING
DIE_AREA
FP_IO_HMETAL
FP_IO_VMETAL
FP_IO_MODE
FP_PDN_VPITCH
FP_PDN_HPITCH
```

### Purpose of these parameters

| Parameter         | Purpose                                |
| ----------------- | -------------------------------------- |
| `FP_CORE_UTIL`    | Controls target core utilization       |
| `FP_ASPECT_RATIO` | Defines the core shape                 |
| `FP_SIZING`       | Determines the floorplan sizing method |
| `DIE_AREA`        | Specifies the die dimensions           |
| `FP_IO_HMETAL`    | Defines the horizontal I/O metal layer |
| `FP_IO_VMETAL`    | Defines the vertical I/O metal layer   |
| `FP_IO_MODE`      | Controls I/O placement configuration   |
| `FP_PDN_VPITCH`   | Controls vertical power-grid pitch     |
| `FP_PDN_HPITCH`   | Controls horizontal power-grid pitch   |

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/4fdb5cb8-36f7-449d-9fd2-e2ae14437285" />


---

# 6. PicoRV32A OpenLane Configuration

The OpenLane configuration defines the basic information required to process the PicoRV32A design.

A typical configuration contains the following settings:

```tcl
set ::env(DESIGN_NAME) "picorv32a"

set ::env(VERILOG_FILES) \
"./designs/picorv32a/src/picorv32a.v"

set ::env(SDC_FILE) \
"./designs/picorv32a/src/picorv32a.sdc"

set ::env(CLOCK_PERIOD) "5.000"

set ::env(CLOCK_PORT) "clk"

set ::env(CLOCK_NET) $::env(CLOCK_PORT)
```

These parameters specify:

* Design name
* RTL source file
* Timing constraint file
* Clock period
* Clock input port
* Clock network

Correct configuration is essential because these values influence synthesis, timing analysis, floorplanning, and placement.

<img width="955" height="970" alt="design config tcl" src="https://github.com/user-attachments/assets/388781a7-b5c5-449a-85f4-9f95cc247a11" />

---

# 7. Pre-Placed Cells

Not every cell or block is suitable for completely automatic placement.

Some important blocks may need to be assigned a fixed physical location before standard-cell placement begins.

These are referred to as **pre-placed cells or blocks**.

Examples include:

* Memory blocks
* Large multiplexers
* Comparators
* Clock-related cells
* Clock-gating structures
* Large IP blocks

Once these locations are established, the placement engine arranges the remaining standard cells around them.

Proper pre-placement can help prevent congestion and ensure that critical blocks remain in suitable physical locations.

---

# 8. Power Distribution Network

A functional chip requires a reliable power delivery network.

The primary power signals are:

```text
VDD → Supply Voltage
VSS → Ground
```

The power distribution network generally consists of structures such as:

```text
VDD / VSS
    │
    ▼
Power Rings
    │
    ▼
Power Straps
    │
    ▼
Standard Cells
```

The objective is to deliver power throughout the chip with minimum voltage drop and noise.

A properly designed power network helps reduce issues such as:

* IR drop
* Ground bounce
* Supply noise
* Local voltage fluctuations

Power planning therefore plays an important role in maintaining reliable circuit operation.
<img width="320" height="240" alt="image" src="https://github.com/user-attachments/assets/9e64186c-dea3-4fc4-bf88-43f7d4d20a50" />


---

# 9. Decoupling Capacitors

**Decoupling capacitors**, commonly called **decap cells**, help stabilize the local power supply.

When a large number of cells switch simultaneously, the instantaneous current demand can increase significantly. This can cause temporary fluctuations in the supply voltage.

A decoupling capacitor stores charge and can provide it locally when required.

```text
          VDD
           │
           ├──── Decoupling Capacitor
           │
        Circuit
           │
          VSS
```

The use of decap cells helps reduce local supply noise and improves power integrity.

---

# 10. I/O Pin Placement

The physical location of input and output pins is determined during the floorplanning stage.

Pins can be distributed around the boundaries of the chip:

```text
          TOP
    ┌───────────────┐
LEFT│               │RIGHT
    │     CORE      │
    │               │
    └───────────────┘
         BOTTOM
```

Pin placement should consider:

* Connectivity
* Routing distance
* Congestion
* Clock requirements
* Signal direction
* Physical design constraints

Clock-related pins require additional attention because clock paths have a major impact on timing.

---

# 11. Placement Blockages

Certain regions of the core may need to be reserved for specific purposes.

A **placement blockage** prevents standard cells from being placed in a selected area.

```text
+----------------------------+
|     Standard Cells         |
|                            |
|      ┌─────────────┐       |
|      │   BLOCKED   │       |
|      │    REGION   │       |
|      └─────────────┘       |
|                            |
|     Standard Cells         |
+----------------------------+
```

Placement blockages can be used around:

* Pre-placed IPs
* Power structures
* Special physical regions
* Routing-sensitive areas

This provides greater control over the physical implementation.

---

# 12. Standard-Cell Placement

After floorplanning, power planning, pin placement, and block placement, the standard cells from the synthesized netlist are physically positioned inside the core.

The placement engine attempts to find locations that satisfy physical and timing requirements.

The general placement process can be represented as:

```text
Gate-Level Netlist
        │
        ▼
Global Placement
        │
        ▼
Detailed Placement
        │
        ▼
Placement Optimization
```

The placement process attempts to:

* Minimize wire length
* Reduce congestion
* Improve timing
* Maintain legal cell locations
* Keep strongly connected cells physically close
* Improve routability



---

# 13. Placement Optimization

Initial placement is usually followed by optimization.

The tool analyzes the physical implementation and attempts to improve important metrics such as:

* Timing
* Wire length
* Congestion
* Capacitance
* Signal delay
* Cell locations

If a signal path is too long, additional buffers may be inserted.

## Repeaters / Buffers

A repeater is typically a buffer inserted along a long interconnect to improve signal propagation.

```text
Source Cell ───── Buffer ───── Destination Cell
```

Repeaters help maintain signal strength and reduce the delay associated with long interconnects.

Placement optimization therefore helps prepare the design for the subsequent routing and timing stages.

---

# 14. Placement Statistics

Once placement is completed, physical design tools provide several statistics that help evaluate the quality of the implementation.

For the PicoRV32A placement stage, the observed values include:

```text
Total Instances      : 21699
Fixed Instances      : 6354
Nets                 : 15449
Design Area          : 420473.3 um²
Utilization          : 36%
Utilization Padded   : 55%
Rows                 : 238
```

These values provide a quick overview of the physical implementation.

### Important Metrics

**Design Area**

Indicates the physical area occupied by the design.

**Number of Instances**

Represents the number of cells or instances present in the design.

**Number of Nets**

Indicates the number of electrical connections between cells.

**Utilization**

Shows how efficiently the available core area is being used.

**Wire Length**

Provides an estimate of the total interconnect length.

**Placement Displacement**

Indicates how far cells move during placement and optimization.


---

# 15. Floorplan vs Placement

The difference between floorplanning and placement can be summarized as follows:

| Floorplanning                            | Placement                                  |
| ---------------------------------------- | ------------------------------------------ |
| Defines the physical framework           | Places standard cells inside the framework |
| Determines core and die dimensions       | Determines individual cell locations       |
| Establishes utilization and aspect ratio | Optimizes cell arrangement                 |
| Defines I/O locations                    | Reduces wire length and congestion         |
| Plans power structures                   | Improves timing and routability            |
| Handles large/pre-placed blocks          | Handles standard-cell distribution         |

Both stages are closely connected. A poor floorplan can make placement and routing significantly more difficult.

---

# 16. Results

## Floorplan

The floorplanning stage established the physical boundaries of the PicoRV32A design, including the core and die regions.



## Placement

After floorplanning, the standard cells were distributed within the available core area.



The placement stage provided useful physical information such as instance count, utilization, area, and number of nets.

---

# 17. Key Takeaways

This module provided practical understanding of the transition from a synthesized netlist to an initial physical implementation.

The major concepts learned were:

```text
Core and Die
      ↓
Aspect Ratio
      ↓
Core Utilization
      ↓
Floorplanning
      ↓
Floorplan Configuration
      ↓
Pre-Placed Cells
      ↓
Power Planning
      ↓
Decoupling Capacitors
      ↓
I/O Pin Placement
      ↓
Placement Blockages
      ↓
Standard-Cell Placement
      ↓
Placement Optimization
      ↓
Placement Statistics
```

The most important takeaway is that physical design is not simply about placing cells inside a chip. **Area, power, timing, congestion, routing, and signal connectivity must all be considered together.**

---

# 18. Physical Design Progress

The overall design flow covered so far can be represented as:

```text
RTL
 │
 ▼
Synthesis
 │
 ▼
Gate-Level Netlist
 │
 ▼
Floorplanning
 │
 ▼
Power Planning
 │
 ▼
Pin Placement
 │
 ▼
Placement
 │
 ▼
Placement Optimization
```

The next major stages of the physical design flow are:

```text
Clock Tree Synthesis (CTS)
        ↓
Routing
        ↓
Parasitic Extraction
        ↓
Static Timing Analysis
        ↓
Physical Verification
        ↓
Final Sign-Off
```

---

## 🏁 Conclusion

The **PicoRV32A floorplanning and placement stage** provided a practical understanding of how a synthesized digital design begins its transformation into a physical chip layout.

By working through core and die sizing, utilization, floorplanning, power distribution, pin placement, placement blockages, standard-cell placement, and optimization, the relationship between the logical netlist and its physical implementation becomes clear.

The placement results and statistics also demonstrate how physical design tools evaluate **area, utilization, connectivity, and placement quality** before moving toward clock-tree synthesis and routing.

Overall, this stage forms an essential foundation for the remaining **physical design and sign-off flow**.
