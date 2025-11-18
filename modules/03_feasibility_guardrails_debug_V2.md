# Module 3 — Feasibility & guardrails critique

This module is solid and practical, but it needs sharper definitions, conflict resolution, and clearer fallback behavior to be reliably executable by an LLM travel planner without ambiguity.

---

## Clarity

- **Ambiguous “closed” criteria:** Define closure detection explicitly (e.g., “closed if the venue’s hours for the target day return closed or unknown after attempted lookup; unknown hours trigger a conservative indoor swap fallback”).
  - Justification: Without a clear trigger, the planner may inconsistently treat “unknown” as open or closed, causing brittle plans.

- **Budget scope unclear:** Specify whether the budget applies per meal, per day, or total trip, and how to compare menu price ranges versus actual spend.
  - Justification: LLMs need a deterministic comparison target; vague budget scopes cause mismatched substitutions.

- **Distance and time thresholds mixing:** The rule uses both “> 25 min or > 5 km” and adds a separate “short walks only” constraint of “<25 minutes walking distance.” Clarify how to measure and which threshold prevails when they conflict.
  - Justification: Conflicting thresholds lead to inconsistent route filtering and oscillating substitutions.

- **Weather conditions definition:** “Rain or cold season likely” should be operationalized (e.g., forecast probability thresholds, date-range heuristics for local climate).
  - Justification: A planner needs a precise trigger to swap to indoor reliably instead of subjective judgment.

- **Mobility needs vagueness:** “Step-free, short-walk options and include breaks” should define step-free verification, max segment distance, and break frequency/duration.
  - Justification: Clear parameters prevent token gestures that don’t practically reduce effort.

- **Dietary constraints matching:** Define what counts as “ensure all meals match” (menu-level tags, cuisine heuristics, or verified vegan-friendly indicators).
  - Justification: Without verification criteria, substitutions may still violate constraints.

- **Bookings reminder phrasing:** Include a standard, concise reminder template and timing (e.g., show at the activity line item).
  - Justification: Consistent placement avoids simulation and maintains user trust.

---

## Correctness

- **Preference precedence:** Integrate the change log rule explicitly: if “short walks only” is active, it overrides distance/time transfer rules and filters all activities to <25-minute walk from the base or previous stop.
  - Justification: The change log mandates prioritization; the module must reflect precedence to prevent violations.

- **Travel mode consistency:** “Add a short transit hop” should specify allowed modes (e.g., public transit or rideshare), and ensure time accounting includes transfer plus buffer.
  - Justification: Correctness requires realistic timing; otherwise the plan underestimates transit overhead.

- **Time overrun handling:** Define what gets shortened first (lunch duration vs. swapping to nearer stop) and by how much, with a maximum compression threshold.
  - Justification: Deterministic resolution keeps schedules consistent and prevents overly aggressive cuts.

- **Indoor swap parity:** For closed or weather swaps, require similarity in category (e.g., “museum → gallery or exhibit space; park → conservatory or covered market”) and proximity constraints.
  - Justification: Preserves plan intent and user expectations while handling feasibility constraints.

---

## Robustness

- **Missing data fallbacks:**
  - **Venue hours unknown:** Assume potential closure; propose an indoor alternative and mark the original as “optional if open.”
  - **Menu price unknown:** Use cuisine median pricing and user budget; if still uncertain, choose budget-friendly casual options.
  - **Distance/time unknown:** Estimate walking time via standard pace and apply conservative selection; mark with an uncertainty note.
  - Justification: Graceful degradation avoids hard failures and keeps plans usable.

- **Negative or zero budgets:** If budget ≤ 0, default to free or very low-cost options and add a note that meals are observational (coffee/snack) or skipped.
  - Justification: Prevents impossible recommendations and sets user expectations.

- **Conflicting constraints:** Define tie-breakers when constraints collide (e.g., short walks only vs must-see distant venue: prioritize short walks and offer the must-see as an optional add-on with transit).
  - Justification: Explicit conflict resolution avoids flip-flopping substitutions.

- **Seasonality and weather uncertainty:** If forecast confidence is low, include a dual-path: indoor-first plan with an optional outdoor alternative.
  - Justification: Robust to volatile conditions without overcommitting.

- **Mobility buffers:** Add timing buffers for elevators, accessible routes, and seating breaks; avoid venues with known stair-only access unless explicitly accepted.
  - Justification: Real-world feasibility for mobility needs requires time and access buffers.

---

## Recommended refinements

### Rule scoping and precedence
- **Lead rule:** If user_preference includes “short walks only,” enforce all activities within a <25-minute walk from the previous stop or a central anchor; do not exceed via transit unless explicitly allowed. When infeasible, replace far activities with closer alternatives.
  - Justification: Aligns with change log and prevents accidental rule bypass.

- **Transfer constraint harmonization:**
  - **Primary:** Enforce <25-minute walking transfers.
  - **Secondary:** If walking exceeds 25 minutes, allow a single short transit hop with total transfer time (walking + transit + buffer) ≤ 25 minutes; distance threshold of ≤ 5 km applies only when transit is used.
  - Justification: Removes ambiguity between time and distance and defines transit fallback clearly.

### Deterministic triggers
- **Closed venue:** Trigger if hours show “closed” for the target day or hours are unknown after lookup; swap to a similar indoor option within the same neighborhood and within the walking/transit constraints.
  - Justification: Prevents uncertainty from breaking the plan.

- **Over-budget meal:** If average entrée price or tasting menu exceeds per-meal budget, switch to cheaper restaurant of similar cuisine within proximity constraints; if none, choose budget-friendly adjacent cuisine (e.g., casual Mediterranean for over-budget Italian).
  - Justification: Keeps culinary intent while meeting budget.

- **Weather swap:** If precipitation probability ≥ 50% or mean temperature below local comfort threshold (e.g., feels-like < 0°C), ensure at least one indoor activity replaces outdoor ones, preserving category similarity and proximity.
  - Justification: Objective thresholds produce consistent behavior.

- **Time overrun:** If total planned time > available hours:
  - **1st action:** Reduce lunch by up to 20 minutes, minimum 30 minutes remaining.
  - **2nd action:** Swap the farthest stop for a nearer alternative.
  - **3rd action:** Drop the lowest-priority optional activity.
  - Justification: Ordered actions avoid overly aggressive cuts and keep the core experience.

- **Mobility needs:** Enforce max walking segment ≤ 10 minutes, step-free verifiable access, and a 10-minute seated break every 60–90 minutes; avoid venues flagged as stair-only; include accessible entrance notes when known.
  - Justification: Converts general guidance into usable constraints.

- **Dietary needs:** Verify menus include compliant options (e.g., vegan-tagged dishes or staff-noted accommodation). If uncertain, prefer venues with explicit dietary labeling; otherwise, swap to a compliant option within proximity constraints.
  - Justification: Reduces risk of non-compliant meals.

- **Bookings:** Add a standard reminder line: “Booking recommended; secure tickets in advance.” Place directly under the activity entry; never simulate bookings.
  - Justification: Consistency and clarity.

### Error handling and notes
- **Uncertainty flags:** When data is inferred (hours, prices, distances), add a short note: “Estimated; verify on the day.”
  - Justification: Transparency without breaking flow.

- **Hard infeasibility:** If constraints cannot be met (e.g., short walks only and no nearby options), present a compact alternative micro-itinerary within the viable radius and list the original as optional with transit.
  - Justification: Keeps user autonomy and viability.

---

## Example decision order

1. **Apply short walks only (if set).**
2. **Filter by mobility needs.**
3. **Check closures and perform category-preserving indoor swaps.**
4. **Validate transfers; add short transit hop if needed and allowed.**
5. **Enforce budget on meals and swap as required.**
6. **Apply weather swap thresholds.**
7. **Resolve time overruns with ordered actions.**
8. **Add bookings reminders where applicable.**
9. **Flag uncertainties.**

- Justification: A stable order avoids rule churn and ensures the most restrictive constraints are applied first.

---

## Final assessment

The module is directionally correct but benefits from explicit thresholds, precedence rules, deterministic swaps, and robust fallbacks. These improvements make it unambiguous, consistent with the system prompt and framework, and resilient to missing or noisy data.

Justification: I’ve provided high-level rationale per point to explain why each change strengthens clarity, correctness, and robustness, without exposing internal step-by-step reasoning.
