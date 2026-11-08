# Conflicts and unknowns

Every place where the sources disagree, or where nothing shows why and it matters. Stage 2 asks, highest harm first; stage 3 answers.

Harm: **high** = data loss or wrong money · **medium** = wrong display or a confusing flow · **low** = cosmetic.

### C-01  <short title>

- **Harm:** high | medium | low
- **Rules:** R-07
- **What disagrees:** <the code does X; the screen / comment / evident purpose says Y>
- **Existing data:** <affected or not; what would need migrating>
- **Question:** <one question; a follow-up only if the first answer needs it>
- **Open** (stage 3), then one of:
  - **Decision:** <what holds from now on> — <who>, <date> — <one-line reason>
  - **Owner:** <who will decide>, by <when>; R-07 blocked until then
  - **Skipped:** <who>, <date> — <why>; R-07 blocked

---

## Example (invented: an invoicing module)

### C-02  Credit notes reduce the total but not the tax

- **Harm:** high
- **Rules:** R-11
- **What disagrees:** the credit-note screen says "Reverses the invoice", but `credit.py:40` subtracts the net amount only and leaves the tax line unchanged
- **Existing data:** 212 credit notes have been issued this way
- **Question:** should a credit note reverse the tax as well as the net amount? If yes, should past credit notes be corrected?
- **Decision:** yes, reverse the tax; past notes are corrected by a one-off adjustment — Finance lead, 2026-03-02 — tax was being over-reported
