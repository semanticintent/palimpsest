# Verify

Stage 3 listed the checks (2026-11-08); stage 4 fills in the results in a fresh session.

44 checks to run. Rules still CONFLICT or UNKNOWN (their questions were skipped in run 1) are listed under **Blocked** and are not checked. Retired rules are checked only for absence.

## Checks

| Rule | Check | Result | Evidence |
| --- | --- | --- | --- |
| R-01 | Given users A and B each with properties, when A opens any property, expense, market-value or overview screen, then none of B's data appears and none of A's actions write under B. | | |
| R-02 | Given properties created on day 1 and day 2, when the list opens, then the day-2 property is first. | | |
| R-03 | Given a property at "12 Elm St, Ottawa, ON, K1A 0A1" with no postal code, when it is listed, then its title reads "12 Elm St, Ottawa, ON, <postal>". | | |
| R-05 | Given the add form with city empty, when SAVE AND NEXT is tapped, then nothing is written and city shows a required-field error. | | |
| R-06 | Given the add form is opened, when it renders, then Country shows "Canada". | | |
| R-07 | Given a valid new property, when it is saved, then the stored document's `id` equals its document key and it has a creation timestamp. | | |
| R-08 | Given a valid new property, when SAVE AND NEXT is tapped, then the details screen for the new property opens. | | |
| R-10 | Given a property with market value 400,000 and taxes 3,000/yr, when only its city is changed and saved, then market value and taxes are unchanged. | | |
| R-11 | Given an edited property, when "SAVE AND GO BACK TO LIST" is chosen, then the change is saved and the property list opens. | | |
| R-14 | Given description "corner lot", when an address is picked from lookup, then the address fields change, the description is still "corner lot", and nothing is stored until SAVE. | | |
| R-16 | Given a property, when Delete is tapped and the dialog is cancelled, then the property and its data remain. | | |
| R-17 | Given a property with 2 additional expenses and 1 borrowing, when it is deleted, then the property, both expenses and the borrowing are all gone from storage. | | |
| R-19 | Given a property, when its details open, then the four sections Main, Credits, Rent, Expenses are available. | | |
| R-20 | Given purchase date exactly 730 days ago, when Main opens, then age reads "2.0 years". | | |
| R-21 | Given start 2020-01-15 and term 12, when Rent opens, then renewal reads "15-Jan-2021". | | |
| R-22 | Given a property, when Property type is edited, then only these three choices are offered and the chosen string is stored. | | |
| R-24 | Retired: not expected in the new code (Opening property details does not need to compute borrowing metrics) | | |
| R-25 | Given purchase price 300,000 and market value 400,000, when market value is changed to 450,000, then purchase price is still 300,000. | | |
| R-26 | Given the Rent tab, when Rent amount is saved, then it is stored under the property's renting data. | | |
| R-27 | Given a saved amount of 1,200 USD yearly, when read back, then it is amount 1200, currency "USD", factor 12. | | |
| R-29 | Given rent 2,000, when cash flow is computed, then 2,000 counts as one month's rent. | | |
| R-30 | Given Rent increase limit set to 2.5, when Rent reloads, then it shows 2.5, stored under `rentIncreaseLimitPercent`; and a value stored earlier as `rentIncreaseLimit` is shown too. | | |
| R-31 | Given an amount of "abc", when SAVE is tapped, then nothing is saved and a numeric error shows. | | |
| R-33 | Given Property taxes with no frequency chosen, when SAVE is tapped, then nothing is saved and frequency shows a required-field error. | | |
| R-34 | Given taxes of 3,600 annually, when the monthly equivalent is computed, then it is 300. | | |
| R-35 | Given insurance 1,200 CAD annually, the Annually period and CAD, when Expenses opens, then the row shows 1,200.00; with Monthly it shows 100.00. | | |
| R-36 | Given rate 1.30 and a 100 USD expense, when totals are shown in CAD, then it counts as 130; in USD, a 130 CAD item shows as 100. | | |
| R-38 | Given taxes 3,600/yr, insurance 100/mo, one additional expense 50/mo and credit payments 1,000/mo, when Monthly is selected, then the total is 1,450.00; Annually shows 17,400.00. | | |
| R-41 | Given two borrowings paying 1,000/mo and 6,000/yr, when Monthly is selected, then Credit payments shows 1,500.00. | | |
| R-42 | Retired: not expected in the new code (The unshown "expenses grand total" need not be computed) | | |
| R-43 | Given property 1's details are open, when "Other expenses" is tapped, then only property 1's additional expenses are listed. | | |
| R-44 | Given the add dialog with no name, when SAVE is tapped, then nothing is saved and the name shows a required error. | | |
| R-45 | Given an expense "Plumber 400", when its amount is edited to 450, then the list still has one "Plumber" entry, now 450. | | |
| R-46 | Given expense A added before B, when the list opens, then B is first. | | |
| R-49 | Given market value 500,000 CAD and borrowings of 300,000 CAD and 50,000 USD at rate 1.30, when Market Value opens in CAD, then LTV is (300,000 + 65,000) / 500,000 × 100 = 73.00. | | |
| R-50 | Given a property with no market value, when Market Value opens, then its LTV shows "—". | | |
| R-51 | Given a new borrowing on property P, when it is stored, then it carries the user's id and P's id. | | |
| R-53 | Given the last details tab opened was Rent, when a market value is changed to 450,000 here and saved, then the property's stored market value is 450,000. | | |
| R-54 | Given LTV 60.00 at market value 500,000 with 300,000 borrowed, when market value is saved as 600,000, then LTV shows 50.00. | | |
| R-55 | Given 2 properties worth 400,000 and 600,000 CAD, when Home opens in CAD, then Properties shows 2 and Value shows 1,000,000. | | |
| R-56 | Given total value 1,000,000 and borrowings 650,000, when Home opens, then Equity shows 350,000. | | |
| R-57 | Given rent 3,000/mo, taxes 3,600/yr, credit payments 1,500/mo, when Home opens, then Cash Flow shows 1,200. | | |
| R-58 | Given a property's market value was just changed, when Home is opened, then Value reflects the change. | | |
| R-59 | Retired: not expected in the new code (No behaviour depends on the unreachable property code) | | |

## Blocked (question skipped in stage 3)

- R-04 The property list shows each property's market value — CONFLICT (C-15), question skipped
- R-09 The user is told "Success processing request." whenever a valid property form is submitted — CONFLICT (C-01), question skipped
- R-12 After SAVE (stay), the edit form shows the property — CONFLICT (C-02), question skipped
- R-13 CANCEL on the property form puts the fields back to their initial values and stays on the form — CONFLICT (C-03), question skipped
- R-15 Address lookup only suggests Canadian addresses — CONFLICT (C-04), question skipped
- R-18 A property delete happens as one operation: related data first, then the property — CONFLICT (C-07), question skipped
- R-23 The details screen reopens on whichever tab was last chosen, for any property — UNKNOWN (C-09), question skipped
- R-28 A money field without a frequency (market value, purchase price, sales cost, rent) is stored with factor 0 — CONFLICT (C-10), question skipped
- R-32 When the user saves a field, they are told it has been updated — CONFLICT (C-01), question skipped
- R-37 When the user has not set an exchange rate, 1.42 is used — UNKNOWN (C-12), question skipped
- R-39 Additional expenses in another currency are converted before being added to a property's totals — CONFLICT (C-13), question skipped
- R-40 The period (monthly/annually) and currency (CAD/USD) chosen for a property's totals are applied to every figure on the Expenses tab — CONFLICT (C-14), question skipped
- R-47 After adding an expense, the user is told it has been saved, by name — CONFLICT (C-01), question skipped
- R-48 An additional expense is deleted immediately, without confirmation — UNKNOWN (C-08), question skipped
- R-52 The market value screen shows values in the currency the user chose — CONFLICT (C-14), question skipped

## Not testable from code

- Behaviour that depends on Firestore itself (cascading deletes of subcollections, security rules, indexes) or on live data written by the original app.
- External calls: Google Places / Geocoder responses.
- Timing: writes that are not awaited, the delay before the overview refreshes.
- See `README.md` › Not traced for the full list.

## Run

- Date:
- Run by: <model>, in <tool>, fresh session: yes
- Totals: <n> pass · <n> fail · <n> not testable
