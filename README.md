# EECS251LA: Introduction to Digital Design and Integrated Circuits Lab
Project report on design of a RISC-V 3-stage pipelined CPU for EECS251LA class at UC Berkeley


## Introduction

We implemented a 3-stage pipeline consisting of the following stages:
- **FD** (Fetch and Decode)
- **X** (Execute)
- **MWB** (Memory and Write Back)

In the **FD** stage, both the **IMEM** and **PC register** were clocked simultaneously. In the **X** and **MWB** stages, the **DMEM** and pipeline registers between these stages were also clocked together.

![Pipeline Diagram](images/pipeline.png)

## Cache

Initally, we used a no-cache architecture that reads directly from the memory. We later swwapped it for a direct-mapped cache. 
![Pipeline Diagram](images/pipeline.png)

### Critical Path Analysis

One of the critical paths we anticipated lies between the pipeline register storing the ALU output during the transition from the **X** stage to the **MWB** stage and the **DMEM** cache **dout**. This path corresponds to a **load instruction**. The signals pass through complex logic to determine the **DCACHE read/write enable signals**, access the **DCACHE**, and then traverse the **load MUX** and **WB MUX**. Due to the complexity of this sequence, we expected this path to have the longest delay.

### Forwarding Mechanism

To address the delays and optimize performance, we implemented forwarding for various dependencies:
- **MWB to ALU (1 cycle)**
- **LOAD (2 cycles)**
- **JALR (1 or 2 cycles)**

These forwarding mechanisms will be explained further in the subsequent sections.

## Pipeline Stages

### (a) FD Stage

The following components are part of the **FD stage**:
- **PC select mux**
- **IMEM**
- **PC register**
- **PC+4 adder**
- **Imm Gen**
- **Register file**

### (b) X Stage

In the **X stage**, we have the following components:
- **Branch comparator**
- **Mux to select the right rs1/rs2 for forwarding**
- **ALU**
- **Store mask**
- **Write select mux (chooses between ALU out and rs2)**

### (c) MWB Stage

The **MWB stage** includes:
- **DMEM**
- **CSR logic**
- **Load MUX**
- **WB MUX**

## Forwarding and Stall Management

We implemented forwarding for 1-cycle and 2-cycle dependencies to minimize stalls, particularly in scenarios with no-cache
