# Finding hidden decisions

A hidden decision is anything that changes the solution's behavior but that the requirement doesn't specify and you can't find in the code. Your job is to find these *before* building, because each one silently guessed is a potential mismatch.

## Categories to check

Go through each category and ask: "Is this specified? If not, would different answers produce a different solution?" Only ask about the ones where the answer matters.

| Category | What to look for |
|---|---|
| **Business rules** | Prices, fees, rates, surcharges, discounts, thresholds, rounding, limits |
| **Variation** | Does the rule change by region, time, user type, weather, day of week, quantity? |
| **Actors & permissions** | Who does what? Who can see / change / approve? |
| **Data** | What inputs exist, where do they come from, what format, what if missing? |
| **Edge cases** | Zero, negative, maximum, duplicate, concurrent, cancelled, partial |
| **Timing** | When is it calculated, paid, settled, expired? Real-time or batch? |
| **Integration** | External APIs, existing systems, payment providers, notifications |
| **Non-functional** | Scale, performance, security, audit/logging, history |
| **Scope** | What is explicitly out of scope? UI or backend only? MVP or production? |

## How to ask

- Most consequential first. A question whose answer changes the data model beats one about rounding.
- No recommendations. Give each option equal weight and state its consequence, so the user chooses by their own goal, not by your label. Always leave room for a free-form answer ("other / tell me your rule").
- Max ~4–5 per round. Ask the rest in the next round. Don't fill them in yourself.
- Explain *why* you're asking when it isn't obvious.

### Adapt the wording, not the content

The same decision, phrased for two audiences:

- **Developer:** "How should the distance surcharge scale? (a) Step function per km bracket: pay changes only at bracket edges, easy to show in a rate table. (b) Linear per-meter rate: pay grows smoothly, harder to explain as a fixed table. (c) Other rule."
- **Non-developer:** "When a delivery is farther, how should the extra pay grow? (a) A fixed amount for each distance range, like +500원 for 3–5 km. Riders can see the rates at a glance. (b) A little extra for every meter. Pay matches the distance more closely, but the amounts are less round. (c) Tell me your own rule."

Both lead to the same recorded decision, and neither pushes the user toward an answer.

## Worked example: "배달기사에게 주어지는 페이 기능을 구현하라"

Hidden decisions found:

1. **Base fee** — fixed for everyone, or different by region? If by region, what's the unit (city / district) and the amounts?
2. **Distance surcharge** — from what distance does it start, and how does it grow (step vs linear)? Distance measured how (straight line vs road route)?
3. **Weather surcharge** — which conditions (rain, snow, heat wave)? Fixed amount or percentage? Where does weather data come from — an external API or manual admin toggle?
4. **Other surcharges** — late night, peak hours, holidays?
5. **Platform commission** — is a fee deducted from the rider's pay? Percentage of what?
6. **Combination** — do surcharges stack? Is there a cap?
7. **Settlement** — paid per delivery, daily, weekly? Cancelled orders?
8. **Rounding** — to 1원, 10원, 100원?

Round 1 asks 1, 2, 3, 6 (they shape the calculation model). Round 2 covers the rest.
