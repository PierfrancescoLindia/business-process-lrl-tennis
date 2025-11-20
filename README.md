# LRL Tennis SRL – Business Process Modeling and Performance Analysis

This project models and analyses the **business process of LRL Tennis SRL**, a manufacturing company that produces tennis rackets, from raw material procurement to shipping and invoicing.  

The work combines **workflow modeling**, **queue-based performance analysis** and **resource-aware simulation** using **Yasper** and **YAWL**. :contentReference[oaicite:0]{index=0}

---

## 1. Objectives

The main objectives of the project are:

1. **Model the end-to-end production and order management process** of LRL Tennis SRL.
2. **Analyse performance** using queueing theory (M/M/1 and M/M/2 queues) for different constructs:
   - sequential tasks  
   - loop/iterative structures  
   - parallel branches (AND-split)  
   - exclusive choices (XOR-split)
3. **Identify bottlenecks** (in particular in the production phase) and quantify their impact on
   - waiting times,
   - cycle time,
   - server utilization.
4. **Propose improvements** (e.g. additional servers, role merging) and validate them through simulation.

The project was developed for the course *Sistemi Informativi: Processi di Business* at the University of Calabria.

---

## 2. Artefacts in this repository

- **`Relazione Finale LRL Tennis SRL.pdf`**  
  Full report (Italian) describing the process, modeling choices, performance tables and simulation results.

- **`Analisi delle prestazioni - LRL Tennis SRL.xlsx`**  
  Excel workbook with theoretical performance measures (L, Lq, W, Wq, ρ) for different inter-arrival times and queue configurations.

- **`LRL Tennis SRL.pnml`**  
  Yasper model of the process in Petri-net format, used for workflow simulation and performance analysis.

- **`LRL Tennis SRL.yawl` / `LRL Tennis SRL.yawl.bak`**  
  YAWL model of the same process, including data variables, conditions and resource allocation.

---

## 3. Business process description

The production process is modelled in Yasper as a Petri-net with **14 tasks** and several control-flow constructs (XOR, AND, loop).

Main steps:

1. **Order Reception** – the clerk receives a new order.
2. **Inventory Check** – the warehouse worker checks material availability.  
   - XOR-split:
     - **Materials available** → directly to Production.
     - **Materials not available** → **Procurement** and then Production.
3. **Production start** – the order is released to the shop floor.
4. **Parallel manufacturing** (AND-split):
   - **Frame production** (task 4.2)  
   - **String production** (task 4.3)  
   Tasks are joined in **Racket Assembly** (task 4.4).
5. **String tensioning** – the racket strings are tensioned.
6. **Quality check** – loop with XOR:
   - if the check fails, the process goes back to **Inventory Check** (start of the production cycle);
   - if the check passes, the product moves to **Packaging**.
7. **Packaging** – the product is packed and ready for shipping.
8. **Payment choice** – XOR-split:
   - **Payment by cheque** (task 8.1),
   - **Payment in cash** (task 8.2).
9. **Shipping** – the shipment is prepared and sent.
10. **Invoice emission** – the final invoice is issued, closing the case.

Task times (in minutes), costs and routing probabilities are specified for each activity in the parametrization tables and are used as input for performance analysis.

---

## 4. Performance analysis (Yasper + Excel)

Performance analysis is based on a sequence of **queueing models** (mainly M/M/1 and M/M/2 queues) associated with each task.

### 4.1 Parameters

For every task the following parameters are defined:

- **Mean service time** and **variance**  
- **Fixed** and **variable cost**  
- **Inter-arrival time** between orders (20 minutes in the baseline scenario)
- **Routing probabilities**:
  - availability/non-availability of materials,
  - pass/fail at quality check,
  - payment method (cheque vs cash, 0.5 / 0.5).

From these, the Excel file computes:

- arrival rate \(\lambda\) and service rate \(\mu\)  
- **utilization** ρ = λ/µ  
- **L**: mean number of customers in the system  
- **Lq**: mean number in queue  
- **W**: mean time in system  
- **Wq**: mean waiting time in queue.

### 4.2 Sequential construct

For the initial task *Order Reception* the analysis shows how different inter-arrival times (10, 20, 50, 100 minutes, …) affect:

- the stability condition (ρ < 1),
- queue length,
- waiting time.

The discussion explains that stability can be improved either by **reducing service times** (e.g. better tools or automation) or by **increasing the number of servers** assigned to the task.

### 4.3 Iterative construct (quality-check cycle)

The **loop** around the quality check is analysed separately:

- Forward branch: normal flow when quality is passed.
- Backward branch: re-entry into production when the product fails the check.

The total performance of the cycle is obtained by combining the two branches, weighted by the probability of rework (5%).  

Results show that the adoption of **M/M/2 queues** in the most critical tasks significantly reduces the total cycle time of this loop, especially for small inter-arrival times.

### 4.4 Parallel construct (AND-split)

In the parallel segment (frame production and string production):

- Tasks **4.2 (Frame)** and **4.4 (Assembly)** initially show severe **bottlenecks** under M/M/1 assumptions:
  - very high **Lq** and **Wq**,
  - high utilization ρ close to 1.
- Switching these tasks to **M/M/2** (two operators per task) reduces congestion:
  - Lq and Wq decrease substantially,
  - ρ is roughly halved,
  - the overall Work Time of the parallel block improves.

A comparison of tables and graphs clearly shows that with M/M/1 queues, the total Work Time explodes for inter-arrival times smaller than 90 minutes, while M/M/2 maintains acceptable performance.

### 4.5 XOR (payment choice)

For the **payment** XOR:

- probability of cheque = 0.5, cash = 0.5,
- inter-arrival times in each branch are doubled (20 → 40 minutes), since arrivals are split across the two alternatives.

Queue metrics in both branches reflect this reduced arrival rate.

### 4.6 Cycle time analysis

A dedicated section compares **theoretical** cycle times (from the Excel queue models) with **simulated** cycle times obtained from Yasper:

- The theoretical and simulated values are aligned.
- When using M/M/2 queues for tasks 4.2 and 4.4, all utilizations ρ remain below 0.7, and no additional workers are required.
- The difference in cycle time between M/M/1 and M/M/2 grows dramatically as inter-arrival time decreases, highlighting the importance of having two servers in the critical tasks.

---

## 5. YAWL model and resource management

The same process is implemented in **YAWL** to:

- represent the control flow, and  
- explicitly manage **data variables** and **resources**.

### 5.1 Data variables and routing

Examples of key boolean variables:

- `Materiali_non_Disponibili` – used in *Inventory Check* to decide whether to go to **Procurement**.
- `Check_Superato` – used in the quality check to decide whether to go to **Packaging** or to repeat production.
- `Assegno` – used at **Order Reception** to select the payment path (*Cheque* vs *Cash*).

These variables are used in **split predicates** that implement the XOR branches.

### 5.2 Roles and resources

Roles and resources are defined as:

- **Clerk** – receives orders.
- **Warehouse worker / Head of warehouse** – inventory check and procurement.
- **Workers** – frame production, string production, assembly, string tensioning.
- **Quality control manager** – product integrity check.
- **Shipping staff** – packaging and shipping.
- **Accountants / Head accountant** – payment handling and invoice emission.

Each task is assigned to a role, and each role is associated with one or more **human resources**, enabling realistic simulation of work distribution and utilization.

The YAWL screenshots in the report illustrate the **work items** that users see and how boolean variables affect the path of each case through the process.

---

## 6. Key findings and improvement suggestions

From the combined theoretical and simulation analysis:

- The **main bottlenecks** are tasks **4.2 (Frame production)** and **4.4 (Racket assembly)**.
- Moving from M/M/1 to **M/M/2 queues** in these tasks:
  - substantially reduces queue length and waiting time,
  - lowers utilization to safe levels,
  - produces a much lower overall cycle time, especially when orders arrive frequently.
- Further improvements can be obtained by:
  - **merging rarely used roles** (e.g. the two accountants or some warehouse roles) to increase server utilization,
  - continuously monitoring inter-arrival and service times to maintain ρ well below 1,
  - using simulation to test alternative staffing policies before applying them in the real company.

---

## 7. Recommended repository structure

A clean structure for this project is:

```text
.
├─ models/
│   ├─ LRL_Tennis_SRL.pnml           # Yasper model
│   ├─ LRL_Tennis_SRL.yawl           # YAWL model
│   └─ LRL_Tennis_SRL.yawl.bak       # YAWL backup
│
├─ data/
│   └─ Analisi_prestazioni_LRL_Tennis.xlsx   # Queueing/performance analysis
│
├─ docs/
│   └─ Relazione_Finale_LRL_Tennis_SRL.pdf   # Full report (Italian)
│
├─ .gitignore
├─ LICENSE
└─ README.md

