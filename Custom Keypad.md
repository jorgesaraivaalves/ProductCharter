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

  Scenario Outline: User enters a valid stake using the numeric grid
    Given the user has an active betslip with an empty stake field
    When the user taps the sequence "<input>" on the custom keypad
    Then the stake field should display "<output>"
    And the potential returns should update based on "<output>"
    Examples:
      | input | output |
      | 1,0  | 10    |
      | 5, ., 5 | 5.5  |
      | ., 7, 5 | 0.75 |
      | 0, 5    | 5    |

  Scenario: User attempts to enter more than two decimal places (This is not the current behaviour)
    Given the stake field currently displays "10.55"
    When the user taps "5" on the keypad
    Then the stake field should remain "10.55"
    And no further digits should be accepted until a backspace is used  

  Scenario: User attempts to enter more than two decimal places (Current behaviour)
    Given the stake field currently displays "10.55"
    When the user taps "5" on the keypad
    Then the stake field shows "10.555" and an alert message is shown "The stake should be a multiple of <currency symbol>0.01"


  Scenario: User corrects a stake entry
    Given the stake field currently displays "125"
    When the user taps the "⌫" key once
    Then the stake field should display "12"
    When the user performs a long-press (>= 800ms) on the "⌫" key
    Then the stake field should be cleared (empty state)
    And the "Place Bet" button should be disabled

  Scenario: User uses quick-stake chips to increment stake
    Given the stake field currently displays "+ €5" or "+ £5"
    When the user taps the "+ €10" or "+ £10" quick-stake chip
    Then the stake field should display "€15" or £15"
    And the keypad should remain visible for further adjustments

  Scenario: Market suspends while user is typing
    Given the user is entering a stake on the keypad
    When the underlying market moves to a "Suspended" or "Closed" state
    Then the keypad should remain active
    But the "Place Bet" button below the keypad must transition to a disabled "Suspended" or "Closed"  state
    And the user should be able to continue editing the stake

  Scenario: User exceeds the maximum allowed character length
    Given the maximum stake character limit is set to 9 digits
    And the stake field currently displays "999999999"
    When the user taps any numeric key
    Then the input should be ignored
    And a haptic feedback or visual cue should indicate the limit has been reached

Scenario: Single bet -> Open betslip -> Open keypad on stake input focus (always on)
  Given the user has 1 selection in the betslip
  When the user opens the betslip
  Then the stake field should get focus and the keypad should be displayed



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
