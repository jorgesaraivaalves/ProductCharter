# [PRD/Charter]: Native / Custom Keypad
 | Metadata | Value |
| :--- | :--- |
| **Product Owner** | Jorge Alves | Simon Burton |
| **Tech Lead** | Ricardo Pereira | Gabriel Pires |
| **Target Branch / Dir**| `src/features/[feature-name]/` or `src/components/[scope]/` 
| **Platforms** | iOS & Android (React Native / Native App Container) |
| **Tracking Ticket** | SSTDBT-63 |
| **Document Version** | '1.0.0' |
| **Status** | [Draft] |
 
---

## 1. Problem Statement
### 1.1 User Friction
* **Unnatural 6×2 Keypad Layout:** The legacy layout spans digits across two wide rows (`1–5 + ⌫` and `6–0 + .`), breaking universal mobile ergonomic conventions and forcing users to visually search for numbers. 
* **Narrow Touch Targets & High Error Rate:** Squeezing 6 columns across mobile screen widths creates cramped touch targets, driving accidental mis-taps, unintended deletions, and decimal errors during fast-paced in-play betting. 
* **Action Key Clutter:** Top-right placement of the backspace key and non-standard decimal positioning increase cognitive friction and stake correction time. 

### 1.2 Business Inefficiency
* **Conversion Drop-off:** Slower stake entry and mistypes cause customers to miss fast-moving in-play odds or abandon bet slips entirely
* **Brand & Experience Fragmentation:** A non-standard keypad layout deviates from modern betting UX patterns across the wider Flutter portfolio

---

## 3. Goals & Intended Outcomes
### 3.1 Primary Objectives
* **Standardized Ergonomics:** Deploy a standard 3×4 numerical grid with enlarged, thumb-friendly touch targets
* **Input Fluidity:** Deliver sub-50ms render latency with zero viewport jumping and seamless keyboard avoidance
* **Domain Ergonomics:** Provide native-grade decimal handling (max 2 d.p.), hold-to-clear backspace, and integrated quick-stake chips (+£5, +£10, +£20)

### 3.2 Success Metrics & KPIs
* **Stake Entry Speed:** Measurable reduction in time-to-enter-stake and overall time-to-place-bet
* **Error Reduction:** Significant reduction in backspace frequency and stake correction events per session
* **Funnel Conversion:** Measurable uplift in betslip-to-bet-placed conversion across in-play and pre-match markets

---

## 4. Scope & Boundary Definition
### 4.1 In Scope
* **UI & Grid Restructuring:** 
 * Migration to 4 rows × 3 columns (`1–9`, `.` bottom-left, `0` bottom-center, `⌫` bottom-right).
 * Maintained horizontal quick-stake preset chips directly above the numeric grid. 
* **Input State & Validation Logic:**  
 * Real-time numeric stake updates with auto-prefixing `0.` for leading decimals.   
 * Prevention of duplicate decimal points
 * Backspace handling: Single tap (delete last char) and Long press (>= 800ms to clear all)
* **Surface Coverage:**  
 * Quick Bet / Single Bet modal bottom sheet
 * Standard Multi-Leg / Full Betslip view. 

### 4.2 Out of Scope
* **Core Betting Services:** Downstream Bet Placement Services (BPS), pricing streams, or wallet balance APIs. 
* **Odds Calculation & Return Models:** Multiplier calculations, accumulator rules, or pricing models. 
* **Unrelated Workstreams:** My Bets / Open Bets, Oasis live bet tracking, cash-out pebbles, and token modals. 
* **Desktop & Web Views:** Physical keyboard events or desktop browser popovers. 
* **Alphanumeric Inputs:** Promo codes, search bars, or free-form text fields. 

---

## 5. UI Architecture & Layout Comparison
 | Element | Legacy Implementation (6×2) | New Standard Implementation (3×4) | 
| :--- | :--- | :--- | 
| **Grid Dimensions** | 2 Rows × 6 Columns | **4 Rows × 3 Columns** | 
| **Row 1** | `1`, `2`, `3`, `4`, `5`, `⌫` | `1`, `2`, `3` | 
| **Row 2** | `6`, `7`, `8`, `9`, `0`, `.` | `4`, `5`, `6` | 
| **Row 3** | — | `7`, `8`, `9` | 
| **Row 4** | — | `.` (Left), `0` (Center), `⌫` (Right) | 
TBR -> | **Touch Target Width** | ~50px (Cramped) | **~100px+ (Standard Thumb Zone)** | 

---  

## 6. Executable Acceptance Criteria / Gherkin Acceptance Scenarios
 ```gherkin

Scenario: Single bet -> Open betslip -> Open keypad on stake input focus (alwauys on)

Scenario: Multiple bet -> Open betslip -> Open keypad on stake input focus

Scenario: Delete selection from multiple

Scenario: Single bet -> Open betslip -> Open keypad on stake input focus

Scenario: Betbuilder bet -> Open betslip -> Open keypad on stake input focus


Scenario: Single Stake field gets focus -> open keypad
  Single stake field in view

Scenario: Multiple Stake field in view gets focus -> open keypad
  Adjust betslip viewport

Scenario: Stake field lost focus -> close keypad

Scenario: betslip upsell -> add selection

Scenario: tabbed betslip navigation

Scenario: quick stake

Scenario: Eac



Scenario: Open keypad on stake input focus
  Given the customer has added a selection to the Quick Betslip or Standard Betslip
  And the stake input field is empty
  When the customer taps on the stake input field
  Then the numeric keypad is displayed from the bottom of the viewport
  And the stake input field is highlighted in an active/focused state
  And the viewport adjusts so that the stake field, potential returns, and "Place Bet" CTA remain visible
