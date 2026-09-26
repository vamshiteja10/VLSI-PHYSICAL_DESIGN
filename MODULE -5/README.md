# Module -5 🛣️ Routing, Power Distribution & TritonRoute

<p align="center">
  <b>Routing • Power Planning • Global Routing • Detailed Routing • TritonRoute • Physical Verification</b>
</p>

<p align="center">
  A practical study of routing and the final physical-design stages in the SKY130 RTL-to-GDSII flow.
</p>

---

## 📚 Table of Contents

1. [🛣️ Introduction to Routing](#1-️-introduction-to-routing)
2. [🧩 Routing Constraints](#2--routing-constraints)
3. [🧭 Maze Routing](#3--maze-routing)
4. [🔍 Lee's Routing Algorithm](#4--lees-routing-algorithm)
5. [📐 Design Rule Checking](#5--design-rule-checking)
6. [⚡ Power Distribution Network](#6-️-power-distribution-network)
7. [🔋 Power Straps](#7--power-straps)
8. [🌐 Global Routing](#8--global-routing)
9. [🎯 Detailed Routing](#9--detailed-routing)
10. [🔄 Global Routing vs Detailed Routing](#10--global-routing-vs-detailed-routing)
11. [🛠️ TritonRoute](#11-️-tritonroute)
12. [🚀 TritonRoute and Detailed Routing](#12--tritonroute-and-detailed-routing)
13. [🧠 Routing Algorithms and Topology](#13--routing-algorithms-and-topology)
14. [💻 Routing and Verification Commands](#14--routing-and-verification-commands)
15. [🔄 Complete Physical Design Flow](#15--complete-physical-design-flow)
16. [📊 Routing Quick Reference](#16--routing-quick-reference)
17. [🛠️ Tools Used](#17-️-tools-used)
18. [🎯 Key Learnings](#18--key-learnings)
19. [🏁 Conclusion](#19--conclusion)

---

# 1. 🛣️ Introduction to Routing

Routing is one of the major stages near the end of the physical-design process.

After synthesis, floorplanning, placement and clock-tree synthesis, the cells have physical locations. However, the connections between those cells still need to be implemented using metal layers and vias.

Routing creates these physical connections.

### 🔹 Physical Design Progression

```text
RTL
 │
 ▼
Synthesis
 │
 ▼
Floorplan
 │
 ▼
Placement
 │
 ▼
Clock Tree Synthesis
 │
 ▼
Routing
 │
 ▼
Physical Verification
 │
 ▼
Final Layout
```

### 🔹 What Routing Does

Suppose two cells need to communicate:

```text
Cell A ─────────────────── Cell B
          Metal Wire
```

The router determines how that connection should physically travel through the chip.

For larger designs, thousands or millions of connections may need to be routed.

### 🔹 Routing Uses

Routing establishes connections between:

* Standard cells
* Macros
* Input/output pins
* Clock elements
* Power structures
* Other physical components

### 💡 Key Idea

> **Routing transforms logical connectivity into actual physical interconnects on the chip.**

---

# 2. 🧩 Routing Constraints

Routing cannot simply draw wires between two points.

The physical layout contains many restrictions that the router must obey.

### 🔹 Major Constraints

A routing engine needs to consider:

* Available metal layers
* Routing tracks
* Metal width
* Metal spacing
* Via rules
* Existing connections
* Cell and macro locations
* Routing congestion
* Obstacles
* Technology design rules

A simplified routing problem looks like:

```text
              Destination
                   ●
                   │
       ┌───────────┼───────────┐
       │           │           │
       │     ███████████       │
       │     █ Obstacle █       │
       │     ███████████       │
       │                       │
       └───────────────────────┘
                   │
                   ●
                Source
```

The router must find a path around obstacles without creating illegal geometry.

### 🔹 Routing Objectives

A practical routing solution should provide:

```text
Connectivity
     +
Legal Geometry
     +
Available Resources
     +
Low Congestion
     +
Technology Rules
```

### 💡 Key Idea

> **A successful route must be electrically connected and physically legal at the same time.**

---

# 3. 🧭 Maze Routing

Maze routing treats the routing area as a search space.

The available area can be divided into grid locations, and the routing algorithm searches those locations for a valid path.

### 🔹 Example Grid

```text
┌───┬───┬───┬───┬───┐
│ S │   │   │   │   │
├───┼───┼───┼───┼───┤
│   │ █ │ █ │   │   │
├───┼───┼───┼───┼───┤
│   │   │ █ │   │   │
├───┼───┼───┼───┼───┤
│   │   │   │   │ D │
└───┴───┴───┴───┴───┘

S → Source
D → Destination
█ → Blocked location
```

The algorithm explores available positions until it reaches the destination.

### 🔹 General Process

```text
Start Point
     │
     ▼
Search Neighbouring Locations
     │
     ▼
Avoid Obstacles
     │
     ▼
Continue Searching
     │
     ▼
Reach Destination
     │
     ▼
Recover the Route
```

Maze routing provides a simple way to understand the basic problem faced by routing algorithms.

### 💡 Key Idea

> **Maze routing searches a constrained physical space to find a valid connection between two points.**

---

# 4. 🔍 Lee's Routing Algorithm

**Lee's algorithm** is a classical algorithm used to explain maze-routing techniques.

It works by exploring the routing grid in an organized manner.

The search begins at the source and expands into neighbouring locations.

### 🔹 Basic Procedure

```text
Source
  │
  ▼
Assign Initial Distance
  │
  ▼
Expand to Neighbours
  │
  ▼
Continue Wave Expansion
  │
  ▼
Reach Destination
  │
  ▼
Backtrace the Route
```

### 🔹 Simplified Example

```text
┌───┬───┬───┬───┐
│ 0 │ 1 │ 2 │ 3 │
├───┼───┼───┼───┤
│ 1 │ █ │ 3 │ 4 │
├───┼───┼───┼───┤
│ 2 │ 3 │ 4 │ 5 │
├───┼───┼───┼───┤
│ 3 │ 4 │ 5 │ D │
└───┴───┴───┴───┘
```

The numbers represent the distance from the starting point.

### 🔹 Characteristics

Lee's algorithm:

* Uses a grid representation
* Expands through neighbouring cells
* Avoids blocked positions
* Searches systematically
* Can recover a path after reaching the destination

### 🔹 Advantages

* Simple to understand
* Guaranteed to find a path if one exists in the modeled grid
* Useful for demonstrating maze-routing concepts

### 🔹 Limitation

For very large routing spaces, exhaustive grid searching can require significant memory and computation.

### 💡 Key Idea

> **Lee's algorithm demonstrates how a routing path can be discovered by systematic exploration of a grid.**

---

# 5. 📐 Design Rule Checking

Once physical geometry is created, it must satisfy the manufacturing rules of the selected technology.

This is the purpose of **Design Rule Checking (DRC)**.

For a SKY130-based design, the generated layout must follow the rules defined by the technology.

### 🔹 Examples of Design Rules

DRC can check conditions such as:

* Minimum metal width
* Minimum metal spacing
* Via dimensions
* Via enclosure
* Layer spacing
* Minimum feature dimensions
* Other manufacturing constraints

### 🔹 Simplified Example

```text
Metal A
══════════════════

       Required
       Spacing

Metal B
══════════════════
```

If the two structures are too close:

```text
❌ DRC Violation
```

If the required spacing is satisfied:

```text
✅ DRC Clean
```

### 🔹 Why DRC Matters

A layout can be logically correct and still be physically invalid.

For example:

```text
Logical Connection
       ↓
Correct

Physical Geometry
       ↓
Rule Violation
       ↓
❌ Layout Not Acceptable
```

DRC helps identify such physical problems.

### 💡 Key Idea

> **DRC verifies whether the layout geometry follows the manufacturing rules of the selected technology.**

---

# 6. ⚡ Power Distribution Network

Every standard cell and physical block requires power.

Therefore, power cannot be supplied through ordinary signal-routing connections alone.

A dedicated **Power Distribution Network (PDN)** is created to distribute power and ground throughout the chip.

### 🔹 Basic PDN

```text
                    VDD
                     │
        ═════════════╪═════════════
                     │
             ┌───────┼───────┐
             │       │       │
             │     Cells     │
             │       │       │
             └───────┼───────┘
                     │
        ═════════════╪═════════════
                     │
                    VSS
```

The PDN generally distributes:

* VDD
* VSS / GND

### 🔹 PDN Requirements

A good power network should provide:

* Reliable power delivery
* Low-resistance paths
* Sufficient current-carrying capability
* Good coverage across the chip
* Connections to standard-cell power rails

### 🔹 Power Flow

```text
Power Source
     │
     ▼
Power Grid
     │
     ▼
Power Straps
     │
     ▼
Standard-Cell Rails
     │
     ▼
Individual Cells
```

### 💡 Key Idea

> **The PDN provides the physical infrastructure required to deliver power and ground throughout the chip.**

---

# 7. 🔋 Power Straps

Power straps are relatively wide metal structures used to carry VDD and VSS across different regions of the layout.

A simplified power grid can be visualized as:

```text
VDD
══════════════════════════════════
    │       │       │       │
    │       │       │       │
    │       │       │       │
══════════════════════════════════
VSS
```

The straps are connected to other portions of the power network to provide broad chip-level coverage.

### 🔹 Functions of Power Straps

Power straps help to:

* Distribute power across the design
* Connect different regions
* Reduce resistance in power paths
* Provide current to standard-cell regions
* Strengthen the overall PDN

### 🔹 Simplified Hierarchy

```text
VDD / VSS Source
       │
       ▼
Main Power Grid
       │
       ▼
Power Straps
       │
       ▼
Local Power Rails
       │
       ▼
Standard Cells
```

### 💡 Key Idea

> **Power straps provide strong physical paths for distributing power across the chip.**

---

# 8. 🌐 Global Routing

Global routing is responsible for deciding the general path that each connection should take.

At this stage, the router focuses more on planning than on producing every final wire segment.

### 🔹 Global Routing Concept

```text
Source
  │
  ▼
┌───────────────────────┐
│                       │
│   ──────────────┐     │
│                 │     │
│                 └─────┼──► Destination
│                       │
└───────────────────────┘
```

The design is divided into routing regions, and the router determines how nets should pass through these regions.

### 🔹 Global Routing Considers

* Routing resources
* Congestion
* Available layers
* General path direction
* Connectivity
* Routing demand

### 🔹 Main Objective

The purpose is to create a routing plan that can later be converted into exact physical wires.

### 💡 Key Idea

> **Global routing answers the question: “Which general path should this connection follow?”**

---

# 9. 🎯 Detailed Routing

Detailed routing takes the global-routing information and converts it into actual physical connections.

At this stage, the router works with specific:

* Metal tracks
* Wire segments
* Via locations
* Routing layers
* Design-rule restrictions

### 🔹 Detailed Routing Process

```text
Global Routing
      │
      ▼
Routing Tracks
      │
      ▼
Metal Segments
      │
      ▼
Vias
      │
      ▼
Physical Net
```

The final route must satisfy the technology rules and maintain electrical connectivity.

### 🔹 Detailed Routing Must Handle

* Exact track assignment
* Metal layer selection
* Via insertion
* Wire geometry
* Obstacles
* Design rules
* Connectivity

### 💡 Key Idea

> **Detailed routing turns the routing plan into actual legal metal and via structures.**

---

# 10. 🔄 Global Routing vs Detailed Routing

Global and detailed routing are closely related, but they have different responsibilities.

| Feature      | Global Routing      | Detailed Routing       |
| :----------- | :------------------ | :--------------------- |
| Main Goal    | Route planning      | Exact implementation   |
| Path         | Approximate         | Precise                |
| Tracks       | Estimated resources | Specific tracks        |
| Wires        | Not finalized       | Created                |
| Vias         | Not finalized       | Inserted               |
| Congestion   | Major consideration | Managed during routing |
| Design Rules | Considered          | Strictly enforced      |
| Output       | Routing plan        | Physical interconnect  |

### 🔹 Routing Progression

```text
               Routing
                  │
          ┌───────┴───────┐
          ▼               ▼
   Global Routing   Detailed Routing
          │               │
          ▼               ▼
    Path Planning    Exact Geometry
          │               │
          └───────┬───────┘
                  ▼
             Routed Layout
```

### 💡 Key Idea

> **Global routing decides the route direction and regions, while detailed routing creates the exact physical implementation.**

---

# 11. 🛠️ TritonRoute

**TritonRoute** is a detailed-routing engine associated with the OpenROAD physical-design flow.

It operates after earlier physical-design stages have established cell placement, clock infrastructure and routing information.

### 🔹 Simplified Flow

```text
Placement
    │
    ▼
Clock Tree Synthesis
    │
    ▼
Global Routing
    │
    ▼
TritonRoute
    │
    ▼
Detailed Routing
    │
    ▼
Physical Verification
```

TritonRoute works with the physical information generated during the earlier stages and creates detailed interconnections.

### 🔹 Routing Information

The detailed router needs information such as:

* Net connectivity
* Routing resources
* Metal layers
* Obstacles
* Routing constraints
* Technology rules

### 💡 Key Idea

> **TritonRoute is responsible for creating detailed physical routes while respecting the available routing resources and constraints.**

---

# 12. 🚀 TritonRoute and Detailed Routing

Detailed routing is more than simply finding any connection between two pins.

The generated route must satisfy multiple conditions simultaneously.

```text
        Connectivity
             +
       Routing Resources
             +
        Metal Layers
             +
        Design Rules
             +
          Obstacles
             ↓
      Legal Physical Route
```

### 🔹 Important Routing Requirements

The router must consider:

* Correct net connectivity
* Available routing tracks
* Metal-layer restrictions
* Via placement
* Existing routes
* Blockages
* Design rules
* Routing congestion

### 🔹 Routing Goal

A successful detailed-routing stage should produce:

```text
Connected Nets
      +
Legal Geometry
      +
Valid Vias
      +
Technology Compliance
      ↓
Routed Design
```

### 💡 Key Idea

> **Detailed routing is a constrained optimization problem where connectivity and physical legality must be achieved together.**

---

# 13. 🧠 Routing Algorithms and Topology

Routing algorithms determine how nets should physically travel from their source pins to their destination pins.

For multi-pin nets, the router also needs to determine a suitable topology.

### 🔹 Example Multi-Pin Connection

```text
                 Pin A
                   │
                   │
Pin B ─────────────┼──────────── Pin C
                   │
                   │
                 Pin D
```

The router must create a structure that connects all required pins.

### 🔹 Routing Decision Process

```text
Identify Connections
        │
        ▼
Check Available Resources
        │
        ▼
Find Suitable Path
        │
        ▼
Avoid Obstacles
        │
        ▼
Select Layers / Tracks
        │
        ▼
Insert Required Vias
        │
        ▼
Check Design Rules
        │
        ▼
Final Route
```

### 🔹 Routing Objectives

A routing algorithm attempts to achieve:

* Complete connectivity
* Low congestion
* Efficient resource usage
* Legal geometry
* Suitable topology
* Reduced routing conflicts

### 💡 Key Idea

> **Routing topology defines how multiple pins are connected, while the routing algorithm determines how that topology is physically realized.**

---

# 14. 💻 Routing and Verification Commands

Several commands can be used during the routing and physical-verification stages.

---

## 🔹 Run Routing

```tcl
run_routing
```

This starts the routing stage.

---

## 🔹 Generate Power Grid

```tcl
run_power_grid_generation
```

This generates the required power-distribution structures.

---

## 🔹 Antenna Check

```tcl
run_antenna_check
```

This checks for antenna-related issues in the routed layout.

---

## 🔹 Magic DRC

```tcl
run_magic_drc
```

This performs design-rule checking using Magic.

---

## 🔹 KLayout DRC

```tcl
run_klayout_drc
```

This performs DRC using KLayout.

---

## 🔹 LVS

```tcl
run_lvs
```

This performs Layout Versus Schematic/netlist checking.

---

## 🔹 Typical Physical-Design Sequence

```text
run_synthesis
       │
       ▼
run_floorplan
       │
       ▼
run_placement
       │
       ▼
run_cts
       │
       ▼
run_routing
       │
       ▼
Physical Verification
```

---

## 🔹 Important Outputs

| Output             | Purpose                                  |
| :----------------- | :--------------------------------------- |
| **Routed Layout**  | Shows physical net connections           |
| **Routing Report** | Provides routing information             |
| **DRC Report**     | Lists physical design-rule violations    |
| **LVS Report**     | Checks layout against the design netlist |
| **Antenna Report** | Identifies antenna-related violations    |

---

# 15. 🔄 Complete Physical Design Flow

The routing stage is part of a larger RTL-to-GDSII process.

```text
                    RTL
                     │
                     ▼
                 Synthesis
                     │
                     ▼
                Floorplanning
                     │
                     ▼
                 Placement
                     │
                     ▼
          Clock Tree Synthesis
                     │
                     ▼
             Power Distribution
                     │
                     ▼
              Global Routing
                     │
                     ▼
             Detailed Routing
                     │
                     ▼
               TritonRoute
                     │
                     ▼
              Routed Layout
                     │
             ┌───────┼───────┐
             ▼       ▼       ▼
            DRC     LVS    Antenna
             │       │       │
             └───────┼───────┘
                     ▼
           Physical Verification
                     │
                     ▼
                Final Layout
```

### 🔹 Stage-by-Stage View

| Stage                | Main Responsibility                    |
| :------------------- | :------------------------------------- |
| **Synthesis**        | Converts RTL into a gate-level netlist |
| **Floorplanning**    | Defines the physical chip organization |
| **Placement**        | Positions standard cells               |
| **CTS**              | Builds the clock network               |
| **PDN**              | Creates power and ground distribution  |
| **Global Routing**   | Plans general routing paths            |
| **Detailed Routing** | Creates exact physical connections     |
| **TritonRoute**      | Performs detailed routing              |
| **DRC**              | Checks physical design rules           |
| **LVS**              | Verifies layout/netlist consistency    |
| **Antenna Check**    | Checks antenna-related issues          |

---

# 16. 📊 Routing Quick Reference

## 🔹 Routing Stages

| Stage                | Purpose                                        |
| :------------------- | :--------------------------------------------- |
| **Global Routing**   | Determines general paths                       |
| **Detailed Routing** | Creates exact wires and vias                   |
| **TritonRoute**      | Performs detailed routing                      |
| **DRC**              | Verifies layout rules                          |
| **LVS**              | Compares layout with the design representation |
| **Antenna Check**    | Detects antenna-related problems               |

---

## 🔹 Power Network

| Component        | Function                           |
| :--------------- | :--------------------------------- |
| **VDD**          | Positive supply                    |
| **VSS**          | Ground / return supply             |
| **Power Grid**   | Main power-distribution structure  |
| **Power Straps** | Wide metal paths across the design |
| **Local Rails**  | Supply standard-cell regions       |

---

## 🔹 Routing Concepts

| Concept           | Meaning                                 |
| :---------------- | :-------------------------------------- |
| **Routing Track** | Available physical path for metal       |
| **Via**           | Connection between metal layers         |
| **Obstacle**      | Region that restricts routing           |
| **Congestion**    | High routing demand in an area          |
| **Net**           | Electrical connection between pins      |
| **Topology**      | Structure used to connect multiple pins |

---

# 17. 🛠️ Tools Used

| Tool            | Main Purpose                             |
| :-------------- | :--------------------------------------- |
| **OpenLane**    | Automated RTL-to-GDSII flow              |
| **OpenROAD**    | Physical-design implementation           |
| **TritonRoute** | Detailed routing                         |
| **Yosys**       | RTL synthesis                            |
| **OpenSTA**     | Static Timing Analysis                   |
| **Magic**       | Layout inspection and verification       |
| **KLayout**     | Layout viewing and DRC                   |
| **SKY130 PDK**  | Technology and manufacturing information |
| **Linux**       | VLSI development environment             |

---

# 18. 🎯 Key Learnings

### 🛣️ Routing

* Routing converts logical connections into physical metal connections.
* Routing must work within limited physical resources.
* Metal layers, tracks, vias and obstacles affect routing decisions.
* Routing congestion can make physical implementation difficult.

### 🧭 Routing Algorithms

* Maze routing searches for paths through constrained regions.
* Lee's algorithm provides a classical grid-based routing approach.
* Routing topology defines how multiple pins are connected.
* Routing algorithms must balance connectivity with physical constraints.

### ⚡ Power Distribution

* The PDN distributes VDD and VSS throughout the design.
* Power straps provide wide, low-resistance paths for power distribution.
* Reliable power delivery is essential for standard-cell operation.

### 🌐 Global and Detailed Routing

* Global routing creates a high-level route plan.
* Detailed routing implements the exact physical wires.
* Detailed routing assigns tracks and vias.
* Final routes must obey technology-specific rules.

### 🛠️ TritonRoute

* TritonRoute is used for detailed routing in the OpenROAD ecosystem.
* It works with routing resources and physical constraints.
* It produces the detailed interconnect required for the physical layout.

### 🔍 Physical Verification

* DRC checks manufacturing-related geometry rules.
* LVS checks consistency between layout and the design representation.
* Antenna checking identifies antenna-related routing issues.
* Verification is required before considering the layout physically complete.

---

# 19. 🏁 Conclusion

Module 5 focuses on the transition from a placed physical design to a fully connected routed layout.

The process begins with understanding how routing works and how algorithms search for legal paths. It then moves into power distribution, global routing and detailed routing.

Finally, TritonRoute and physical-verification tools are used as part of the final implementation flow.

The complete concept can be summarized as:

```text
                  PHYSICAL DESIGN
                         │
                         ▼
                     Placement
                         │
                         ▼
                        CTS
                         │
                         ▼
                 Power Distribution
                         │
                         ▼
                  Global Routing
                         │
                         ▼
                 Detailed Routing
                         │
                         ▼
                   TritonRoute
                         │
                         ▼
                  Routed Layout
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
             DRC        LVS       Antenna
              │          │          │
              └──────────┼──────────┘
                         ▼
               Physical Verification
                         │
                         ▼
                    Final Layout
```

The key transition is:

```text
Logical Connectivity
        │
        ▼
Routing Planning
        │
        ▼
Exact Physical Wires
        │
        ▼
Power + Signal Connectivity
        │
        ▼
Physical Verification
        │
        ▼
Final Layout
```

A successful routing stage must satisfy both connectivity and physical requirements:

```text
Electrical Connectivity
          +
Routing Resources
          +
Design Rules
          +
Power Distribution
          +
Physical Constraints
          │
          ▼
      Valid Layout
```

> **Routing is the stage where the logical design finally becomes a physically connected structure of metal layers, vias and power networks.**

---

## 📌 Final Takeaway

```text
       ROUTING
          │
   ┌──────┼──────┐
   ▼      ▼      ▼
Global  Detailed  PDN
Route    Route
   │       │       │
   └───────┼───────┘
           ▼
       TritonRoute
           │
           ▼
    Routed Layout
           │
           ▼
   DRC + LVS + Antenna
           │
           ▼
      Final Design
```

**Module 5 — Routing, Power Distribution & TritonRoute completed.**
