# Rules

One entry per rule. IDs never change once given; a retired rule keeps its ID with `Disposition: retire`.

### R-01  <one sentence that must hold>

- **Workflow:** W-01
- **Evidence:** `file:line`, <screen>, <doc>, <person>
- **Status:** RESOLVED | ASSUMED | UNKNOWN | CONFLICT (C-01)
- **Confidence:** high | medium | low — <why, when not high>
- **Edge cases:** <limits, and what the old system did wrong there>
- **Check:** Given <state>, when <action>, then <result>
- **Disposition:** transform | preserve | retire

---

## Example (from case PX-001)

### R-04  Loan-to-value is total borrowings divided by market value, per property

- **Workflow:** W-06 View market values
- **Evidence:** original market-value model; "Market values" screen
- **Status:** RESOLVED
- **Confidence:** high
- **Edge cases:** market value 0: the original showed NaN or Infinity (see C-02, decided)
- **Check:** Given a property with market value 0, when its LTV is shown, then it reads "—"
- **Disposition:** transform
