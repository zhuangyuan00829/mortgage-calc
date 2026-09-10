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
  - **Cash-shortfall warning copy (Confirmed 2026-09-09)**: shown in the `#cash-warning` banner (Step 1 card, below the challenge checkboxes) whenever `S.cash && !r.cashEligible` — i.e. the buyer still has cash intent (Option 3) but doesn't clear the 5% liquid-cash bar, and Cross Collateral doesn't cover the gap either (see fallback above), so `calc()`'s else-if chain falls through past the Cash Offer branch to the Instant Equity branch and that becomes the actual resolved/displayed main solution (`r.solutionName`), even though `S.cash` stays `true` as a record of the user's original cash intent. Copy: "All-Cash Advantage solution requires a down payment of at least 5%. Based on your inputs, you are $[gap] short of the requirement, so we recommend [r.solutionName] instead. To use the All-Cash Advantage solution, please adjust the amount entered for "Cash and Liquid Assets Available."" — `[gap]` is `r.cashThreshold - S.assets` and `[r.solutionName]` is the actually-resolved product name (e.g. "Equity for Down Payment"), so the message always names whatever is genuinely shown as the main solution card, never a hardcoded product.
- **Rule 5: Cross-State Block**
  - **Condition**: IF the user selects different states for "What state is your current home in?" and "What state will you be buying in?" (both fields populated, values differ)
  - **Result**: Do NOT recommend **Cross Collateral** regardless of any other rule, **including Rule 0 (Downsize Override)**. Cross Collateral requires both homes to be in the same state — no exception for downsize.
    - **Rule 0 downsize blocked**: Fall back to **Instant Equity** (IE) + **DTI Buster** (GBC). Downsize alone (Option 4, no Option 3) carries no competitive-cash-offer intent — its core value prop is "use equity toward the new home," which Instant Equity preserves. Cash Offer does not fit here since it solves a different problem (winning a bidding war) that a pure-downsize user never asked for.
    - **Rule 1 equity+cash combo blocked**: Fall back to **All-Cash Advantage** (CO) + **DTI Buster** (GBC), since the user explicitly selected Option 3 (competitive cash offer).
    - Fall back to **DTI Buster** (GBC) alone for DTI-only intent.
  - **Confirmed 2026-08-13** (fixes §6.1 bug): when Rule 5 blocks a downsize-triggered CC, the UI must also switch its title/badge/subtitle away from "Retire & Downsize"/"Cross Collateral" to match whatever product actually gets computed — the label must always be derived from the resolved product, never from the raw Option 4 checkbox alone. Do not maintain the solution-card label in two separate places (e.g. once from the resolved calc() branch, once again from raw checkbox state) — that duplication is what let the label and the math disagree.
  - **Removed 2026-09-09**: the solution card no longer shows a "(Same state purchase and sale required.)" note under the title for Cross Collateral products (both the plain "Move with $0 out of pocket" card and the "Retire & Downsize" skin) — removed per design request on both. Rule 5's block is already enforced in the calc logic itself (a cross-state selection can never resolve to Cross Collateral in the first place), so the UI disclaimer was redundant. Do not re-add a `cc-same-state-note`-style element without checking this note first.
- **Rule 4: Pure DTI**
  - **Condition**: IF (Option 1 checked ONLY)
  - **Result**: Show **DTI Buster** (GBC) only.

### C. UI Badges & Labels
- **"Most Likely Needed"**: Apply this badge to the **DTI Buster** card if Option 1 was **NOT** manually checked but Rule 2 or Rule 3 triggered it. 
- **Badge copy (Confirmed 2026-08-21, updated 2026-09-08)**: this auto-triggered DTI Buster badge reads "Often Paired With" (was "Optional Add-On", was "Optional"). Title case, not uppercase — same font style as the "Recommended" tag below (no `uppercase tracking-wider`).
- **Rule 2 combo ordering (Confirmed 2026-08-21, extended 2026-09-08)**: When **Equity for Down Payment** (IE) + **DTI Buster** (GBC) are shown together (Rule 2), Equity for Down Payment renders FIRST with a "Recommended" tag, and DTI Buster renders SECOND tagged "Often Paired With". **As of 2026-09-08 this same treatment also applies to Rule 3's All-Cash Advantage + DTI Buster combo**, with different GBC-side copy: All-Cash Advantage renders FIRST tagged "Recommended" (same shared `#core-badge` element/copy as the IE combo), and DTI Buster renders SECOND tagged **"Required"** (not "Often Paired With") — DTI Buster's badge always reads "Required" whenever it's paired with All-Cash Advantage, regardless of whether Option 1 was manually checked or auto-triggered (unlike the generic "Often Paired With" case, which only shows when DTI was auto-triggered — see the "Most Likely Needed" rule above). Other combos that show DTI Buster alongside a main solution besides these two do not get any main-solution tag and keep DTI Buster first. Implemented via a `reorder-main-first` class (renamed from `reorder-equity-first` 2026-09-08 since it now serves two combos) toggled on `#solution-card-wrapper` in `applyProductLabel()` (index.html), driven by CSS `order` rather than moving DOM nodes. The "Recommended" tag (formerly "Core", renamed 2026-09-08) and "Required"/"Often Paired With" tags all reuse the same pink pill style (`background:#FFF7F8;color:#D87486;border-color:#D87486`) — do not give any of them a distinct color. **All of these tags are title case, not uppercase/tracked** (no `uppercase tracking-wider`) — at title-case width "Recommended" fits on the same line as the product title; the all-caps form was wide enough to force a wrap onto its own line.
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
- **Representative APR (Confirmed 2026-09-10)**: a line reading "APR: [rate]" is shown directly below the "No monthly payments..." sub-copy, whenever the Accrued Interest row itself is shown (i.e. `Loan Amount > 0`). Rate depends on the resolved product track (`r.productName`), not the branded UI solution name: **Instant Equity → 14.35%**; **Cash Offer and Cross Collateral (including the "Retire & Downsize" skin) → 12.35%**. Since "Retire & Downsize" and "Move with $0 out of pocket" are both just UI labels for the same underlying Cross Collateral product (§4D), they share the same 12.35% rate — do not key this off `solutionName`, which would require listing both labels separately and risks drifting if a new UI skin is added; key off `productName` instead, which only has three possible values.

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

**Fixed bug (2026-09-09) — GBC fee was being silently dropped from the total whenever DTI Buster was auto-triggered rather than manually checked**: `S.gbc_base` (index.html, `syncStateFromChallenges()`) used to be `selected.has('dti') && !isCrossCollateral` — i.e. the GBC product fee was only added to "One Time Fees" / Estimated Cost / Estimated Total Value if the user had personally ticked Option 1, even in scenarios (Rule 2, Rule 3) where DTI Buster is shown as part of the recommended combo regardless of whether Option 1 was checked (see §4C "Most Likely Needed" — that badge is about how the recommendation was *arrived at*, not about whether the fee applies). Confirmed via direct comparison: identical inputs, only difference being Option 1 checked vs. not, produced Estimated Cost totals $5,000 apart (a full bundle GBC fee) for the exact same recommended Equity for Down Payment + DTI Buster combo — and the same gap reproduced for Rule 3's All-Cash Advantage + DTI Buster combo, where DTI Buster is tagged "Required" yet its fee was still being omitted unless manually checked.
  - **Second instance found and fixed the same day**: the "Switch to Cash Offer" override (`S.forceCashOffer`, the banner offered whenever Cross Collateral is the natural Rule 1/Rule 3 result — see §5F "Estimated Savings Logic" context and `renderSwitchBanner()`/`toggleForceCashOffer()` in index.html) resolves the product to All-Cash Advantage + DTI Buster without ever calling `syncStateFromChallenges()` again, so the original `gbc_base` (computed for the *natural* Cross Collateral result, where it's correctly `false`) stayed stale and kept suppressing the fee even after the override flipped the actual product away from Cross Collateral. Confirmed via direct invocation of the real `toggleForceCashOffer()` handler on a Rule 1 (equity+cash) scenario: before the fix, `{noCash:false, cash:true, gbc:false}` — DTI Buster's card visible (`noCash:false`) but its fee suppressed (`gbc:false`).
  - **Root fix**: removed `S.gbc_base` entirely. `S.gbc` is now computed once, at the very end of `applyRule3Gating()`, as `!S.noCash` — derived from the same, always-final `S.noCash` flag that `showGBC` (card visibility) already keys off, so the fee can never disagree with whether the DTI Buster card is actually shown, in any branch (Rule 0 downsize, Rule 1, Rule 3's dynamic LTV auto-gating, Rule 5's cross-state fallback, or the forceCashOffer override) — no separately-tracked flag left to go stale. Cross Collateral (§5C "CC cannot combine with GBC") is unaffected — it still always resolves `S.noCash = true` → `S.gbc = false`.
- **GBC fee shown as its own line item (Confirmed 2026-09-10)**: "One Time Fees" in the Estimated Cost breakdown used to bundle origination + broker + GBC fee together (`other-costs-display`). It now shows **origination + broker fee only**; the GBC fee is a separate row (`#gbc-fee-row`, right below it) shown only when `r.gbcActive`. The overall "Estimated Cost" total (`r.total`) is unchanged — same four components (origination + broker + GBC + accrued interest) — only the breakdown's presentation changed. The GBC row's label mirrors the DTI Buster badge precedence exactly (same `isCashDtiCombo`/`mostLikely`-equivalent flags, computed inline in `render()` since the row must update before the cost-breakdown panel's height is recalculated — see the code comment there for why this can't live in `applyProductLabel()`):
  - Paired with All-Cash Advantage (badge "Required"): **"Guaranteed Backup Contract Fee"** (plain).
  - Auto-triggered and not the All-Cash Advantage combo (badge "Often Paired With"): **"Guaranteed Backup Contract Fee (Optional)"**.
  - Option 1 manually checked in a non-Cash combo (no badge shown at all, e.g. Equity + DTI both manually checked): plain **"Guaranteed Backup Contract Fee"** — not documented as its own case in the original request, but follows the same "(Optional)" only when auto-suggested" rule as the badge.
- **DTI-only card's fee label matches (Confirmed 2026-09-10)**: the separate `#gbc-dti-costs` card (shown when DTI Buster is the *only* recommended solution, Rule 4) had its own independent "GBC Fee" row label — renamed to **"Guaranteed Backup Contract Fee"** to match the spelled-out label used in the combo card's `#gbc-fee-row` above. Same dollar value (`r.gbcFee`, standalone rate since no Flyhomes loan is present in this case), just the abbreviated label was inconsistent.
- **Tooltip clipping fix (Confirmed 2026-09-10)**: the DTI-only card's tooltips (DTI Buster title info icon, "Estimated Cost" info icon, and the GBC fee row's info icon) were getting cut off — visually cropped mid-sentence — because their containing cards (`#gbc-solution-card` in `.dti-solo` mode, and `#gbc-dti-costs`) had `overflow: hidden` set, which clips any `.tip-box` tooltip that extends past the card's boundary (tooltips are `position: absolute` within a `.tip` wrapper, but still get clipped by any `overflow: hidden` ancestor regardless of z-index). That `overflow: hidden` existed to hide the square corners of the dark "Estimated Total Value" accordion header, which bleeds to its container's edges via negative margins (`-mx-5 -mt-4`) to sit flush against the card's rounded corners. Fixed by removing `overflow: hidden` from both card-level CSS rules — the combo-card equivalent (`#cost-benefit-card`, which has always used `overflow-visible`) has the exact same negative-margin bleed technique on its own "Estimated Total Value" header and never needed the clip, so the DTI-only cards didn't need it either. **Do not re-add `overflow: hidden` to `#gbc-solution-card` or `#gbc-dti-costs`** to "fix" a corner-rounding nitpick — it will re-break every tooltip inside those cards; if a corner artifact is ever actually visible, scope any clip narrowly to just the specific bleeding child element, not the whole card.

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

**Loan-amount-unavailable messaging (Confirmed 2026-09-09)**: `fmtMoney()` (index.html) renders any value `<= 0` as an em dash "—", which is a display bug when the underlying loan amount is a real, legitimately-computed $0 (or near-$0) rather than a value that just hasn't loaded yet — showing a bare dash gives the buyer no explanation. Two known cases replace the "Estimated Max Loan Amount" figure with an explanatory message instead of a dash, computed as `r.loanUnavailable = r.belowIEMin || r.ccZeroLoan` in `calc()`:
  - **Instant Equity minimum loan amount**: $75,000 (`IE_MIN_LOAN` in index.html, `r.belowIEMin`). When Instant Equity is the recommended product and the computed loan amount is below this minimum, the loan isn't actually available at that amount. Message: "This loan amount is below Equity for Down Payment's $75,000 minimum. Please contact us to learn more." (uses the UI product name "Equity for Down Payment," not the internal name "Instant Equity" — buyer-facing copy should match the label shown on the card above it).
  - **Cross Collateral zero loan** (`r.ccZeroLoan`, added 2026-09-09): Cross Collateral (including the "Retire & Downsize" skin) has no stated minimum, but its dual-capped formula (`MIN(NewPrice * 1.05, (NewPrice + DepartingPrice) * 0.75 - CurrentMortgage)`) can clamp to $0 when the current mortgage balance is high relative to the combined 75% CLTV cap — not a real loan offer. Message: "This loan amount isn't available. Please contact us to learn more."
  - Both messages are plain text (not a link — removed 2026-09-09 per design request) set via `textContent` on the shared `#loan-unavailable-message` element in `render()` — deliberately say "contact us," not "AE," matching buyer-facing terminology used elsewhere in the tool (e.g. the top banner's "Your Loan Officer will review your details").
  - In either case, the Estimated Cost / Estimated Total Value card is hidden too, since those figures (origination fee, broker fee, accrued interest) are all calculated off the same unavailable loan amount and would be misleading to show. The product recommendation itself (title, link, "Recommended" tag, paired DTI Buster card) still displays normally — only the loan-amount-dependent numbers are suppressed.
  - Cash Offer isn't covered by either case — its formula (`NewPrice * 0.95`) can't realistically reach $0 since the purchase-price slider floors at $200K. If Cash Offer's `fmtMoney` dash bug is ever seen in practice, add a third `loanUnavailable` case rather than special-casing it separately.

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

### Card 3 CTA (Confirmed 2026-08-21)
- **Layout**: "Share with buyers, loan officers, agents" label → **Save as PDF** + **Share link** side by side (outline buttons, file/link icons) → "For loan officers and agents only" → primary pink **Save and continue in Partner Portal** → outline **Talk to us** (opens `https://flyhomes.com/contact` in a new tab).
- **Share link mechanism**: `shareCalculatorLink()` (index.html) serializes the inputs that drive `calc()` — `SHARE_STATE_KEYS` subset of `S` (homeValue, mortgage, newPrice, assets, forceCashOffer, flyhomesLoan, savCashPct, savStagedPct, savHpaPct, transitionDays), the `selected` challenge-option set, and the current/buying state selects — into a base64 JSON payload in the URL hash (`#s=...`). `restoreFromShareLink()` runs at page bootstrap (before `renderChallenges()`/`render()`) to decode it and repopulate `S`, `selected`, and the visible inputs, so a shared link reproduces the same numbers and recommended results for whoever opens it. Clicking "Share link" copies this URL via `navigator.clipboard.writeText`, falling back to a hidden-textarea `execCommand('copy')` for insecure/`file://` contexts, then shows a "Link copied!" toast above the button.
- **Not serialized**: `savMovingFee`/`savHousing`/`savStorage` (the editable "Move Once" sub-fields) — `render()` unconditionally recomputes these from `homeValue`/`transitionDays` via `initSavingsDefaults()` on every render pass, so a restored value would just be overwritten on the next render; restoring `homeValue` + `transitionDays` already reproduces the same defaults.

## 7. Capture & Learn
- **Self-Correction**: After any correction on visuals or structure, **update this file** with the new rule.
- **Goal**: The same class of mistake must not repeat.