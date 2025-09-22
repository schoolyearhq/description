## The Mathematical Problem

A school with 100 students costs $20,000/year. How should Schoolyear decide how to allocate the funds, year after year?

---

In places like **Uganda**, a **primary school year costs ~$200** per child. That money usually covers **uniforms, books, meals, exam fees, and other essentials** that aren’t fully funded by the government. 

To keep schools running smoothly, we want to build a **pooled fund** that:
1. sends each partner school an **up-front allocation at the start of the academic year**, and
2. **releases next year’s allocation only if last year’s anonymised enrollment/completion certificates were provided** (our proof of impact).

This creates a **risk-management problem**:

- What is the **best strategy to maximise continuity**—i.e., the odds that each partner school keeps receiving payments year after year?
- When should Schoolyear **add a new school** without endangering existing commitments?
- How to define an **acceptable risk threshold** for expansion?
- How can to **estimate the reliability** of each **funder** (corporate donor, foundation, etc.) so Schoolyear's promises to schools are credible?

Below is a first draft of a framework that developers/researchers can formalise and simulate.

---

### 1) Core goal

> **Allocate a finite yearly budget across schools to minimise the risk of disruption (missed payments) while safely expanding to new schools.**

We measure *disruption* as: a school that was promised support but **doesn’t receive the full amount on time** for the coming academic year.

---

### 2) Inputs (what we track)

- **Schools** \(i = 1..N\)
  - Annual need per school \(C_i\) (e.g., $200 × number of sponsored students).
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