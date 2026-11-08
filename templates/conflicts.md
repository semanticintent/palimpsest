# Conflicts and unknowns

Every place where the sources disagree, or where nothing says why. Stage 2 asks; stage 3 answers.

### C-01  <short title>

- **Rules:** R-07
- **What disagrees:** <the code does X; the screen / doc / evident purpose says Y>
- **Question:** <the one question a person must answer>
- **Decision:** <what holds from now on> — <who>, <date> — <one-line reason>
  *or* **Owner:** <who will decide>, by <when>; R-07 blocked until then
  *or* **Skipped:** <who>, <date> — <why>

---

## Example (from case PX-001)

### C-03  Exchange-rate direction

- **Rules:** R-12
- **What disagrees:** the spec's default is 1.42 labelled "USD/CAD"; the build's default is 0.74 labelled "1 CAD = X USD"
- **Question:** which direction and value is intended?
- **Decision:** 0.72, read as 1 CAD = 0.72 USD — Michael, 2026-04-03 — current rate
