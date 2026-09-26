# ⏱️ Timing Analysis & Clock Tree Synthesis

<p align="center">
  <b>Timing Characterization • STA • Clock Distribution • CTS • Signal Integrity</b>
</p>

<p align="center">
  Understanding how cell delay, setup/hold timing, clock networks, skew,
  crosstalk and timing reports come together in the SKY130 physical design flow.
</p>

---

## 📚 Table of Contents

1. [⏱️ Understanding Timing in VLSI](#1-️-understanding-timing-in-vlsi)
2. [📊 Standard Cell Timing Models](#2--standard-cell-timing-models)
3. [⚡ Slew, Load and Cell Delay](#3--slew-load-and-cell-delay)
4. [🕐 Setup Timing](#4--setup-timing)
5. [🔒 Hold Timing](#5--hold-timing)
6. [⚠️ Clock Jitter and Uncertainty](#6-️-clock-jitter-and-uncertainty)
7. [🌳 Clock Tree Synthesis](#7--clock-tree-synthesis)
8. [📐 Clock Skew and Clock Latency](#8--clock-skew-and-clock-latency)
9. [🔊 Crosstalk and Signal Integrity](#9--crosstalk-and-signal-integrity)
10. [🛡️ Clock Shielding](#10-️-clock-shielding)
11. [🔄 Ideal Clock and Propagated Clock](#11--ideal-clock-and-propagated-clock)
12. [🔬 Static Timing Analysis with OpenSTA](#12--static-timing-analysis-with-opensta)
13. [📈 WNS and TNS](#13--wns-and-tns)
14. [💻 OpenLane and OpenROAD Commands](#14--openlane-and-openroad-commands)
15. [🔄 Complete Timing Flow](#15--complete-timing-flow)
16. [📊 Quick Reference Tables](#16--quick-reference-tables)
17. [🛠️ Tools Used](#17-️-tools-used)
18. [🎯 Key Learnings](#18--key-learnings)
19. [🏁 Conclusion](#19--conclusion)

---

# 1. ⏱️ Understanding Timing in VLSI

Timing analysis is one of the most important stages in digital IC design.

A circuit may produce the correct logical output, but it is still not considered reliable if the signals do not arrive within the required time.

A simplified timing path looks like this:

```text
        Launch
       Flip-Flop
           │
           ▼
   ┌─────────────────┐
   │ Combinational   │
   │     Logic       │
   └────────┬────────┘
            │
            ▼
        Capture
       Flip-Flop
```

The data launched from one sequential element must travel through the logic and reach the receiving flip-flop within the available timing window.

### 🔹 Main Timing Questions

Timing analysis essentially answers:

* When does the data arrive?
* When is the data required?
* How much delay does the path have?
* Is there enough timing margin?
* Are setup and hold requirements satisfied?
* How does the physical clock network affect timing?

### 💡 Key Idea

> **Functional correctness tells us whether the circuit works logically, while timing analysis tells us whether it works at the required speed.**

---

# 2. 📊 Standard Cell Timing Models

Standard cells such as:

* AND gates
* OR gates
* NAND gates
* NOR gates
* Inverters
* Buffers
* Flip-flops

have different timing characteristics.

Their delay is not a single fixed number.

Instead, delay changes according to the electrical conditions at the cell input and output.

The most important factors are:

```text
Input Slew
    +
Output Load
    ↓
Cell Delay
```

### 🔹 Timing Characterization

Instead of simulating every transistor of every cell during the complete design flow, cells are characterized beforehand.

The resulting timing information is stored in a technology library.

```text
Transistor-Level Characterization
              │
              ▼
       Timing Measurements
              │
              ▼
       Characterized Cells
              │
              ▼
        Timing Library
              │
              ▼
       STA / Synthesis
```

Timing libraries provide information required by synthesis and timing-analysis tools.

### 🔹 Important Timing Information

A timing library may contain:

* Cell propagation delay
* Transition information
* Input capacitance
* Output capacitance
* Setup time
* Hold time
* Timing constraints
* Power information

### 💡 Key Idea

> **The timing library provides the electrical behaviour of standard cells in a form that EDA tools can use efficiently.**

---

# 3. ⚡ Slew, Load and Cell Delay

Two parameters have a major influence on standard-cell delay:

1. Input slew
2. Output load

---

## 🔹 Input Slew

Input slew represents the transition speed of a signal.

A signal with a sharp transition has a smaller slew time, while a slowly changing signal has a larger slew time.

```text
Fast Transition

Voltage
  │
  │        ┌────────
  │       /
  │      /
  │_____/
  │
  └──────────────────► Time
```

```text
Slow Transition

Voltage
  │
  │             ┌──────
  │          __/
  │       __/
  │_____/
  │
  └──────────────────► Time
```

A poor input transition can increase the delay of the receiving cell.

---

## 🔹 Output Load

Output load is the amount of capacitance that a cell must drive.

```text
              Cell
               │
               │
               ▼
        ┌─────────────┐
        │ Output Load  │
        │      C       │
        └─────────────┘
```

More load generally means that the cell requires more time to charge or discharge the output.

---

## 🔹 Combined Effect

```text
        Input Slew
            │
            ▼
      ┌─────────────┐
      │             │
      │ Timing Cell │
      │             │
      └──────┬──────┘
             ▲
             │
        Output Load
             │
             ▼
        Cell Delay
```

### 🔹 General Relationship

```text
Fast Slew + Small Load
          ↓
      Lower Delay
```

```text
Slow Slew + Large Load
          ↓
      Higher Delay
```

### 📌 Delay Comparison

| Input Transition | Output Load | Expected Delay |
| :--------------- | :---------- | :------------- |
| Fast             | Low         | Low            |
| Fast             | High        | Moderate       |
| Slow             | Low         | Moderate       |
| Slow             | High        | High           |

### 💡 Key Idea

> **Increasing the load or degrading the input transition usually increases cell delay.**

---

# 4. 🕐 Setup Timing

Setup timing deals with the maximum amount of time allowed for data to travel between sequential elements.

Consider:

```text
Launch FF
    │
    ▼
Logic
    │
    ▼
Capture FF
```

The data must reach the capture flip-flop sufficiently early before the active clock edge.

---

## 🔹 Basic Setup Requirement

For an ideal clock:

```text
Data Path Delay + Setup Time
        ≤
Clock Period
```

Therefore:

```text
Maximum Data Delay
=
Clock Period - Setup Time
```

When clock uncertainty is included:

```text
Available Time
=
Clock Period
-
Setup Time
-
Clock Uncertainty
```

---

## 🔹 Setup Slack

Setup slack can be represented as:

```text
Setup Slack
=
Required Time - Arrival Time
```

### Timing interpretation

```text
Positive Slack
      │
      ▼
Setup Requirement Satisfied
```

```text
Negative Slack
      │
      ▼
Setup Violation
```

---

## 🔹 Example

Assume:

```text
Clock Period = 10 ns
Setup Time   = 0.8 ns
Uncertainty  = 0.2 ns
```

Available time:

```text
10 - 0.8 - 0.2
=
9.0 ns
```

Therefore, the data path must complete within the available timing window.

### 💡 Key Idea

> **Setup analysis checks whether data arrives early enough before the capture edge.**

---

# 5. 🔒 Hold Timing

Hold analysis is the minimum-delay check.

After the active clock edge reaches the capture flip-flop, the incoming data must remain stable for the required hold period.

```text
              Hold Window
                   │
                   ▼
Clock ─────────────┼──────────────
                   │
Data  ─────────────┼──────────────
                   │
              Stable Data
```

If new data reaches the capture element too quickly, a hold violation can occur.

---

## 🔹 Basic Hold Requirement

For an ideal clock:

```text
Minimum Data Delay
≥
Hold Time
```

For a real clock network, clock arrival differences also affect the condition.

A simplified relationship is:

```text
Launch Clock Delay + Data Delay
≥
Capture Clock Delay + Hold Time
```

---

## 🔹 Hold Slack

```text
Hold Slack
=
Arrival Time - Required Time
```

### Timing result

```text
Positive Hold Slack
        │
        ▼
   Hold Passed
```

```text
Negative Hold Slack
        │
        ▼
   Hold Violation
```

---

## 🔹 Setup vs Hold

| Parameter           | Setup                | Hold                |
| :------------------ | :------------------- | :------------------ |
| Checks              | Maximum delay        | Minimum delay       |
| Problem             | Data arrives late    | Data arrives early  |
| Important Parameter | Setup time           | Hold time           |
| Timing Window       | Before capture edge  | After capture edge  |
| Violation           | Negative setup slack | Negative hold slack |

### 💡 Key Idea

> **Setup prevents data from arriving too late, while hold prevents data from changing too soon.**

---

# 6. ⚠️ Clock Jitter and Uncertainty

An ideal clock would always arrive exactly where expected.

A real clock can experience variations in its edge position.

This variation is known as **clock jitter**.

```text
Expected Edge
      │
      ▼
──────┼──────────────
   Earlier       Later
       ← Jitter →
```

The clock edge can move slightly earlier or later than its nominal position.

---

## 🔹 Sources of Clock Variation

Clock uncertainty may account for effects such as:

* Clock jitter
* Clock variation
* Modeling margins
* Other clock-related uncertainties

---

## 🔹 Effect on Timing

For setup timing:

```text
Clock Period
      │
      ├── Setup Time
      │
      ├── Clock Uncertainty
      │
      ▼
Available Data Time
```

Increasing uncertainty reduces the usable timing margin.

```text
Clock Uncertainty ↑
        │
        ▼
Available Margin ↓
```

### 💡 Key Idea

> **Clock uncertainty provides a safety margin for variations in clock arrival.**

---

# 7. 🌳 Clock Tree Synthesis

A modern digital design can contain thousands of sequential elements.

Driving all of them directly from a single clock source is inefficient because the clock network would experience:

* Large fanout
* High capacitance
* Large delay
* Unequal arrival times

**Clock Tree Synthesis (CTS)** creates a structured clock distribution network.

---

## 🔹 Basic Clock Tree

```text
                    Clock Source
                         │
                      Buffer
                         │
                  ┌──────┴──────┐
                  │             │
               Buffer        Buffer
                  │             │
              ┌───┴───┐     ┌───┴───┐
              │       │     │       │
             FF1     FF2   FF3     FF4
```

Buffers divide the load and help distribute the clock throughout the design.

---

## 🔹 CTS Objectives

A useful clock tree attempts to control:

* Clock skew
* Clock latency
* Clock transition
* Fanout
* Capacitance
* Clock signal quality

The goal is not simply to connect every flip-flop to the clock source, but to create a controlled physical network.

---

## 🔹 CTS in OpenLane/OpenROAD

The physical-design flow can invoke CTS using:

```tcl
run_cts
```

The CTS stage inserts and organizes clock buffers to create the clock distribution network.

### 💡 Key Idea

> **CTS converts an ideal clock connection into a physical clock distribution network.**

---

# 8. 📐 Clock Skew and Clock Latency

Clock arrival is not identical at every sequential element.

Two important clock parameters are:

* Clock skew
* Clock latency

---

## 🔹 Clock Skew

Clock skew is the difference between clock arrival times at two endpoints.

```text
                Clock Source
                     │
             ┌───────┴───────┐
             │               │
           Path A           Path B
             │               │
            FF1             FF2
             │               │
           tCLK1           tCLK2
```

Therefore:

```text
Clock Skew
=
tCLK2 - tCLK1
```

Ideally, clock arrival should be closely balanced.

---

## 🔹 Clock Latency

Clock latency is the time taken for the clock signal to travel from its source to a sequential element.

It can include:

```text
Clock Buffer Delay
        +
Wire Delay
        +
RC Effects
        ↓
Clock Latency
```

Example:

```text
Clock Source
     │
     ▼
  Buffer
     │
     ▼
   Wire
     │
     ▼
  Buffer
     │
     ▼
 Flip-Flop
```

### 💡 Key Idea

> **Skew describes the difference between clock arrivals, while latency describes the travel time of the clock.**

---

# 9. 🔊 Crosstalk and Signal Integrity

Physical wires are placed close to each other in an integrated circuit.

Because of this proximity, electrical coupling can occur between neighbouring wires.

This effect is commonly called **crosstalk**.

```text
Aggressor
══════════════════════════════

          ⇅
     Coupling Effect
          ⇅

Victim
══════════════════════════════
```

When the aggressor switches, the victim signal may experience unwanted disturbance.

---

## 🔹 Possible Effects

Crosstalk can result in:

* Noise
* Glitches
* Delay variation
* Transition degradation
* Timing changes
* Clock disturbance

---

## 🔹 Crosstalk and Timing

```text
Normal Signal
      │
      ▼
Normal Delay
      │
      +
Coupling Effect
      │
      ▼
Modified Delay
```

This becomes especially important for high-speed and critical nets.

### 🔹 Signal Integrity

Signal integrity refers to maintaining the electrical quality of a signal while it travels through the physical interconnect.

Important factors include:

```text
Coupling
  +
Noise
  +
RC Effects
  +
Transition
  +
Crosstalk
```

### 💡 Key Idea

> **Physical interconnect can affect timing just as much as logic-cell delay.**

---

# 10. 🛡️ Clock Shielding

Clock signals are particularly sensitive because timing of the clock directly affects sequential operation.

A nearby switching wire can couple noise into the clock network.

Clock shielding is used to reduce this unwanted interaction.

---

## 🔹 Simplified Arrangement

```text
        Shield          Clock          Shield
═══════════════      ═══════════      ═══════════════
      GND                CLK                GND
```

A shield is connected to a stable reference such as:

```text
VDD
```

or

```text
GND
```

depending on the routing methodology.

---

## 🔹 Benefits

Clock shielding can help:

* Reduce coupling
* Reduce clock noise
* Limit crosstalk
* Improve clock integrity
* Reduce unwanted timing variation

### 💡 Key Idea

> **Critical nets such as clocks often need special physical-routing techniques.**

---

# 11. 🔄 Ideal Clock and Propagated Clock

Clock behavior changes as the design moves through physical implementation.

Before CTS, timing analysis often uses an **ideal clock**.

After CTS, the actual clock network can be considered using a **propagated clock**.

---

## 🔹 Ideal Clock

An ideal clock assumes that the clock reaches the sequential elements without modeling the physical clock-tree delay.

```text
              CLK
               │
        ┌──────┴──────┐
        │             │
       FF1           FF2
```

This model is useful during early timing analysis.

---

## 🔹 Propagated Clock

After CTS:

```text
              CLK
               │
             Buffer
               │
              Wire
               │
          ┌────┴────┐
       Buffer      Buffer
          │           │
         FF1         FF2
```

Now the clock path can include:

* Buffer delay
* Wire delay
* Clock latency
* Clock skew
* RC effects

---

## 🔹 Comparison

| Feature         |   Ideal Clock  | Propagated Clock |
| :-------------- | :------------: | :--------------: |
| CTS Network     |   Not modeled  |      Modeled     |
| Clock Buffers   |     Ignored    |     Included     |
| Wire Delay      |     Ignored    |     Included     |
| Skew            |    Idealized   |     Physical     |
| Latency         |   Simplified   |     Included     |
| RC Effects      |  Not included  |    Considered    |
| Timing Accuracy | Early estimate |  More realistic  |

### 🔄 Timing Evolution

```text
Ideal Clock
     │
     ▼
Pre-CTS Timing
     │
     ▼
Clock Tree Synthesis
     │
     ▼
Physical Clock Network
     │
     ▼
Propagated Clock
     │
     ▼
Post-CTS Timing
```

### 💡 Key Idea

> **Post-CTS timing gives a more physical representation because the actual clock network contributes to the timing calculation.**

---

# 12. 🔬 Static Timing Analysis with OpenSTA

**Static Timing Analysis (STA)** checks the timing behaviour of a digital design without applying simulation vectors.

OpenSTA analyzes the timing graph and determines whether timing constraints are satisfied.

---

## 🔹 Basic STA Path

```text
Launch Register
      │
      ▼
Logic Gates
      │
      ▼
Interconnect
      │
      ▼
Capture Register
```

The timing engine evaluates the path using information such as:

* Cell delays
* Wire delays
* Clock timing
* Setup requirements
* Hold requirements
* Clock skew
* Timing constraints

---

## 🔹 Arrival Time

Arrival time represents when a signal reaches a particular timing point.

```text
Launch
   │
   ▼
Logic Delay
   │
   ▼
Arrival Time
```

---

## 🔹 Required Time

Required time indicates when the signal must arrive to satisfy the timing constraint.

---

## 🔹 Slack

A simplified timing relationship is:

```text
Slack
=
Required Time - Arrival Time
```

For a path where this relationship applies:

```text
Positive Slack
       ↓
Timing Requirement Met
```

```text
Negative Slack
       ↓
Timing Violation
```

---

## 🔹 OpenSTA and Propagated Clocks

After CTS, timing analysis can account for the physical clock network.

This allows analysis of:

```text
Clock Buffer Delay
       +
Clock Wire Delay
       +
Clock Latency
       +
Clock Skew
       ↓
More Realistic STA
```

### 💡 Key Idea

> **OpenSTA provides a timing-based view of whether the implemented design can operate within its specified constraints.**

---

# 13. 📈 WNS and TNS

Timing reports can contain many paths, so two summary values are particularly useful:

* **WNS — Worst Negative Slack**
* **TNS — Total Negative Slack**

---

## 🔹 WNS

WNS represents the smallest slack found among the analyzed paths.

```text
WNS = Minimum Slack
```

For example:

```text
Path 1 → +0.18 ns
Path 2 → -0.07 ns
Path 3 → -0.21 ns
```

Then:

```text
WNS = -0.21 ns
```

A negative WNS means that at least one analyzed path has a timing violation.

---

## 🔹 TNS

TNS represents the combined negative slack across violating paths.

```text
TNS
=
Sum of Negative Slack Values
```

Example:

```text
Path 1 → -0.05 ns
Path 2 → -0.10 ns
Path 3 → -0.15 ns
```

Therefore:

```text
TNS
=
-0.05 - 0.10 - 0.15

TNS = -0.30 ns
```

---

## 🔹 Interpretation

```text
WNS ≥ 0
```

means the worst analyzed path has no negative slack.

```text
TNS = 0
```

means there is no accumulated negative slack.

### 💡 Key Idea

> **WNS identifies the most critical negative-slack path, while TNS indicates the total amount of negative slack across violating paths.**

---

# 14. 💻 OpenLane and OpenROAD Commands

The following commands are useful while working through the physical-design flow.

---

## 🔹 Synthesis

```tcl
run_synthesis
```

This starts RTL synthesis.

---

## 🔹 Floorplanning

```tcl
run_floorplan
```

This performs the initial floorplanning stage.

---

## 🔹 Placement

```tcl
run_placement
```

This places the synthesized standard cells inside the floorplan.

---

## 🔹 Clock Tree Synthesis

```tcl
run_cts
```

This creates the physical clock distribution network.

---

## 🔹 Check Synthesis Strategy

```tcl
echo $::env(SYNTH_STRATEGY)
```

This displays the configured synthesis strategy.

---

## 🔹 Enable Synthesis Buffering

```tcl
set ::env(SYNTH_BUFFERING) 1
```

This enables synthesis buffering.

---

## 🔹 Enable Synthesis Sizing

```tcl
set ::env(SYNTH_SIZING) 1
```

This enables cell-sizing optimization during synthesis.

---

## 🔹 Launch OpenROAD

```bash
openroad
```

OpenROAD can be used for interactive physical-design analysis and commands.

---

## 🔹 Generate Timing Report

```tcl
report_checks \
-path_delay min_max \
-format full_clock_expanded \
-digits 4
```

This command can be used to inspect timing paths and their detailed clock information.

---

## 🔹 Report Setup Clock Skew

```tcl
report_clock_skew -setup
```

---

## 🔹 Report Hold Clock Skew

```tcl
report_clock_skew -hold
```

---

## 🔹 Important Timing Report Fields

| Parameter         | Description                                   |
| :---------------- | :-------------------------------------------- |
| **Arrival Time**  | Time at which the signal reaches the endpoint |
| **Required Time** | Time by which the signal should arrive        |
| **Slack**         | Available timing margin                       |
| **Data Delay**    | Delay through the data path                   |
| **Clock Delay**   | Delay through the clock path                  |
| **Clock Skew**    | Difference in clock arrival                   |
| **WNS**           | Worst slack in the analyzed paths             |
| **TNS**           | Combined negative slack                       |

---

# 15. 🔄 Complete Timing Flow

Timing analysis becomes clearer when viewed as a sequence of physical-design stages.

```text
             Standard Cell Models
                     │
                     ▼
              Timing Tables
                     │
                     ▼
             Slew + Load Effects
                     │
                     ▼
              Initial STA
                     │
                     ▼
                Placement
                     │
                     ▼
          Clock Tree Synthesis
                     │
                     ▼
          Clock Skew / Latency
                     │
                     ▼
          Routing + Parasitics
                     │
                     ▼
           Signal Integrity
                     │
                     ▼
          Propagated Clock STA
                     │
              ┌──────┴──────┐
              ▼             ▼
            Setup          Hold
              │             │
              └──────┬──────┘
                     ▼
               Timing Report
                     │
                     ▼
                 WNS / TNS
                     │
                     ▼
              Timing Closure
```

The important idea is that timing becomes increasingly realistic as physical information is added.

---

# 16. 📊 Quick Reference Tables

## 🔹 Setup vs Hold

| Feature             | Setup Check            | Hold Check          |
| :------------------ | :--------------------- | :------------------ |
| Delay Type          | Maximum delay          | Minimum delay       |
| Main Problem        | Data arrives late      | Data arrives early  |
| Reference           | Next active clock edge | Current clock edge  |
| Important Parameter | Setup time             | Hold time           |
| Violation           | Negative setup slack   | Negative hold slack |
| Target              | Slack ≥ 0              | Slack ≥ 0           |

---

## 🔹 Ideal Clock vs Propagated Clock

| Property        | Ideal Clock  | Propagated Clock |
| :-------------- | :----------- | :--------------- |
| Physical CTS    | Not included | Included         |
| Clock Buffers   | Not modeled  | Modeled          |
| Clock Wires     | Not modeled  | Modeled          |
| Clock Latency   | Simplified   | Included         |
| Clock Skew      | Idealized    | Physical         |
| RC Effects      | Limited      | Included         |
| Timing Accuracy | Preliminary  | More realistic   |

---

## 🔹 Clock Parameters

| Parameter             | Meaning                                  |
| :-------------------- | :--------------------------------------- |
| **Clock Period**      | Time between consecutive clock edges     |
| **Clock Latency**     | Time taken by clock to reach an endpoint |
| **Clock Skew**        | Difference in clock arrival times        |
| **Clock Jitter**      | Variation in clock edge position         |
| **Clock Uncertainty** | Timing margin used for clock variations  |

---

## 🔹 Timing Metrics

| Metric            | Meaning                                      |
| :---------------- | :------------------------------------------- |
| **Arrival Time**  | Actual signal arrival                        |
| **Required Time** | Allowed arrival time                         |
| **Slack**         | Difference between required and arrival time |
| **WNS**           | Minimum slack                                |
| **TNS**           | Sum of negative slack                        |

---

# 17. 🛠️ Tools Used

| Tool           | Main Purpose                              |
| :------------- | :---------------------------------------- |
| **OpenLane**   | Automated RTL-to-GDS physical-design flow |
| **Yosys**      | RTL synthesis                             |
| **OpenROAD**   | Physical implementation                   |
| **OpenSTA**    | Static Timing Analysis                    |
| **TritonCTS**  | Clock Tree Synthesis                      |
| **SKY130 PDK** | Technology and standard-cell information  |
| **Magic**      | Layout and physical verification          |
| **Linux**      | VLSI development environment              |

---

# 18. 🎯 Key Learnings

### 🧠 Timing Concepts

* Cell delay changes with input slew and output load.
* Standard-cell timing information is stored in timing libraries.
* Delay tables allow timing tools to estimate cell behaviour efficiently.
* Setup timing is a maximum-delay check.
* Hold timing is a minimum-delay check.
* Clock jitter introduces uncertainty into clock arrival.
* Clock uncertainty reduces usable timing margin.

### 🌳 Clock Network

* CTS creates the physical clock distribution network.
* Clock buffers are inserted to distribute clock signals.
* Clock skew represents differences in clock arrival.
* Clock latency represents clock travel time.
* Proper clock distribution is essential for sequential timing.

### 🔊 Physical Effects

* Interconnect contributes to total path delay.
* Crosstalk can disturb neighbouring signals.
* Signal integrity becomes increasingly important after routing.
* Clock shielding can reduce coupling around critical clock nets.

### 🔬 Timing Verification

* OpenSTA performs static timing analysis.
* Timing can be analyzed using setup and hold checks.
* WNS identifies the smallest slack.
* TNS represents the accumulated negative slack.
* Timing closure requires all important timing constraints to be satisfied.

---

# 19. 🏁 Conclusion

Module 4 focuses on how timing behaviour is evaluated as a digital design moves from logical implementation toward physical realization.

The learning starts with standard-cell timing and gradually introduces the physical effects that influence real circuits.

The overall progression can be represented as:

```text
       Timing Characterization
                │
                ▼
          Cell Delay Models
                │
                ▼
          Slew + Load
                │
                ▼
          Setup / Hold
                │
                ▼
       Clock Uncertainty
                │
                ▼
        Clock Tree Synthesis
                │
                ▼
        Skew + Clock Latency
                │
                ▼
        Routing / Interconnect
                │
                ▼
       Crosstalk / SI Effects
                │
                ▼
       Propagated Clock STA
                │
                ▼
             WNS / TNS
                │
                ▼
          Timing Closure
```

A useful way to understand the final timing picture is:

```text
                TIMING
                   │
        ┌──────────┴──────────┐
        │                     │
        ▼                     ▼
     DATA PATH            CLOCK PATH
        │                     │
        ▼                     ▼
  Cell + Wire Delay       CTS Network
        │                     │
        │                Buffer + Wire
        │                     │
        └──────────┬──────────┘
                   ▼
                 OpenSTA
                   │
             ┌─────┴─────┐
             ▼           ▼
           Setup        Hold
             │           │
             └─────┬─────┘
                   ▼
             Timing Closure
```

The major transition in this module is from an ideal timing model toward a physical timing model.

```text
Ideal Timing
     │
     ▼
Cell Delay
     +
Wire Delay
     +
Clock Delay
     +
Clock Skew
     +
Parasitic Effects
     +
Crosstalk
     │
     ▼
Realistic Timing Analysis
```

Therefore, timing analysis is not limited to checking logic delay. It combines the behaviour of standard cells, interconnects, clock networks and physical effects to determine whether the implemented design can operate reliably at its target frequency.

---

## 📌 Final Takeaway

> **A successful physical design must not only be logically correct — its data and clock signals must also reach their destinations within the required timing windows.**

```text
        LOGIC
          +
     STANDARD CELLS
          +
      INTERCONNECT
          +
      CLOCK NETWORK
          +
     SIGNAL INTEGRITY
          │
          ▼
      TIMING ANALYSIS
          │
          ▼
      TIMING CLOSURE
```
