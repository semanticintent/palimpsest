# Rules

One entry per rule. IDs never change once given; a retired rule keeps its ID with `Disposition: retire`.

Write rules as what must hold for the user or the data, not as how the old screens did it. Old field names and labels go in the check as a hint.

### R-01  <one sentence that must hold>

- **Workflows:** W-01 (or several, for a rule that spans them)
- **Evidence:** `file:line`, <screen text>, <comment>, <doc>, <person>
- **Status:** RESOLVED | ASSUMED | UNKNOWN (C-01) | CONFLICT (C-01)
- **Edge cases:** <limits, and what the old system did wrong there>
- **Check:** Given <state>, when <action>, then <result> *(old: <field or label, as a hint>)*. For UNKNOWN or CONFLICT: `after C-01`.
- **Disposition:** transform | preserve | retire

---

## Example (invented: an invoicing module)

### R-07  An invoice's due date is its issue date plus the customer's payment terms

- **Workflows:** W-02 Create invoice, W-05 Edit customer terms
- **Evidence:** `invoice_service.py:88` adds `terms_days`; the invoice screen shows "Due in {terms} days"
- **Status:** RESOLVED
- **Edge cases:** terms of 0 days: due on the issue date
- **Check:** Given a customer on 30-day terms, when an invoice is issued on 3 March, then it is due on 2 April *(old: field `due_dt`)*
- **Disposition:** transform
