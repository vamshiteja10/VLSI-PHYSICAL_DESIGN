# OpenLane Physical Design – PicoRV32A

A hands-on learning project that explores the complete **RTL-to-GDSII physical design flow** using the **PicoRV32A RISC-V processor**, **OpenLane**, and the **Sky130 PDK**.

The project walks through the major stages of ASIC physical design, including synthesis, floorplanning, power planning, placement, clock tree synthesis, routing, static timing analysis, and physical verification.

---

## 1. Project Overview

ASIC physical design is the process of converting a digital design described in RTL into a physical layout that can eventually be manufactured as a chip.

In this project, **PicoRV32A**, a small RISC-V processor, is used as the design to understand how an RTL description progresses through the complete physical design flow.

### RTL-to-GDSII Flow

```text
RTL
 ↓
Synthesis
 ↓
Floorplanning
 ↓
Power Planning
 ↓
Placement
 ↓
Clock Tree Synthesis
 ↓
Routing
 ↓
Static Timing Analysis
 ↓
Physical Verification
 ↓
Signoff
 ↓
GDSII
```

The main objective is to understand what happens at each stage and how the different open-source tools work together to transform RTL into a physical chip layout.

---

## 2. PicoRV32A

**PicoRV32A** is a compact **RISC-V processor** implemented using RTL.

RTL, or Register Transfer Level, describes the behavior and structure of a digital circuit before it is converted into physical hardware.

The design includes elements such as:

* Logic gates
* Flip-flops
* Multiplexers
* Registers
* Control logic

The RTL description is used as the starting point for synthesis.

<img width="955" height="550" alt="image" src="https://github.com/user-attachments/assets/44e64471-9287-4cba-a9f1-d4b128a731f1" />


---

## 3. Sky130 PDK

**PDK** stands for **Process Design Kit**.

A PDK provides the technology-specific information required to design and verify an integrated circuit. It defines the characteristics and rules that physical design tools need to create a manufacturable layout.

The PDK provides information such as:

* Standard-cell libraries
* Technology layers
* Design rules
* Timing information
* Physical characteristics

This project uses the **Sky130 PDK** as the target technology.

---

## 4. OpenLane

**OpenLane** is an open-source RTL-to-GDSII implementation flow.

It integrates several open-source tools and automates many of the steps required to take an RTL design through synthesis and physical implementation.

### Basic OpenLane Flow

```text
RTL + PDK
   ↓
Synthesis
   ↓
Floorplanning
   ↓
Placement
   ↓
Clock Tree Synthesis
   ↓
Routing
   ↓
Signoff
   ↓
GDSII
```

Using OpenLane makes it possible to study an ASIC physical design flow using open-source tools and the Sky130 technology.

---
<img width="1500" height="1200" alt="image" src="https://github.com/user-attachments/assets/e9c82e40-5fa2-4c28-b25c-bd83da0babf8" />


## 5. Synthesis

Synthesis is the stage where the **RTL description is converted into a gate-level netlist**.

The synthesis tool analyzes the RTL and maps its logic into standard cells available in the target technology library.

```text
RTL
 ↓
Synthesis
 ↓
Gate-Level Netlist
```

The resulting netlist can contain cells such as:

* AND gates
* OR gates
* NAND gates
* NOR gates
* Buffers
* Inverters
* Multiplexers
* Flip-flops

### Synthesis Result

The synthesis stage produces the gate-level representation that is used by the following physical design stages.
<img width="1920" height="983" alt="printing statistics openlane" src="https://github.com/user-attachments/assets/176412a5-ec65-48d1-987f-9f4fb8f4f4eb" />


---

## 6. Gate-Level Netlist

A **netlist** describes the cells used in a digital circuit and the connections between those cells.

After synthesis, the original RTL is represented using standard cells from the target technology library.

```text
RTL
 ↓
Standard Cells
 ↓
Connections
 ↓
Gate-Level Netlist
```

For the PicoRV32A design, synthesis results in a large number of standard cells connected according to the functionality described by the original RTL.
<img width="1920" height="983" alt="OPENLANE NETLIST" src="https://github.com/user-attachments/assets/6590a386-df5e-4ce6-bd17-4395a0395bf6" />


---

## 7. Floorplanning

**Floorplanning** defines the basic physical structure of the chip.

At this stage, decisions are made about:

* Chip dimensions
* Core dimensions
* I/O pin locations
* Available area for standard cells

A well-planned floorplan is important because it affects later stages such as placement, routing, congestion, and timing.

---

## 8. Power Planning

**Power planning** creates the power distribution network required to supply power throughout the chip.

The power network includes elements such as:

* Power rings
* Power straps
* Power rails
* VDD
* VSS

The goal is to distribute power reliably to the standard cells while providing an appropriate physical power network for the design.

---

## 9. Placement

**Placement** determines the physical locations of the standard cells inside the chip core.

Placement generally involves two major stages.

### Global Placement

Global placement determines approximate locations for the cells while considering factors such as connectivity and congestion.

### Detailed Placement

Detailed placement adjusts the cells so that they occupy legal physical positions according to the technology rules.

Good placement helps to:

* Reduce wire length
* Reduce routing congestion
* Improve timing
* Make routing easier

---

## 10. Clock Tree Synthesis

**CTS**, or **Clock Tree Synthesis**, builds the clock distribution network for sequential elements such as flip-flops.

Clock buffers are inserted to distribute the clock signal across the design while maintaining suitable timing characteristics.

### Main Goal

The primary objective of CTS is to ensure that the clock reaches the required sequential elements at appropriate times.

CTS also attempts to minimize **clock skew**, which is the difference in clock arrival time between different sequential elements.

```text
              Clock
                |
              Buffer
             /  |  \
           FF1  FF2  FF3
```

A properly designed clock network is essential for reliable sequential operation.

---

## 11. Routing

**Routing** creates the physical metal connections between the placed cells.

Routing is generally divided into two stages.

### Global Routing

Global routing determines approximate paths for the required connections.

### Detailed Routing

Detailed routing creates the actual metal-layer connections while following the design rules of the target technology.

The routing stage must satisfy the physical constraints defined by the Sky130 technology.

### RTL-to-GDSII Implementation

```text
RTL
 ↓
Synthesis
 ↓
Floorplanning
 ↓
Power Planning
 ↓
Placement
 ↓
CTS
 ↓
Routing
 ↓
Signoff
 ↓
GDSII
```
<img width="1507" height="681" alt="image" src="https://github.com/user-attachments/assets/ebe55db8-69a7-4a99-a371-befae2feb729" />

---

## 12. Static Timing Analysis

**STA**, or **Static Timing Analysis**, is used to determine whether the design satisfies its timing requirements.

Timing analysis considers factors such as:

* Cell delay
* Net delay
* Clock delay
* Setup time
* Hold time
* Slew

### Slack

**Slack** indicates whether a timing requirement has been met.

```text
Positive Slack → Timing requirement is satisfied

Negative Slack → Timing violation
```

**OpenSTA** is used for timing analysis in this flow.

Timing analysis is important because a design can be logically correct but still fail if signals do not arrive within the required timing limits.
<img width="955" height="970" alt="OPENSTA" src="https://github.com/user-attachments/assets/fee2c69d-0d48-4d2e-8374-208310b2c587" />


---

## 13. Signoff

**Signoff** is the final verification stage before the physical layout is considered ready for final output.

Several checks are performed to ensure that the design is physically and electrically acceptable.

### Design Rule Check — DRC

**DRC** verifies that the physical layout follows the manufacturing rules defined by the technology.

It checks whether the geometry and spacing of layout features meet the required design rules.

### Layout Versus Schematic — LVS

**LVS** checks whether the physical layout corresponds correctly to the intended circuit.

It helps verify that the connectivity represented by the layout matches the circuit being implemented.

### Timing Verification

Timing checks confirm that the implemented design continues to satisfy its required timing constraints.

After the required verification checks are successfully completed, the final physical layout can be represented in **GDSII** format.

---

## 14. OpenLane Commands

The OpenLane flow can be started in interactive mode using:

```bash
./flow.tcl -interactive
```

Some of the commands used during the flow include:

```tcl
package require openlane 0.9
prep -design picorv32a
run_synthesis
```

These commands prepare the design and start the synthesis stage.

> **Note:** OpenLane commands can vary depending on the version being used. Always check the commands supported by your installed OpenLane version.

---

## 15. Results and Learning

The PicoRV32A synthesis run produced the following recorded statistics:

| Parameter        |  Value |
| ---------------- | -----: |
| Total Wires      | 14,596 |
| Wire Bits        | 14,978 |
| Public Wires     |  1,565 |
| Public Wire Bits |  1,947 |
| Memories         |      0 |
| Processes        |      0 |
| Total Cells      | 14,876 |
| Flip-Flops       |  1,613 |

### Flip-Flop Ratio

The flip-flop ratio represents the percentage of flip-flops compared with the total number of cells.

### Formula

```text
Flip-Flop Ratio =
(Flip-Flops / Total Cells) × 100
```

For this design:

```text
(1613 / 14876) × 100
= 10.84%
```

Therefore:

**Flip-Flop Ratio ≈ 10.84%**

---

## 16. Complete Physical Design Flow

The complete flow studied in this project can be summarized as:

```text
             RTL
              ↓
          Synthesis
              ↓
        Floorplanning
              ↓
        Power Planning
              ↓
          Placement
              ↓
     Clock Tree Synthesis
              ↓
           Routing
              ↓
             STA
              ↓
    Physical Verification
              ↓
           Signoff
              ↓
            GDSII
```

This flow demonstrates how a digital design progresses from an RTL description to a physical chip layout.

---

## 17. Tools and Technologies Used

| Tool / Technology | Purpose                           |
| ----------------- | --------------------------------- |
| **PicoRV32A**     | RISC-V processor design           |
| **OpenLane**      | RTL-to-GDSII physical design flow |
| **Yosys**         | RTL synthesis                     |
| **OpenSTA**       | Static timing analysis            |
| **Sky130**        | Technology / PDK                  |
| **GDSII**         | Final physical layout format      |

---

## 18. What I Learned

This project provided practical exposure to the **ASIC physical design flow** and helped me understand how an RTL design is gradually transformed into a physical layout.

The key stages studied were:

* RTL design
* PDK
* Synthesis
* Gate-level netlist
* Floorplanning
* Power planning
* Placement
* Clock Tree Synthesis
* Routing
* Static Timing Analysis
* Physical verification
* Signoff
* GDSII generation

The overall concept can be summarized as:

```text
RTL
 ↓
Netlist
 ↓
Physical Design
 ↓
Verification
 ↓
GDSII
```

Working through the PicoRV32A design helped connect the theoretical concepts of ASIC design with an actual open-source implementation flow.

---

## 19. Conclusion

The project demonstrates the complete **RTL-to-GDSII physical design flow** using the PicoRV32A processor, OpenLane, and the Sky130 PDK.

Starting from RTL, the design passes through synthesis, floorplanning, power planning, placement, clock tree synthesis, routing, timing analysis, and physical verification before reaching the final GDSII representation.

Overall, this project provided a practical understanding of how **open-source EDA tools can be used to take a digital design from RTL to physical layout**.
