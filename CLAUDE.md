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
- **Primary CTA**: Background `#D9848B` (pink), text `#232226` (near-black). Hover `#CB7178`. Apply to ALL primary CTAs (e.g., nav "Get Started", "Submit scenario to get official offer"). Do not use the brand blue for primary CTAs.
- **Top Nav**: Dark background `#1a1a1a`, white Flyhomes wordmark logo, gray-200 link text, hover white.
- **Typography**: `Libre Franklin` (Google Fonts, weights 300/400/500/600/700) is the project font. Do not use Inter.
- **Tokens**: Success (#10B981), Warning (#F59E0B), Border (#E5E7EB).
- **Style**: Ant Design patterns, generous whitespace, high-trust minimalist UI.

---

## 4. DREAM Solutions & Recommendation Engine

### A. Challenge Selection (The 4-Option UI)
**Confirmed 2026-08-14**: card copy below updated per compliance/copy spec (`BBYS_Calculator_Copy_Spec_8_13.pdf`) — global no-dash rule (no em dashes, en dashes, or hyphens in visible copy), plain buyer-first pain framing. Solutions/rules unaffected.
1. **DTI Issue**: "My current mortgage is capping my approval"
   - *Subtitle*: You cannot carry two mortgages at once, so your approval on the new home comes in lower than you need.
2. **Liquidity Issue**: "My cash is locked in my current home"
   - *Subtitle*: Your down payment and closing costs are sitting in your current home, stuck until it sells.
3. **Strategy/Speed**: "My offers keep losing to cash buyers"
   - *Subtitle*: You wish your offer could be as strong as cash, winning without having to bid the highest. (Also supports mortgage-free downsizing.)
4. **Retirement Focus**: "I want to downsize, ideally without a new mortgage"
   - *Subtitle*: You have plenty of home equity but not enough income to qualify for a new loan the usual way.

### B. Trigger Rules (Solution Mapping)
- **Rule 0: The Downsize Override (Priority)**
  - **Condition**: IF (Option 4 IS checked)
  - **Result**: Show **Retire & Downsize** solution ONLY. This selection overrides all other logic combinations — **except Rule 5** (Cross-State Block), which still applies to downsize since Retire & Downsize is a Cross Collateral product and CC requires same-state homes. See Rule 5.
- **Rule 1: The "Cross Collateral" Winner**
  - **Condition**: IF (Option 2 IS checked AND Option 3 IS checked)
  - **Result**: Show **Cross Collateral** ONLY. (Hide IE/CO/GBC cards as CC is the integrated $0 cash/downsize king).
- **Rule 2: Equity Focused**
  - **Condition**: IF (Option 2 checked ONLY) OR (Option 1 + 2 checked)
  - **Result**: Show **Equity for Down Payment** (IE) + **DTI Buster** (GBC).
- **Rule 3: Cash Offer Focused (Auto-Gating with Toggle Override)**
  - **Condition**: IF (Option 3 checked ONLY) OR (Option 1 + 3 checked)
  - **Evaluation**: 
    - Check the state of the "Keep More Cash" override flag (`S.forceCashOffer`).
    - Calculate `projectedFinalLTV = (NewHomePrice + CurrentMortgage - (DepartingPrice * 0.9)) / NewHomePrice`.
  - **Result**: 
    - IF `S.forceCashOffer` is TRUE: Show **All-Cash Advantage** (CO) + **DTI Buster** (GBC) immediately.
    - IF `S.forceCashOffer` is FALSE AND `projectedFinalLTV <= 0.75`: Show **Cross Collateral** ONLY.
    - IF `S.forceCashOffer` is FALSE AND `projectedFinalLTV > 0.75`: Show **All-Cash Advantage** (CO) + **DTI Buster** (GBC), **UNLESS** the cash-shortfall fallback below applies.
  - **Cash-shortfall fallback to Cross Collateral (Confirmed 2026-08-14)**: When this rule would resolve to Cash Offer (`projectedFinalLTV > 0.75`) but the buyer's liquid cash (`assets`) can't cover the 5% down payment on the new home, check whether Cross Collateral would actually get the buyer all the way there: `CC Max Loan + assets >= NewHomePrice`, where `CC Max Loan = MIN(NewHomePrice * 1.05, (NewHomePrice + DepartingPrice) * 0.75 - CurrentMortgage)`. If yes, show **Cross Collateral** instead. **This is a gap-coverage check, not a fixed percentage** — Cross Collateral has no minimum-down-payment rule of its own (unlike Cash Offer's 5%), so the test is simply "does the CC loan plus the buyer's actual cash cover the purchase," not "does the CC loan clear some flat threshold like 95%." (An earlier version of this rule checked `CC Max Loan >= NewHomePrice * 0.95`, matching Cash Offer's own qualifying bar — that undercounts a real shortfall whenever the CC loan clears 95% but still leaves a gap bigger than the buyer's cash, e.g. a $35K gap against $10K cash. Replaced with the gap-coverage check above.) If Cross Collateral would ALSO leave an uncovered gap, do NOT switch — keep showing **All-Cash Advantage** (CO) + **DTI Buster** (GBC) with the existing cash-shortfall warning, since swapping to an equally-inadequate Cross Collateral loan (with no warning UI for that shortfall) would be more confusing, not less.
- **Rule 5: Cross-State Block**
  - **Condition**: IF the user selects different states for "What state is your current home in?" and "What state will you be buying in?" (both fields populated, values differ)
  - **Result**: Do NOT recommend **Cross Collateral** regardless of any other rule, **including Rule 0 (Downsize Override)**. Cross Collateral requires both homes to be in the same state — no exception for downsize.
    - **Rule 0 downsize blocked**: Fall back to **Instant Equity** (IE) + **DTI Buster** (GBC). Downsize alone (Option 4, no Option 3) carries no competitive-cash-offer intent — its core value prop is "use equity toward the new home," which Instant Equity preserves. Cash Offer does not fit here since it solves a different problem (winning a bidding war) that a pure-downsize user never asked for.
    - **Rule 1 equity+cash combo blocked**: Fall back to **All-Cash Advantage** (CO) + **DTI Buster** (GBC), since the user explicitly selected Option 3 (competitive cash offer).
    - Fall back to **DTI Buster** (GBC) alone for DTI-only intent.
  - **Confirmed 2026-08-13** (fixes §6.1 bug): when Rule 5 blocks a downsize-triggered CC, the UI must also switch its title/badge/subtitle away from "Retire & Downsize"/"Cross Collateral" to match whatever product actually gets computed — the label must always be derived from the resolved product, never from the raw Option 4 checkbox alone. Do not maintain the solution-card label in two separate places (e.g. once from the resolved calc() branch, once again from raw checkbox state) — that duplication is what let the label and the math disagree.
- **Rule 4: Pure DTI**
  - **Condition**: IF (Option 1 checked ONLY)
  - **Result**: Show **DTI Buster** (GBC) only.

### C. UI Badges & Labels
- **"Most Likely Needed"**: Apply this badge to the **DTI Buster** card if Option 1 was **NOT** manually checked but Rule 2 or Rule 3 triggered it. 
- **Downsize Hint**: If Option 3 is active, ensure UI emphasizes: "Perfect for mortgage-free downsizing."

### D. Scenario Branding: "Retire & Downsize"
- **Trigger**: Option 4 selected.
- **UI Title**: "Retire & Downsize"
- **Product Badge**: "Cross Collateral"
- **Main Subtitle**: "Leverage home equity to downsize for retirement and enjoy greater financial freedom, often without taking on a new long-term mortgage."
- **Logic Sync**: This is a marketing skin for the **Cross Collateral** product. It must use all formulas and data points defined for Cross Collateral in Section 5.D.

---

## 5. Calculation Logic (Source of Truth)

### A. Total Estimated Upfront Cost Formula
`Total Estimated Cost = Origination Fee + Broker Fee + GBC Fee + Accrued Interest`

**Confirmed 2026-08-14** (raised by engineering — two archived Notion doc variants disagreed with each other and with the prototype, one omitting Broker Fee and the other omitting Accrued Interest): the prototype's formula is the correct, complete one — all four terms are part of the total. Broker Fee is never broken out as its own visible line item though; per §5D it's bundled into the "One Time Fees" display alongside Origination Fee. Do not use a total formula that's missing any of these four terms.

**Accrued Interest**
- Rate: 9.99% per annum
- Formula: `Loan Amount × 9.99% × (transitionDays / 365)`
- Only shown when `Loan Amount > 0` (hidden in DTI-only scenario where there is no loan).
- Displayed as a separate line item in the Estimated Cost dropdown alongside "One Time Fees" (Origination Fee + Broker Fee bundled together).

### B. Origination Fee % Rules
- **BBYS + Cash Offer & Cross Collateral**:
  - If **LTV > 90.00%**: 1.5%
  - If **LTV <= 90.00%**: 1.0%
- **Instant Equity**:
  - If **1st Lien**: 2.0% | **2nd Lien**: 2.5%
  - **Confirmed 2026-08-13**: this calculator is a preliminary/directional estimate, not a bindable quote, so lien-position detection is intentionally NOT implemented. The app has no lien-position input and none of the existing fields (e.g. `Current Mortgage Balance`) reliably imply it — whether an existing mortgage forces 2nd lien or gets paid off/subordinated to keep the new loan in 1st position depends on the specific lending product structure, which this app doesn't model. `originationRate` is hardcoded to `0.02` (1st lien) everywhere (index.html:1245). Do not "fix" this by inferring lien position from `Current Mortgage Balance` without a new product decision — that was explicitly considered and rejected as unreliable.

### C. GBC Fee Matrix (Standalone vs. Bundle)
Determined by the **Departing Home Price**. Standalone applies if no Flyhomes loan is selected.
*Note: Cross Collateral CANNOT be combined with GBC Fee.*

**Formula:** `Net Bundle Fee = Standalone Fee − Bundle Credit`

| Guaranteed Price Range | Standalone Fee (No Loan) | Bundle Credit | Net Bundle Fee (With Loan) |
| :--- | :--- | :--- | :--- |
| Up to $500,000 | $2,500 | $0 | **$2,500** |
| $500,001 – $750,000 | $3,500 | $1,000 | **$2,500** |
| $750,001 – $1,000,000 | $5,000 | $2,500 | **$2,500** |
| $1,000,001 – $1,500,000 | $7,500 | $2,500 | **$5,000** |
| $1,500,001 – $2,000,000 | $10,000 | $2,500 | **$7,500** |
| Above $2,000,000 | Exception | Exception | **$10,000+** (exception — requires manual handling; not auto-calculated) |

**Exception rule for Above $2,000,000:** Both the Standalone Fee and Bundle Credit fall outside the standard tier table. The system must flag this as an exception. Default display value is $10,000+, but the actual fee must be confirmed manually and is not programmatically computed.

**Confirmed 2026-08-13 — input is capped at $2,000,000, not left open with an exception state:** "Estimated Value of Your Current Home" (`S.homeValue`, index.html:319-326) has its slider max set to `2,000,000` (not $3M). If the user types a value above $2,000,000 directly into the text field, `render()` (index.html:1543) hides the entire `#results-panel` and shows "Please contact us to learn more." under the field instead of computing an estimate. This avoids ever hitting the >$2M GBC exception path in a way that silently produces an incomplete total (see §6.5-style gap) — above $2M, the tool simply doesn't attempt an estimate.

### D. LO Broker Fee Rules (Estimated Average)
Programmatically derive the Broker Fee based on the active product track and its corresponding loan/advance base amount:

- **BBYS + Cash Offer Track**: 
  `BrokerFee = FCO_LoanAmount * 0.01` (1% of the standalone Cash Offer loan amount)
  
- **Instant Equity Track**: 
  `BrokerFee = IE_AdvanceAmount * 0.005` (0.5% of the unlocked equity advance amount)
  
- **Cross Collateral Track**: 
  `BrokerFee = CC_CappedLoanAmount * 0.01` (1% of the final dual-capped Cross Collateral loan amount)

*UI & Multi-panel Guidelines:*
1. **Encapsulation**: These calculated fees must not be shown as a standalone raw line item to the buyer. They must be rolled into the `One Time Fees` parent element (previously labeled "Other Costs" — renamed per the 2026-08-13/14 copy spec).
2. **Tooltip Rendering**: Inside the `One Time Fees` disclosure tooltip, map this value to the professional label: `Loan Processing & Underwriting: $[Calculated_Value]`.
3. **Reactivity**: Any change to input variables (e.g., purchase price or departing home value) that shifts the underlying loan amount must instantly re-run these percentage models to keep the totals synchronized.

### E. Core Product Formulas
Instant Equity: (Departing Price * 0.75 * 0.9) - Current Mortgage. Max LTV: 90% of GBC Price.

Cash Offer eligibility (`cashEligible`): Liquid Assets >= New Home Price * 5%. Ensures the 5% Down Payment is funded by actual liquid cash.

**Confirmed 2026-08-14**: Cash Offer's down payment must come from liquid cash only — home equity does NOT count toward it, since that equity is still locked in the departing home and isn't accessible to the buyer unless they're also using Instant Equity or Cross Collateral. Do not combine assets with unlockable home equity (previously called `combinedResources` in code) for Cash Offer — that was a bug, since a buyer with little cash but high home equity would be shown as Cash Offer eligible despite having no way to actually fund the down payment.

BBYS + Cash Offer: New Home Price * 95%. Max LTV: 95%. Min Down Payment: 5%. Only computed once `cashEligible` (above) has already passed.

**History (resolved 2026-08-14)**: engineering flagged that the prototype's original code (`MIN(New Home Price, Max Purchase Price) * 95%`, with `Max Purchase Price = combined cash + equity / 0.05`) diverged from the design doc's flat `New Home Price * 95%`. Two things got resolved in sequence:
1. The "Max Purchase Price" cap should use liquid cash only, never home equity (equity isn't accessible for a Cash Offer down payment — see the confirmation above).
2. Once liquid-cash-only `cashEligible` gates entry into this branch at all (`Liquid Assets >= New Home Price * 5%`), the `MIN(New Home Price, Max Purchase Price)` cap became mathematically redundant — `cashEligible` passing already guarantees `Max Purchase Price >= New Home Price`, so the MIN always resolves to `New Home Price` and never actually caps anything. The cap was removed; eligibility is checked once, and the loan amount is the plain formula above. Net effect on output: none — this only simplifies the formula, it doesn't change any computed number.

Cross Collateral: MIN((New Home Price * 1.05), ((New Home Price + Departing Price) * 0.75 - Current Mortgage)). Hard risk caps: 105% Acquiring LTV maximum and 75% Combined CLTV maximum.


### F. Estimated Savings Logic
Trigger: Display in "Review Results" section to replace the old "Moving Once" module.

Data Provenance:

departingHomePrice: Live-synced from Step 2 "Estimated Departing Home Value" input/slider.

newHomePrice: Live-synced from Step 2 "Estimated Purchase Price" input/slider.

isCashOfferSelected: Boolean flag from Step 1 selection.

1. Cash Offer Price Advantage (Conditional)
Visibility: Show whenever the recommended product uses a cash offer — i.e. `S.cash` (BBYS + Cash Offer track) OR `S.noCash` (Cross Collateral, including Retire & Downsize) is true. Do NOT gate this further on a standalone liquid-asset check (e.g. `assets >= newPrice * 0.05`) — that duplicates the same liquid-cash eligibility check `calc()` already runs to decide whether the Cash Offer branch applies in the first place, and re-checking it again here can hide the benefit from a user who was legitimately recommended the Cash Offer product. (Confirmed 2026-08-13, fixing a prior undocumented gap where this exact liquid-asset re-check suppressed the row. Note: as of 2026-08-14, Cash Offer eligibility itself is liquid-cash-only too, per §5E — see that section for the current formula.)

Logic: Default 3.5% (Slider: 1% - 10%).

Formula: `newHomePrice * cashOfferDiscount%`

Tooltip: "A cash-strong offer can win at a lower price than a financed one. Drag the slider to set how big a discount to estimate; our 3.5% default is the conservative floor."

2. Staged Home Premium
Logic: Default 5% (Slider: 1% - 10%).

Formula: departingHomePrice * stagedHomePremium%

Tooltip: "Staging an empty home can lift the sale price. This shows the estimated increase minus a typical $3,500 staging cost."

3. Home Price Appreciation (HPA)
Logic: Default 4% (Slider: -10% - 10%).

Formula: `newHomePrice * (hpaRate% / 12) * (transitionDays / 30)`

Note: Buy-side only — counts only the price increase avoided on the new purchase. The sell-side gain is excluded. Driven by the user-editable `transitionDays` field (default 60 days).

Tooltip: "Based on the annual price-change rate you set. This counts only the buy-side savings from acting now — we always recommend selling your current home as soon as possible."

4. "Move Once, Not Twice" (Logistics)
Layout: Single accordion item containing three editable text fields.

Tooltip: "Estimated savings are based on the extra cost of moving twice and two months of transition, compared with moving only once."

Sub-Items (Auto-calculated but editable):

Extra Moving Fee: (departingHomePrice * 0.003) + 500

Temporary Housing: (departingHomePrice * 0.006) * (transitionDays / 30)

Storage Fee: (150 + departingHomePrice * 0.0002) * (transitionDays / 30)

Category Total: Sum of the three sub-fields above.

transitionDays: User-editable input field in the results panel (above Estimated Cost). Default: 60. Minimum: 1. Affects HPA, Temporary Housing, and Storage Fee calculations.

5. Grand Total
Calculation: Sum of all active items (Note: Cash Offer Discount = 0 if hidden).

UI: Display "Total Estimated Savings" prominently at the bottom of the section.

---

## 6. Technical Implementation
- **Single File**: All CSS (Tailwind), HTML, and JS in `index.html`.
- **Reactivity**: Use a single state object `S`. Any input change triggers `calc()` -> `render()`.
- **UX**: Currency formatting on blur; bidirectional slider sync; keyboard-accessible tooltips.

## 7. Capture & Learn
- **Self-Correction**: After any correction on visuals or structure, **update this file** with the new rule.
- **Goal**: The same class of mistake must not repeat.