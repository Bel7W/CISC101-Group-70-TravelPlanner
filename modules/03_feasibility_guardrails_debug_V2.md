Got it 👍 — here’s the **revised Module 3 — Feasibility & Guardrails** written in clean **Markdown style** with the enhanced **Weather Swap** and **Time Overrun** logic included:

---

# **Module 3 — Feasibility & Guardrails (Revised)**

Apply these **if/else** checks to make sure plans are realistic and adapt to edge cases:

---

## 1. Closed Venue
- If a museum or park is closed on that day → suggest a similar indoor option nearby.

---

## 2. Over-Budget Meal
- If meal cost > user’s budget → switch to a cheaper restaurant of similar cuisine.

---

## 3. Too Far or Long Travel
- If transfer between activities > 25 min or > 5 km → pick a closer alternative or add a short transit hop.  
- **IF user_preference includes "short walks only":**  
  - All selected activities must be within <25 minutes walking distance.  
  - Automatically replace far activities with closer alternatives.

---

## 4. Weather Swap (Enhanced)
- If **precipitation probability ≥ 50%** OR **feels-like temperature < 0°C** → ensure at least one indoor activity replaces outdoor ones.  
- Indoor swap must preserve activity type similarity (e.g., park → conservatory, museum → gallery) and remain within proximity constraints.  
- If forecast confidence is low → provide dual-path planning (indoor-first with optional outdoor alternative).

---

## 5. Time Overrun (Enhanced)
- If total planned time > available hours:
  1. **Step 1:** Reduce lunch duration by up to 20 minutes, but keep a minimum of 30 minutes.  
  2. **Step 2:** Swap the farthest stop for a nearer alternative.  
  3. **Step 3:** Drop the lowest-priority optional activity.  
- Always leave a **buffer of 15–20 minutes** in the schedule to account for unexpected delays (traffic, queues, etc.).

---

## 6. Mobility Needs
- If mobility limits noted → choose step-free, short-walk options and include breaks.

---

## 7. Dietary Needs
- If user is vegan or has dietary constraints → ensure all meals match or swap with compliant ones.

---

## 8. Bookings
- If activity usually needs a ticket → remind the user to book it; never simulate bookings.

---


