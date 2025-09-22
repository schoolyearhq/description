# Schoolyear – Open Problem Statement

## The Problem  
- Millions of children in low-income countries are unable to attend school because their families cannot pay the annual fees.  
- Schools, teachers, and government-recognised institutions *already exist*. The bottleneck is **tuition costs**.  
- Existing charities often:  
  - Spend heavily on overhead, marketing, or “feel-good” activities.  
  - Lack transparency (donors don’t see where their money goes).  
  - Don’t scale, because they try to build new infrastructure instead of plugging into existing systems.  

## Our Approach  
- Treat education as a **unit of distribution**:  
  - **€200 = 1 school year.**  
- **98% efficiency target** → money goes directly to paying school fees at recognised institutions.  
- **Verifiable impact** → for every unit funded, we provide anonymised copies of official enrollment/completion certificates.  
- **No feel-good charity** → just clean infrastructure that channels resources into existing schools.  

## Why This Matters  
- **Scalable** → the model can run anywhere there are schools + fees.  
- **Transparent** → donors see proof of exactly what they funded.  
- **Efficient** → almost all funds go directly to the solution.  

## What We Need Help With  
We want developers and system thinkers to help design a lean, auditable, and scalable platform.  
Key open questions include:  

1. **Data model** – how do we structure “school years” as verifiable units?  
2. **Verification** – what’s the simplest system for collecting, anonymising, and publishing enrollment certificates?  
3. **Donor portal** – how do we let donors (especially corporates) see impact clearly and securely?  
4. **Efficiency** – what’s the lightest infrastructure that avoids overhead creep?  
5. **Fraud prevention** – how do we minimise risk of fake students or double-counting?  

## How to Contribute  
- Discuss ideas in [Issues](../../issues).  
- Propose simple system designs.  
- Help us test lean prototypes.  
- Share knowledge from similar domains (payments, transparency, auditing).  

---

✦ This project is at the **idea and pilot stage**. We’re not asking for donations — we’re asking for **collaborators** who want to help design a transparent, scalable way to fund education.

## The Mathematical Problem (Risk-Minimizing Allocation)

We want the **optimal way to allocate a finite pool of funds** across many schools so that we **minimize the risk of default (or disruption)** at schools while respecting timing and budget constraints.

### Formalization (first draft)

- **Sets**
  - Schools \(i \in \{1,\dots,N\}\); each may represent a single school or a batch (grade, district).
  - Billing periods \(t \in \{1,\dots,T\}\) (e.g., months/terms).

- **Parameters**
  - \(f_{i,t}\): fee due at school \(i\) in period \(t\).
  - \(d_{i,t}\): due date (or latest safe payment date) for \(f_{i,t}\).
  - \(B_t\): budget available in period \(t\) (cash on hand + expected inflows).
  - \(p_{i,t}\): penalty if \(f_{i,t}\) is missed/late (can be monetary or mapped to risk points).
  - \(r_{i,t}(\Delta)\): **default risk function** for school \(i\) if payment is delayed by \(\Delta\) days relative to \(d_{i,t}\). Monotone increasing in \(\Delta\).
  - \(w_i\): priority weight (e.g., number of students covered, equity weighting).

- **Decision variables**
  - \(x_{i,t} \in [0,f_{i,t}]\): amount paid to school \(i\) in period \(t\).
  - \(s_{i,t} \ge 0\): shortfall at \((i,t)\), where \(s_{i,t} = f_{i,t} - \sum_{\tau \le t} x_{i,\tau}\).

- **Objective (examples)**
  1. **Minimize expected default risk**  
     \[
     \min \sum_{i,t} w_i \cdot r_{i,t}\!\big(\mathrm{late}_{i,t}\big)
     \]
     where \(\mathrm{late}_{i,t}\) is the days late implied by remaining shortfall \(s_{i,t}\) after \(d_{i,t}\).
  2. **Or** a hybrid: minimize (risk + financial penalties)  
     \[
     \min \sum_{i,t} \big[w_i \cdot r_{i,t}(\cdot) + \lambda \, p_{i,t}\cdot \mathbb{1}\{s_{i,t}>0\}\big]
     \]

- **Constraints**
  - **Budget per period:** \(\sum_i x_{i,t} \le B_t \quad \forall t\)
  - **No overpaying:** \(0 \le x_{i,t} \le f_{i,t}\)
  - **Carryover optional:** unspent \(B_t\) can roll into \(B_{t+1}\) if treasury policy allows.
  - **Service level (optional):** \(\Pr[s_{i,t}=0] \ge \alpha\) for target reliability \(\alpha\) (chance-constraints).

### Modeling the risk function \(r_{i,t}(\cdot)\)

We’re open to proposals. Options include:
- **Piecewise-linear lateness cost** (simple, MILP-friendly).
- **Survival/hazard model** using historical defaults vs. delay.
- **Logistic risk:** \(r(\Delta)=\frac{1}{1+e^{-(a_i+b_i\Delta)}}\).
- **Queueing/credit-risk analogs** for cascading effects.

### Baselines to beat

- **EDD (Earliest Due Date)**: pay the soonest deadlines first.
- **Risk-Weighted EDD**: order by \(w_i \cdot \partial r_{i,t}/\partial \Delta\) at current lateness.
- **Knapsack per period**: choose payments that maximize risk reduction per euro.

### What we want from contributors

- A **tractable optimization** (LP/MILP/stochastic program or heuristic) that:
  - Works online with partial information and rolling budgets.
  - Produces **payment schedules** \(x_{i,t}\) and **risk reports**.
  - Handles **uncertain inflows** (Monte Carlo / robust constraints).
- A minimal **simulation harness** to compare policies on synthetic data:
  - Inputs: \(\{f_{i,t}, d_{i,t}, B_t, w_i, r_{i,t}\}\)
  - Outputs: defaults avoided, total penalties, % on-time, compute time.

### Edge cases & policy knobs

- Hard caps per school (equity): \(x_{i,\cdot} \le \beta_i\).
- Reserve buffer: keep \(k\%\) of \(B_t\) uncommitted for shocks.
- Prepayment discounts or installment plans (change \(f_{i,t}\), \(p_{i,t}\)).

> **Goal:** a lean algorithm that can run nightly to recommend payments that **minimize expected defaults** under a rolling budget, with clear guarantees or strong empirical performance.
