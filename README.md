
![Schoolyear Diagram](img.webp)

# Schoolyear – Open Problem Statement

## The Problem  
- Millions of children in low-income countries are unable to attend school because their families cannot pay the annual fees.  
- Schools, teachers, and government-recognised institutions *already exist*. One of the main bottlenecks is **tuition costs**.  
- Existing charities often:  
  - Spend heavily on overhead, marketing, or “feel-good” activities.  
  - Lack transparency (donors don’t see where their money goes).  
  - Don’t scale, because they try to build new infrastructure instead of plugging into existing systems.  

## Schoolyear Approach  
- Treat education as a **unit of distribution**, for example:  
  - **€200 = 1 school year.**  
- **98% efficiency target** → money goes directly to paying school fees at recognised institutions, which emit a receipt.  
- **Verifiable impact** → for every unit funded, we provide anonymised copies of official enrollment/completion certificates.  
- **No feel-good charity** → just clean infrastructure that channels resources into existing schools.  

## Advantages  
- **Scalable** → the model can run anywhere there are schools + fees.  
- **Transparent** → donors see proof of exactly what they funded.  
- **Efficient** → almost all funds go directly to the solution.  

## What Schoolyear is *Not*

To stay focused and efficient, it is important to be clear about what Schoolyear does **not** attempt to solve.  

1. **Not addressing the quality of education**  
   - Schoolyear does not intervene in teacher training, curriculum design, or classroom pedagogy.  
   - Many organisations already work on improving quality; the role of Schoolyear is to ensure that more children can access school in the first place.  

2. **Not preferring one demographic segment over another**  
   - Schoolyear is not designed to prioritise boys over girls, or vice versa.  
   - Every child’s school year is treated as the same €200 unit in the model, and funding is provided on equal terms.  
   - Partner organisations may choose to emphasise particular groups, but the underlying infrastructure remains neutral.  

3. **Not building or managing physical infrastructure**  
   - Schoolyear does not construct schools, hire teachers, or run logistics.  
   - The system plugs into government-recognised institutions that already exist.  
   - The value lies in removing the financial bottleneck of school fees, not duplicating infrastructure.  

---

### Why these boundaries matter

- **Clarity** → Donors, funders, and partners know exactly what their contribution does: *€200 = one year of school*.  
- **Efficiency** → By avoiding scope creep into quality, equity politics, or infrastructure, the model maintains the **98% efficiency target**.  
- **Scalability** → Neutrality and simplicity make the system easy to replicate in multiple contexts without becoming entangled in policy debates.  

> Schoolyear is a **distribution mechanism**: it channels funds directly into school years, with proof. Nothing more, nothing less.

## The Mathematical Problem (Risk-Minimizing Allocation)

We want the **optimal way to allocate a finite pool of funds** across many schools so that we **minimize the risk of default (or disruption)** at schools while respecting timing and budget constraints.

## The Mathematical Problem (in plain language)

In places like **Uganda**, a **primary school year costs ~€200** per child. That money usually covers **uniforms, books, meals, exam fees, and other essentials** that aren’t fully funded by the government. 

To keep schools running smoothly, we want to build a **pooled fund** that:
1. sends each partner school an **up-front allocation at the start of the academic year**, and
2. **releases next year’s allocation only if last year’s anonymised enrollment/completion certificates were provided** (our proof of impact).

This creates a **risk-management problem**:

- What is the **best strategy to maximise continuity**—i.e., the odds that each partner school keeps receiving payments year after year?
- When should we **add a new school** without endangering existing commitments?
- How do we define an **acceptable risk threshold** for expansion?
- How can we **estimate the reliability** of each **funder** (corporate donor, foundation, etc.) so our promises to schools are credible?

Below is a first draft of a framework that developers/researchers can formalise and simulate.

---

### 1) Core goal

> **Allocate a finite yearly budget across schools to minimise the risk of disruption (missed payments) while safely expanding to new schools.**

We measure *disruption* as: a school that was promised support but **doesn’t receive the full amount on time** for the coming academic year.

---

### 2) Inputs (what we track)

- **Schools** \(i = 1..N\)
  - Annual need per school \(C_i\) (e.g., €200 × number of sponsored students).
  - Due date \(D_i\) (start of school year / latest safe payment date).
  - **Compliance score** \(K_i\) (0–1): did they deliver certificates on time last year? quality? audit flags?
  - **Priority weight** \(W_i\) (e.g., equity, girls’ education, rural hardship).

- **Funders** \(j = 1..M\)
  - Pledged amount \(P_j\) and **payment schedule** (lump sum vs quarterly).
  - **Reliability score** \(R_j \in [0,1]\): probability they pay as pledged (based on track record, contract type, credit proxy).
  - Correlation info (optional): are some funders correlated (e.g., same industry cycle)?

- **Treasury**
  - Cash in bank at t=0 and expected inflows over the year (from funders).
  - Reserve policy: minimum **buffer** \(b\%\) of expected inflows kept uncommitted.

---

### 3) Key risk ideas (simple, but useful)

- **Funding reliability (portfolio view)**  
  Compute **expected available funds** and a **downside (stress) scenario**:
  - Expected: \( \mathbb{E}[\text{Funds}] = \sum_j R_j \cdot P_j \)
  - Stress (e.g., 10th percentile): assume a subset of funders under-deliver (simulate via Monte Carlo or apply haircut \(h\%\)).

- **School continuity risk**  
  A school is at risk if its allocation depends on fragile inflows. Useful surrogate:
  \[
  \text{ContinuityScore}_i = K_i \cdot \frac{\text{CommittedSecure}_i}{C_i}
  \]
  where *CommittedSecure* is funded from high-reliability sources plus treasury buffer. Higher is safer.

- **System expansion risk**  
  Define a target **Probability of Coverage** \( \Pr(\text{all existing schools fully funded on time}) \ge \alpha \) (e.g., 95%).  
  Only add schools if this constraint remains true under the **stress scenario**.

---

### 4) Policies we can implement (and compare)

#### A. Allocation at year start (who gets paid first?)
- **Continuity-first**: fully fund all existing compliant schools (highest \(K_i\)) **in order of earliest due date** \(D_i\), while keeping the reserve buffer.
- **Risk-weighted priority**: rank by \( W_i \times K_i \times \text{ContinuityScore}_i \) (or its shortfall), pay until the stress-tested budget is exhausted.
- **Hybrid**: guarantee a **baseline %** (e.g., 80%) to every compliant school, then distribute remaining funds by risk-weighted priority.

#### B. When to add a new school?
Only add a new school \(s\) if **all** are true:
1. **Compliance gate**: last year’s certificates from existing schools were received (or exceptions are justified).
2. **Coverage constraint**: under stress, \(\Pr(\text{existing fully funded on time}) \ge \alpha\).
3. **Buffer intact**: reserve after adding \(s\) remains ≥ \(b\%\).
4. **Unit test**: the new school has passed **onboarding checks** (licensing, fee validation) → provisional \(K_s\) is acceptable.
5. **Diversity** (optional): avoid concentration (e.g., max share per country/district).

#### C. What is an acceptable expansion risk?
Pick **two numbers** and stick to them:
- **Reliability target**: \(\alpha = 95\%\) (or 97.5%) probability that all existing schools are funded on time, **under stress**.
- **Buffer**: \(b = 15\%\) (example) of expected annual inflows kept as cash or near-cash.

Expansion proceeds only when both are satisfied.

---

### 5) Estimating **funder reliability** \(R_j\)

A simple scorecard (0–100 → scale to 0–1):

- **Track record** (on-time payments last 2 years): up to 40 pts  
- **Contract strength** (binding agreement vs soft pledge): up to 20 pts  
- **Payment schedule** (lump sum early > quarterly > end-loaded): up to 15 pts  
- **Financial health proxy** (public info/ratings): up to 15 pts  
- **Concentration risk** (not all from one sector/country): up to 10 pts  

**Map to \(R_j\)**:  
- 85–100 → 0.95  
- 70–84  → 0.85  
- 50–69  → 0.70  
- <50    → 0.50 (or exclude)

Update \(R_j\) **Bayesian-style** each quarter: if they pay on time, nudge up; if they’re late/miss, nudge down.

---

### 6) Minimal math formulation (for implementers)

- **Decision**: allocation \(x_i \in [0, C_i]\) at year start.
- **Budget constraint (stress-tested)**:
  \[
  \sum_i x_i \le (1-b)\cdot \text{Funds}_{\text{stress}}
  \]
- **Objective (one option)**: maximise risk-adjusted continuity
  \[
  \max \sum_i W_i \cdot K_i \cdot \min\!\left(1,\ \frac{x_i}{C_i}\right)
  \]
- **Guarantee**: all previously supported, compliant schools \(i\) must receive \(x_i \ge C_i\) **before** any new additions—unless violating the buffer or \(\alpha\)-constraint.

This can be solved as a **small LP/MILP**, then re-run whenever pledges change.

---

### 7) What to build (developer brief)

- A tiny **simulator**:
  - Inputs: schools \(\{C_i, D_i, K_i, W_i\}\), funders \(\{P_j, R_j\}\), buffer \(b\), reliability target \(\alpha\).
  - Engine:  
    1) compute stress funds (Monte Carlo or haircut),  
    2) allocate by chosen policy,  
    3) report who is fully covered, buffer remaining, and \(\Pr(\text{coverage})\).
- **Dashboards**:
  - “Can we add 5 schools?” → green/red based on \(\alpha\) & buffer.
  - “Which funders drive risk?” → sensitivity by \(R_j\).
  - “What’s our continuity score by country?” → prioritise attention.

---