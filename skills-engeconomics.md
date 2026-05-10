# Engagement Economics — Complete Reference

> **Purpose of this document:** A complete, accurate reference for how the Engagement Economics system works — covering every metric, formula, workflow stage, and design decision. Written for use when working with Claude in Excel to build or extend budget templates.

---

## 1. The Mental Model

The system answers two questions simultaneously:

1. **Planning** — Given an agreed fee, how should I staff this engagement to hit my target margin?
2. **Tracking** — As the engagement runs, am I over or under budget? Have I billed what I've earned?

Everything flows from **one primary input: the Agreed Fee**. All metrics derive from it.

---

## 2. Rate Card

Every rank has exactly two rates:

| Field | Meaning |
|-------|---------|
| `NSR/hr` | Net Standard Revenue rate — the rack-rate value of one hour of this rank's time |
| `Cost/hr` | Internal labour cost — what the firm pays for one hour of this rank |

These are **per-hour dollar rates**, not annual figures. The gap between them is what generates margin.

### FY2025-26 Rate Card

| Rank | Abbr | NSR/hr | Cost/hr |
|------|------|--------|---------|
| Partner 3 | P3 | 634 | 1,489 |
| Partner 2 | P2 | 629 | 1,192 |
| Partner/Principal | P | 621 | 993 |
| Exec Director | ED | 454 | 381 |
| Director | D | 785 | 190 |
| Asso Director | AD | 437 | 122 |
| Asst Director | ASD | 294 | 86 |
| Senior Manager 2 | SM2 | 384 | 320 |
| Senior Manager 1 | SM | 342 | 213 |
| Manager | M | 223 | 128 |
| Senior Associate 3 | S3 | 176 | 92 |
| Senior Associate 2 | S2 | 159 | 84.96 |
| Senior Associate 1 | S1 | 138 | 79 |
| Staff 2 | A2 | 119 | 64 |
| Staff 1 | A1 | 93 | 64 |
| Supervising Associate | SA | 168 | 62 |
| Senior Associate | SRA | 138 | 50 |
| Associate | AS | 108 | 37 |
| Intern (CS) | IN | 45 | 9 |

### FY2026-27 Rate Card (abbreviated)

| Rank | Abbr | NSR/hr | Cost/hr |
|------|------|--------|---------|
| Partner/Principal | P | 640 | 1,038 |
| Director | D | 468 | 398.10 |
| Senior Manager 1 | SM | 396 | 334.40 |
| Manager | M | 230 | 133.80 |
| Senior Associate 3 | S3 | 181 | 96.14 |
| Senior Associate 2 | S2 | 164 | 85.69 |
| Senior Associate 1 | S1 | 142 | 82.56 |
| Staff 1 | A1 | 96 | 66.88 |
| Intern (CS) | IN | 45 | 9 |

> **Note on Partner cost/hr:** Partners have a very high Cost/hr (993–1,489) but relatively lower NSR/hr. This means Partner hours are expensive — they compress margin. The system typically targets only 2 Partner hours unless the budget is large enough to absorb more.

---

## 3. Inputs (What the User Provides)

### 3a. Budget Inputs (set once at engagement start)

| Input | Symbol | Notes |
|-------|--------|-------|
| Agreed Fee | `fee` | Total fee agreed with client. The primary driver of everything. |
| Billable Expenses | `bxp` | Expenses to be billed to client (travel, printing, etc.). |
| Tech Fee % | `tp` | Technology/overhead surcharge on labour cost. Default: 0.5%. |
| Target Margin % | `tgt` | Desired margin. Affects **Cost Allowance only** — does not affect NSR, ANSR, or EAF. |

### 3b. Tracking Inputs (updated as engagement progresses)

| Input | Symbol | Notes |
|-------|--------|-------|
| Charged Expenses | `cxp` | Expenses actually incurred to date. |
| Billed Fees | `bf` | Fee invoices raised to date. |
| Billed Expenses | `bbx` | Expense invoices raised to date. |
| Actual Hours | `a` per row | Hours actually charged per rank. Keyed in manually. |

### 3c. Resource Inputs (per staff row)

| Input | Notes |
|-------|-------|
| Rank / Desig | Determines Cost/hr and NSR/hr from rate card. |
| Budget Hours (`b`) | Planned hours for this rank. Source of truth for budget metrics. |
| ETC Hours (`e`) | Estimate-to-complete hours added post-budget-set. |
| Actual Hours (`a`) | Hours burned to date. Pre-filled to Budget Hours at budget-set, then updated. |

---

## 4. Derived Metrics — Complete Formula Reference

All formulas use these intermediate totals:

```
NSR_total   = Σ (hours_per_rank × NSR/hr_per_rank)
Labour_cost = Σ (hours_per_rank × Cost/hr_per_rank)
Tech_fee    = Labour_cost × Tech_Fee_%
```

---

### 4a. Revenue & Cost Metrics

| Metric | Formula | Notes |
|--------|---------|-------|
| **TER** (Total Engagement Revenue) | `fee + bxp` | The ceiling. Margin is calculated as a % of TER, not just fee. Matches Reporting Hub convention. |
| **Labour Cost** | `Σ(budget_hrs × cost/hr)` | Pure cost of the team at budgeted hours. |
| **Tech Fee** | `Labour_Cost × tp` | Small overhead levy on labour. |
| **Total Cost** | `Labour_Cost + Tech_Fee + bxp` | Everything that comes out of the fee. |
| **Margin $** | `fee − Total_Cost` | Net dollars remaining. |
| **Margin %** | `Margin_$ / TER` | Uses TER as denominator (includes expenses). |

---

### 4b. Planning Metrics

| Metric | Formula | Notes |
|--------|---------|-------|
| **Cost Allowance** | `TER × (1 − Target_Margin_%)` | The maximum you can spend and still hit your target. **This is the only output driven by Target Margin %.** |
| **NSR** | `Σ(budget_hrs × NSR/hr)` | Rack-rate value of resources deployed. The denominator for EAF. |
| **EAF** | `(fee / NSR) − 1` | Engagement Adjustment Factor. The premium (+) or discount (−) vs standard rates. Negative EAF is normal when rack rates exceed the agreed fee. |

> **Understanding EAF:** If NSR = $50,000 and Fee = $40,000, then EAF = (40,000/50,000) − 1 = −0.20 (−20%). The engagement is priced at a 20% discount to rack rates. The system flags EAF outside ±60% as an amber health warning.

---

### 4c. Tracking Metrics

| Metric | Formula | Notes |
|--------|---------|-------|
| **Actual Labour Cost** | `Σ(actual_hrs × cost/hr)` | What has actually been spent on people. |
| **Variance (Allow − Act)** | `Cost_Allowance − Actual_Labour_Cost` | **Positive = CREDIT** (headroom remaining). **Negative = OVERRUN** (spending exceeds what margin allows). |
| **Recognized ANSR** | `Σ(actual_hrs × NSR/hr) × (1 + EAF)` | Revenue recognized from actual hours worked, adjusted by the engagement's premium/discount factor. |
| **NUI** (Net Unbilled Inventory) | `(Recognized_ANSR + cxp) − (bf + bbx)` | **Positive = work done but not yet billed.** **Negative = billed ahead of work performed.** |

> **NUI intuition:** NUI tells you if your billing is keeping pace with your work. A high positive NUI means you're doing the work but not invoicing — cash flow risk. A negative NUI means you've billed more than you've earned — you're running ahead.

---

### 4d. ANSR — How It Works

ANSR (Adjusted NSR) is the fee allocated per person, weighted by their NSR contribution:

```
ANSR_per_person = (NSR_person / NSR_total) × Agreed_Fee
```

**By construction, Total ANSR always equals the Agreed Fee.** It's not a separate computation — it's just the fee redistributed proportionally by NSR weight.

At budget time: `ANSR = fee` (total).  
During tracking: `ANSR = Recognized ANSR` (based on actual hours × rate × EAF adjustment).

---

### 4e. EAF — Which NSR Is Used for NUI?

This is the subtle part. The EAF used to recognize revenue depends on the workflow stage:

| Stage | EAF Basis |
|-------|-----------|
| Pre-budget-set | EAF calculated live from current rows |
| Post-budget-set, pre-ETC-transfer | EAF uses the **snapshot NSR** taken at budget-set |
| Post-ETC-transfer | EAF recalculates using the **committed (budget + ETC) NSR** |

This ensures that once a budget is locked, the revenue recognition basis doesn't drift until an ETC is formally transferred.

---

## 5. Workflow Stages

The system has four states. Understanding these is critical for tracking.

```
plan  →  budgetSet (locked)  →  etcOpen  →  etcTransferred
```

### Stage 1: Plan (`lk = false`)
- All fields editable.
- Hours = Budget only (`b`). ETC (`e`) = 0.
- All metrics are live/provisional.
- Auto-allocate is available.

### Stage 2: Budget Set (`lk = true`)
- Triggered by clicking **Set Budget**.
- Requires Budget Hours > 0.
- **Snapshot is taken** at this moment:
  ```
  snap.bN  = Σ(b × NSR/hr)     ← Budget NSR
  snap.bL  = Σ(b × cost/hr)    ← Budget Labour Cost
  snap.bT  = Labour × tp       ← Budget Tech Fee
  snap.bC  = Labour + Tech + bxp ← Budget Total Cost
  snap.bM  = fee − snap.bC     ← Budget Margin $
  snap.bMP = snap.bM / TER     ← Budget Margin %
  ```
- Actuals pre-filled to Budget Hours (user updates from here).
- Budget pane now shows **snapshot values** (frozen). Live pane shows current actuals.

### Stage 3: ETC Open (`etcOpen = true`)
- Triggered by **Run ETC**.
- ETC hours (`e`) become editable per rank.
- Projected metrics recalculate as `b + e` (budget + estimate-to-complete).

### Stage 4: ETC Transferred (`ed = true`)
- Triggered by **Transfer ETC**.
- ETC is committed. Committed hours = `b + e`.
- EAF for NUI now uses the committed NSR (not the original budget snapshot).
- A log entry is created with the timestamp.
- Mercury should be updated: enter the new **projected NSR** as the Budget NSR.

---

## 6. Auto-Allocate Logic

The auto-allocate feature fills Budget Hours from the Cost Allowance. Rules:

1. **Locked rows** (any row with Budget Hours > 0) are held fixed. Auto-allocate fills only empty rows.
2. **Labour Allowance** = `(Cost_Allowance − bxp) / (1 + tech_%)` ← strips out expenses and tech fee to get pure labour budget.
3. **Managerial block** (Partner + Director + SM + Manager grades) = **20% of total hours** by default.
   - Exception: if the user has manually set any managerial row, the 20% rule is skipped entirely.
4. **Partner hours**: target 2 hours if budget permits, 1 hour if tight, 0 if the budget can't absorb it.
5. **Seniors and Associates** split the remaining 80% per the selected Mix Profile:

| Profile | Senior Share | Associate Share |
|---------|-------------|----------------|
| Standard | 55% | 45% |
| Associate | 40% | 60% |
| Senior | 60% | 40% |

6. Within each group, hours are distributed by each rank's **Mix %** weight (defined in the rate card). If no mix weights are set, they're distributed equally.

---

## 7. Mercury Pre-Flight

Before submitting to Mercury (the firm's billing/tracking system), the calculator provides a pre-flight check:

| Output | Formula | What to do with it |
|--------|---------|-------------------|
| **Budget NSR** | `Σ(projected_hrs × NSR/hr)` | Enter this as "Budget NSR" in Mercury |
| **EAF (projected)** | `(fee / projected_NSR) − 1` | Health check — flag if outside ±60% |
| **Max Hrs at Target** | `Labour_Allowance / blended_cost_per_hr` | Max hours the cost allowance supports |
| **Hrs Headroom** | `Max_Hrs − Projected_Hrs` | Positive = room to add hours; Negative = overstaffed |

> After an ETC is transferred, update Mercury's Budget NSR with the new projected NSR. This keeps Mercury's NUI calculation aligned with the calculator.

---

## 8. The Target Margin Chain (Full Picture)

This is the single most important thing to understand about the system:

```
Target Margin %
      ↓
Cost Allowance = TER × (1 − Target_Margin_%)
      ↓
Labour Allowance = (Cost_Allowance − Expenses) / (1 + Tech_%)
      ↓
Auto-Allocate: fills hours until Labour cost = Labour Allowance
      ↓
Variance = Cost_Allowance − Actual_Labour_Cost
      (tracks whether you're inside the allowance as the job runs)
```

**Target Margin % does NOT affect:** NSR, ANSR, EAF, NUI, Margin $, Margin %, TER, Recognized ANSR.

Those metrics are all driven by the **Agreed Fee** and the **actual hours** deployed.

---

## 9. Key Formulas for Excel

These are the exact formulas to use in any Excel implementation. Assume:
- `B2` = Agreed Fee
- `D2` = Billable Expenses
- `F2` = Tech Fee %
- `H2` = Target Margin %
- `J2` = Charged Expenses
- `L2` = Billed Fees
- `N2` = Billed Expenses
- Rows 6–12 = staff (C=Budget Hrs, D=Cost/hr, E=NSR/hr, F=NSR$, G=ANSR$, H=Actual Hrs, I=Actual Cost$)

```
TER             = B2 + D2
Labour_Cost     = SUMPRODUCT(C6:C12, D6:D12)
Tech_Fee        = Labour_Cost * F2
Total_Cost      = Labour_Cost + Tech_Fee + D2
Margin_$        = B2 - Total_Cost
Margin_%        = IF(TER=0, 0, Margin_$ / TER)
Cost_Allowance  = TER * (1 - H2)

NSR_total       = SUM(F6:F12)          [where F = C*E per row]
EAF             = IF(NSR_total=0, 0, (B2/NSR_total)-1)

ANSR_per_person = IF(SUM(F6:F12)=0, 0, F_row/SUM($F$6:$F$12)*B2)

Actual_Cost     = SUMPRODUCT(H6:H12, D6:D12)
Variance        = Cost_Allowance - Actual_Cost

Actual_NSR      = SUMPRODUCT(H6:H12, E6:E12)
Recognized_ANSR = Actual_NSR * (1 + EAF)
NUI             = (Recognized_ANSR + J2) - (L2 + N2)
```

---

## 10. Common Misunderstandings

| Wrong assumption | Correct understanding |
|-----------------|----------------------|
| "Cost should drive the fee" | The fee is always the **input**. Cost is derived from hours. |
| "ANSR is a separate number from Fee" | Total ANSR = Agreed Fee, always. It's just the fee allocated per person by NSR weight. |
| "Target Margin drives margin" | Target Margin only drives Cost Allowance and Variance. Actual margin is determined by hours × rates vs fee. |
| "Use NSR as the billing number" | NSR is the **denominator for EAF**, not the billing amount. ANSR (= Fee) is what gets billed. |
| "EAF should be positive" | Negative EAF is completely normal. It means the engagement is priced below rack rates, which is typical on competitive or discounted jobs. |
| "The Rate column should be cost ÷ (1−margin)" | That is cost-plus pricing. This system is fee-in, cost-out. There is no "rate" column — only Cost/hr and NSR/hr as separate lookups. |
| "Budget NSR = Agreed Fee" | No. NSR = rack-rate value of hours. Fee = what client pays. EAF is the difference. NSR and Fee are only equal if EAF = 0. |

---

## 11. Prompting Claude in Excel Context

When asking Claude to build or modify formulas in the Excel template, always specify:

1. **Which row the data starts on** (e.g. "staff rows start at row 6")
2. **Which column holds what** (e.g. "C = Budget Hrs, D = Cost/hr, E = NSR/hr")
3. **Which cells hold the inputs** (e.g. "Agreed Fee is in B2, Target Margin in H2")
4. **Whether you want absolute or relative references** (e.g. "Fee should be $B$2 in all formulas")
5. **Which metrics you want updated** — don't just say "fix the cost column", say "I want the Labour Cost metric to use SUMPRODUCT(budget hrs × cost/hr) not hrs × NSR/hr"

### Example prompt structure:
> "In my Excel sheet, staff rows are 6–12. Column C = Budget Hrs, D = Cost/hr, E = NSR/hr. Cell B2 = Agreed Fee, H2 = Target Margin %. In cell A4 I want a formula for Cost Allowance. In cell B4 I want Variance = Cost Allowance minus actual labour cost, where actual hours are in column H."

---

*Last updated based on Praxis EngEconomics.jsx source — FY2025-26 and FY2026-27 rate cards, full workflow including ETC and NUI logic.*
