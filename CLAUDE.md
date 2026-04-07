# Flyhomes BBYS Mortgage Calculator: Master Project Bible

## 1. Project Overview & Persona
A "Buy Before You Sell" (BBYS) mortgage calculator for Flyhomes. Focus on unlocking equity and providing smart, scenario-based loan recommendations.

### Team Persona
- **UX Designer**: Focus on "finance-grade" UI, high-trust, and clear benefit visualization.
- **Loan Officer**: Ensure 100% math accuracy and precise lending terminology.
- **Lead Dev**: Expert in clean, single-file HTML/Tailwind/Vanilla JS.

---

## 2. 🚀 Workflow Orchestration (Design-Driven Development)

### Plan Before Building
- **Visual Plan First**: For any UI task, output a brief plan (layout, components, spacing) **before** writing code.
- **Ask, Don't Guess**: If direction is unclear, ask the designer first. Stop and re-align immediately when blocked.

### Handle Token Limits Proactively
- **Phase-Based Work**: Break complex tasks into: Structure -> Logic/Math -> Interactions/UI.
- **No Over-Output**: Summarize at the end of each phase. Never try to complete all code in a single output.

### Simplicity Over Cleverness
- **Direct Approach**: Implement in the simplest way possible; avoid over-abstraction.
- **Readability**: Code readability matters more than complexity. If a solution feels convoluted, stop and ask.

### Design Consistency & Iteration
- **Reuse Tokens**: Strictly reuse existing colors/spacing. Don't add new ones arbitrarily.
- **Iteration-Friendly**: Only touch what needs changing—don't rewrite entire files for small tweaks.
- **Intent Over Literal**: Understand the *design intent* behind visual feedback—don't just literally translate words into code.

### Visual Verification
- **Self-Check**: Before delivering, verify visual hierarchy, alignment, and responsiveness.
- **Explicit Signal**: Tell the user **"Ready to preview in browser"**—do not silently finish.

---

## 3. Branding & Design System
- **Primary Color**: `#4C7994` (Flyhomes Blue).
- **Tokens**: Success (#10B981), Warning (#F59E0B), Border (#E5E7EB).
- **Style**: Ant Design patterns, generous whitespace, high-trust minimalist UI.

---

## 4. DREAM Solutions & Recommendation Engine

### A. Challenge Selection (The 3-Option UI)
1. **DTI Issue**: "Current mortgage hurting DTI"
   - *Subtitle*: Exclude your current mortgage to qualify for more.
2. **Liquidity Issue**: "Low cash / Equity tied in home"
   - *Subtitle*: Unlock your home equity for your down payment and closing costs.
3. **Strategy/Speed**: "Need a competitive cash offer"
   - *Subtitle*: Win with an all-cash offer and 10-day close. (Also supports mortgage-free downsizing.)

### B. Trigger Rules (Solution Mapping)
- **Rule 1: The "Cross Collateral" Winner**
  - **Condition**: IF (Option 2 IS checked AND Option 3 IS checked)
  - **Result**: Show **Cross Collateral** ONLY. (Hide IE/CO/GBC cards as CC is the integrated $0 cash/downsize king).
- **Rule 2: Equity Focused**
  - **Condition**: IF (Option 2 checked ONLY) OR (Option 1 + 2 checked)
  - **Result**: Show **Equity for Down Payment** (IE) + **DTI Buster** (GBC).
- **Rule 3: Cash Offer Focused**
  - **Condition**: IF (Option 3 checked ONLY) OR (Option 1 + 3 checked)
  - **Result**: Show **All-Cash Advantage** (CO) + **DTI Buster** (GBC).
- **Rule 4: Pure DTI**
  - **Condition**: IF (Option 1 checked ONLY)
  - **Result**: Show **DTI Buster** (GBC) only.

### C. UI Badges & Labels
- **"Most Likely Needed"**: Apply this badge to the **DTI Buster** card if Option 1 was **NOT** manually checked but Rule 2 or Rule 3 triggered it. 
- **Downsize Hint**: If Option 3 is active, ensure UI emphasizes: "Perfect for mortgage-free downsizing."

---

## 5. Calculation Logic (Source of Truth)

### A. Total Estimated Upfront Cost Formula
`Total Estimated Cost = (Loan Amount × Origination Fee %) + (Loan Amount × Broker Fee %) + GBC Fee`

### A1. Broker Fee % by Product
- **Cash Offer**: 1%
- **Instant Equity**: 0.5%
- **Cross Collateral**: 1%

### B. Origination Fee % Rules
- **Cash Offer**:
  - If **LTV 90.01%–95.00%**: 1.5%
  - If **LTV <= 90.00%**: 1.0%
- **Instant Equity**:
  - **1st Lien**: 2.0%
  - **2nd Lien**: 2.5%
  - High Cost loans are not permitted.
- **Cross Collateral** (LTV based on Acquiring Property):
  - If **LTV > 90.00%**: 1.5%
  - If **LTV <= 90.00%**: 1.0%

### C. GBC Fee Matrix (Standalone vs. Bundle)
Determined by the **Departing Home Price**. Standalone applies if no Flyhomes loan is selected.
*Note: Cross Collateral CANNOT be combined with GBC Fee.*

| Guaranteed Price Range | Standalone Fee (No Loan) | Net Bundle Fee (With Loan) |
| :--- | :--- | :--- |
| Up to $500,000 | $2,500 | **$2,500** |
| $500,001 - $750,000 | $3,500 | **$2,500** |
| $750,001 - $1,000,000 | $5,000 | **$2,500** |
| $1,000,001 - $1,500,000 | $7,500 | **$5,000** |
| $1,500,001 - $2,000,000 | $10,000 | **$7,500** |
| Above $2,000,000 | $10,000+ | **$10,000+** |

### D. Core Product Formulas
- **Instant Equity**: `(Departing Price * 0.78 * 0.9) - Current Mortgage`. Max LTV: 90% of GBC Price.
- **BBYS + Cash Offer**: `New Home Price * 95%`. Max LTV: 95%. Min Down Payment: 5%.
- **Cross Collateral**: `(New + Departing) * 0.75 - Current Mortgage`. Max LTV: 105% (Acquiring) | 75% CLTV (Combined).

---

## 6. Technical Implementation
- **Single File**: All CSS (Tailwind), HTML, and JS in `index.html`.
- **Reactivity**: Use a single state object `S`. Any input change triggers `calc()` -> `render()`.
- **UX**: Currency formatting on blur; bidirectional slider sync; keyboard-accessible tooltips.

## 7. Capture & Learn
- **Self-Correction**: After any correction on visuals or structure, **update this file** with the new rule.
- **Goal**: The same class of mistake must not repeat.