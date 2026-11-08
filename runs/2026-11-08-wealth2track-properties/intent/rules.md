# Rules

One entry per rule. IDs never change once given; a retired rule keeps its ID with `Disposition: retire`.

All paths are relative to `wealth2track/lib/`. The only sources are the code, the text the code shows on screen (labels, help text, messages) and code comments. No running app, docs or people were consulted. **RESOLVED** here means the code, its on-screen text and/or comments agree and the purpose is clear. **ASSUMED** means one source only (usually code alone), with a plausible purpose.

---

## W-01 List properties

### R-01  A user only ever sees and changes their own properties and the data under them

- **Workflow:** all (W-01 … W-16)
- **Evidence:** every query is rooted at `users/{uid}`: `firestore/property_service.dart:16-21`, `data_delete_service.dart:9-14`, `expense_service.dart:28`, `metric_service.dart:22-27`, `overview_service.dart:53`, `borrowing_service.dart:146-150`. The collection-group borrowings query filters on `userId == uid` (`borrowing_service.dart:137-140`).
- **Status:** RESOLVED
- **Confidence:** medium. Server-side enforcement (Firestore security rules) is not in `lib/` and was not seen.
- **Edge cases:** the collection-group query relies on every borrowing carrying a correct `userId` (see R-51).
- **Check:** Given users A and B each with properties, when A opens any property, expense, market-value or overview screen, then none of B's data appears and none of A's actions write under B.
- **Disposition:** preserve

### R-02  The property list shows the most recently created property first

- **Workflow:** W-01, also W-14
- **Evidence:** `property_service.dart:66-69` (`orderBy('timestamp', descending: true)`), `borrowing_service.dart:151`
- **Status:** ASSUMED
- **Confidence:** medium. Code only; the order is not stated anywhere on screen.
- **Edge cases:** a property document without `timestamp` is left out by Firestore `orderBy` (platform behaviour, not seen in `lib/`). The overview query has no `orderBy` (`overview_service.dart:132`), so its count can include such documents.
- **Check:** Given properties created on day 1 and day 2, when the list opens, then the day-2 property is first.
- **Disposition:** transform

### R-03  A property's display name is its address ("line 1, city, province, postal"), not a separately entered name

- **Workflow:** W-01, W-06, W-14
- **Evidence:** `models/property/property_header.dart:12` (stored `propertyName` field commented out), `:20-28` (getter, `@JsonKey(ignore: true)`); the edit form has no name field (`screens/properties/property_edit.dart:83-167`)
- **Status:** ASSUMED
- **Confidence:** medium. The commented-out field suggests a deliberate change, but nothing says why.
- **Edge cases:** missing parts show as `<address line1>`, `<city>`, `<province>`, `<postal>` (`property_header.dart:23-26`). `property_edit.dart:71` still seeds a `property_name` form key that has no field.
- **Check:** Given a property at "12 Elm St, Ottawa, ON, K1A 0A1" with no postal code, when it is listed, then its title reads "12 Elm St, Ottawa, ON, <postal>".
- **Disposition:** transform

### R-04  The property list shows each property's market value

- **Workflow:** W-01
- **Evidence:** `properties_list.dart:130-133` (`formatCurrency(prop.propertyDetails.marketValue?.amount)`, or `'N/A'` when `propertyDetails` is null); `property_tile.dart:84` (placeholder `<Market value>` for `'N/A'`)
- **Status:** CONFLICT (C-15)
- **Confidence:** high on behaviour
- **Edge cases:** the raw stored amount is shown with no currency and no conversion, whereas W-14/W-16 convert to CAD/USD. If `propertyDetails` exists but `marketValue` is null, `formatCurrency(null)` shows `0.00`, not the placeholder (`config/utils.dart:51`).
- **Check:** stage 3. Option A: Given a property with market value 500,000 USD and a CAD view, when listed, then it shows the CAD-converted value with a currency label. Option B: it shows "500,000.00" exactly as entered.
- **Disposition:** transform

---

## W-02 Create a property

### R-05  A new property needs a building type, address line 1, city, province and country; description, line 2 and postal code are optional

- **Workflow:** W-02, W-03
- **Evidence:** `property_edit.dart:83-95` (building type required; options Single-Family Home, Duplex, Apartment Building, Condominium, Townhouse), `:106-121` (line 1 required), `:130-137` (city required), `:145-152` (province required), `:154-167` (country required; Canada, USA, Other); validation runs on save (`buttons/property_save_button.dart:49`)
- **Status:** RESOLVED
- **Confidence:** high
- **Edge cases:** postal code is optional, but it is part of the display name (R-03).
- **Check:** Given the add form with city empty, when SAVE AND NEXT is tapped, then nothing is written and city shows a required-field error.
- **Disposition:** transform

### R-06  A new property's country starts as Canada

- **Workflow:** W-02
- **Evidence:** `property_service.dart:247` (`new Address(country: 'Canada')`); consistent with Canada-only lookup (`location_lookup_button.dart:32`) and CAD defaults (`form_currency_field.dart:58`, `expense_service.dart:14`)
- **Status:** ASSUMED
- **Confidence:** medium. Nothing states the user base, but every default points to Canada.
- **Edge cases:** —
- **Check:** Given the add form is opened, when it renders, then Country shows "Canada".
- **Disposition:** transform

### R-07  The store assigns each new property's id; the property records its own id and when it was created

- **Workflow:** W-02
- **Evidence:** `property_service.dart:23-28` (`add({... 'timestamp': now})`, then `updateData({'id': documentID})`)
- **Status:** ASSUMED
- **Confidence:** medium. `id` is used for routing (`property_save_button.dart:55-58`) and `timestamp` for ordering (R-02); the reason for storing the id inside the document is not stated.
- **Edge cases:** if `add` fails, the error is only printed (`:24`) and the follow-up `doc.updateData` would run on null. See R-09.
- **Check:** Given a valid new property, when it is saved, then the stored document's `id` equals its document key and it has a creation timestamp.
- **Disposition:** transform

### R-08  After a new property is saved, the user lands on that property's details, with it selected

- **Workflow:** W-02
- **Evidence:** button label "SAVE AND NEXT" (`property_save_button.dart:26`); `:36-37`, `:53-58` (`setSelectedProperty(newId)`, go to `/property_details`)
- **Status:** RESOLVED
- **Confidence:** high
- **Edge cases:** the selected details tab is whatever was last chosen (R-23).
- **Check:** Given a valid new property, when SAVE AND NEXT is tapped, then the details screen for the new property opens.
- **Disposition:** transform

### R-09  The user is told "Success processing request." whenever a valid property form is submitted

- **Workflow:** W-02, W-03
- **Evidence:** `property_save_button.dart:69-73` (snackbar shown right after starting the save, outside the `.then`); `property_service.dart:24` (add error printed); `:61-63` (update error not handled)
- **Status:** CONFLICT (C-01)
- **Confidence:** high on behaviour
- **Edge cases:** if the save fails, the message still says success.
- **Check:** stage 3. Option A: Given the store rejects the write, when SAVE is tapped, then the user sees a failure message and stays on the form. Option B: as now.
- **Disposition:** transform

---

## W-03 Edit a property's header

### R-10  Editing a property's header never changes its money data (details, rent, expenses)

- **Workflow:** W-03
- **Evidence:** `property_service.dart:55-64` explicitly removes `propertyDetails`, `propertyRenting`, `propertyExpenses` before `updateData`. The edit form has no money fields (`property_edit.dart:83-167`); those are edited per field on the details tabs (W-07, W-08).
- **Status:** RESOLVED
- **Confidence:** high
- **Edge cases:** the header model is loaded once when the edit screen opens (`changeNotifiers/property_tab.dart:23-25`). Writing the full nodes would overwrite money edits made since, which the removal avoids.
- **Check:** Given a property with market value 400,000 and taxes 3,000/yr, when only its city is changed and saved, then market value and taxes are unchanged.
- **Disposition:** preserve

### R-11  After editing, the user can stay on the form, go to the property's details, or go back to the list

- **Workflow:** W-03
- **Evidence:** labels "SAVE", "SAVE AND GO TO DETAILS", "SAVE AND GO BACK TO LIST" (`property_save_button.dart:26, 87, 93`); routes `:38-39`, `:95-100`
- **Status:** RESOLVED
- **Confidence:** high
- **Edge cases:** see R-12 for what "stay" shows.
- **Check:** Given an edited property, when "SAVE AND GO BACK TO LIST" is chosen, then the change is saved and the property list opens.
- **Disposition:** transform

### R-12  After SAVE (stay), the edit form shows the property

- **Workflow:** W-03
- **Evidence:** after save, the edit route is reopened (`property_save_button.dart:38-39, 64-66`); the form is built from `notifier.propertyHeader` (`property_edit.dart:49-53`), which still holds the value loaded before the save; nothing re-loads it (`property_save_button.dart:45-75`)
- **Status:** CONFLICT (C-02)
- **Confidence:** medium. Traced from code, not observed running.
- **Edge cases:** if the address lookup was used, the form shows the looked-up model.
- **Check:** stage 3. Option A: Given city changed from Ottawa to Toronto, when SAVE is tapped, then the form shows Toronto. Option B: as now, it shows Ottawa until re-entered.
- **Disposition:** transform

### R-13  CANCEL on the property form puts the fields back to their initial values and stays on the form

- **Workflow:** W-02, W-03
- **Evidence:** `property_edit.dart:177-186` (`_fbKey.currentState.reset()`, no navigation); leaving is via the back arrow (`:51`)
- **Status:** CONFLICT (C-03)
- **Confidence:** high on behaviour
- **Edge cases:** —
- **Check:** stage 3. Option A (leave): Given edited fields, when CANCEL is tapped, then nothing is saved and the list opens. Option B (as now): the fields revert and the form stays.
- **Disposition:** transform

---

## W-04 Look up an address

### R-14  Choosing a looked-up address fills the address fields, keeps the description and building type, and saves nothing until the form is saved

- **Workflow:** W-04
- **Evidence:** `location_lookup_button.dart:42-73` (geocode, build model keeping `property_desc` and `building_type`); `property_tab.dart:40-47` (in-memory only); the write happens only in W-02/W-03
- **Status:** ASSUMED
- **Confidence:** medium. Whether the form actually re-applies the new values was not traced (README › Not traced, framework).
- **Edge cases:** only the first geocoder result is used (`:51`). Line 1 is `"subThoroughfare thoroughfare"` and can read "null Main St" when there is no street number (`:58`). Line 2 is set to `subAdminArea` (a region, not a unit) (`:59`).
- **Check:** Given description "corner lot", when an address is picked from lookup, then the address fields change, the description is still "corner lot", and nothing is stored until SAVE.
- **Disposition:** transform

### R-15  Address lookup only suggests Canadian addresses

- **Workflow:** W-04
- **Evidence:** `location_lookup_button.dart:31-34` (`Component(Component.country, "ca")`, US line commented out); the form offers Canada, USA, Other (`property_edit.dart:163`); the app supports USD money (`form_currency_field.dart:59`)
- **Status:** CONFLICT (C-04)
- **Confidence:** high on behaviour
- **Edge cases:** —
- **Check:** stage 3. Option A: Given a US address typed into lookup, when suggestions show, then US addresses appear. Option B: only Canadian ones appear.
- **Disposition:** transform

---

## W-05 Delete a property

### R-16  Deleting a property needs an explicit confirmation that warns it cannot be undone

- **Workflow:** W-05
- **Evidence:** `buttons/property_delete_button.dart:52-58` ("Deleting Property?", "This operation can't be undone…"); CANCEL `:32-38`, CONTINUE `:40-48`
- **Status:** RESOLVED
- **Confidence:** high
- **Edge cases:** —
- **Check:** Given a property, when Delete is tapped and the dialog is cancelled, then the property and its data remain.
- **Disposition:** preserve

### R-17  Deleting a property also deletes its additional expenses and its borrowings

- **Workflow:** W-05
- **Evidence:** intent comment `data_delete_service.dart:16-19` ("delete properties / delete additionalExpense / delte borrowins"); `:21-25`. But expenses are deleted from subcollection `additionalProperties` (`:28`), while they are stored in `additionalExpenses` (`property_service.dart:116, 235`).
- **Status:** CONFLICT (C-05, C-06)
- **Confidence:** high
- **Edge cases:** as written, a deleted property's additional expenses remain in storage. Nothing reads them afterwards, because every reader starts from existing properties (`overview_service.dart:178-187`).
- **Check:** Given a property with 2 additional expenses and 1 borrowing, when it is deleted, then the property, both expenses and the borrowing are all gone from storage.
- **Disposition:** transform

### R-18  A property delete happens as one operation: related data first, then the property

- **Workflow:** W-05
- **Evidence:** the author awaits the steps in order (`data_delete_service.dart:22-24`), but the inner `get().then(...)` is neither returned nor awaited (`:28, :36`), so the property can be deleted before or without its children
- **Status:** CONFLICT (C-07)
- **Confidence:** medium. Exact ordering at runtime was not traced.
- **Edge cases:** an interrupted delete can leave borrowings whose property is gone. Those still match the collection-group query (`borrowing_service.dart:136-144`), and `getBorrowingsAll` would hit a null property (`:128-130`, out of scope).
- **Check:** stage 3. Option A (all-or-nothing): Given a delete interrupted after the first child delete, when the app is reopened, then either everything still exists or nothing does. Option B: best effort, as now.
- **Disposition:** transform

---

## W-06 View a property's details

### R-19  A property's details are grouped as Main, Credits, Rent and Expenses

- **Workflow:** W-06
- **Evidence:** `widgets/property_tabs.dart:13`; `property_details.dart:77-88`
- **Status:** RESOLVED
- **Confidence:** high
- **Edge cases:** Main is the default when no other tab matches (`property_details.dart:86-87`).
- **Check:** Given a property, when its details open, then the four sections Main, Credits, Rent, Expenses are available.
- **Disposition:** transform

### R-20  Property age is the time since the purchase date, in years to one decimal; "N/A" without a purchase date

- **Workflow:** W-06
- **Evidence:** `config/utils.dart:80-91` (days / 365, `toStringAsFixed(1)` + " years"); label "Property age", help "Since purchase date" (`tabs/main_tab.dart:80-88`)
- **Status:** RESOLVED
- **Confidence:** high
- **Edge cases:** uses 365-day years (leap days ignored); a future purchase date gives a negative age.
- **Check:** Given purchase date exactly 730 days ago, when Main opens, then age reads "2.0 years".
- **Disposition:** transform

### R-21  The rent renewal date is the start date plus the term in months; "N/A" if either is missing

- **Workflow:** W-06
- **Evidence:** `config/utils.dart:60-67`; `tabs/rent_tab.dart:48-65` ("Term in month", "Renewal date", read-only)
- **Status:** RESOLVED
- **Confidence:** high
- **Edge cases:** month overflow follows Dart `DateTime`: a 31 January start + 1 month gives 3 March (2 March in a leap year). Fractional terms are truncated (`term?.toInt()`).
- **Check:** Given start 2020-01-15 and term 12, when Rent opens, then renewal reads "15-Jan-2021".
- **Disposition:** transform

### R-22  Property type is one of "Primary Residence", "Secondary Residence", "Investment"

- **Workflow:** W-06, W-07
- **Evidence:** `models/property/property_type.dart:3-22`; radio field `main_tab.dart:93-100` ("Investment or residence"); stored as the display string (`property_service.dart:163-170`)
- **Status:** RESOLVED
- **Confidence:** high
- **Edge cases:** stored data holds these exact strings.
- **Check:** Given a property, when Property type is edited, then only these three choices are offered and the chosen string is stored.
- **Disposition:** preserve

### R-23  The details screen reopens on whichever tab was last chosen, for any property

- **Workflow:** W-06 (affects W-07, W-15)
- **Evidence:** tab and index live on the app-wide `PropertyTabModel` (`changeNotifiers/property_tab.dart:10-11, 29-33`, provided app-wide at `main.dart:50-52`); the only setter call is the tab tap (`property_tabs.dart:30`)
- **Status:** UNKNOWN (C-09)
- **Confidence:** high on behaviour; nothing says why
- **Edge cases:** the same index decides where field edits are written (R-26, R-52).
- **Check:** stage 3. Option A: Given Expenses was last open on property 1, when property 2's details open, then Main is shown. Option B: Expenses is shown.
- **Disposition:** transform

### R-24  Opening property details does not need to compute borrowing metrics

- **Workflow:** W-06
- **Evidence:** `property_details.dart:40-41` calls `MetricService.calculate` on every build; no screen reads `MetricService.metrics` (grep of `screens/`: only `property_details.dart:40`). `ExpenseService` computes its own instance (`expense_service.dart:117-119`).
- **Status:** RESOLVED
- **Confidence:** high
- **Edge cases:** —
- **Check:** Given details open, when the screen renders, then no displayed value depends on the provider-level `MetricService`.
- **Disposition:** retire

---

## W-07 Edit a property detail or rent field

### R-25  Editing one field saves only that field; other stored fields of the property are untouched

- **Workflow:** W-07, W-08, W-15
- **Evidence:** `property_service.dart:123-201` (`setData({node: {field: value}}, SetOptions(merge: true))`)
- **Status:** RESOLVED
- **Confidence:** high
- **Edge cases:** a Money field is written whole (`amount`, `currency`, `factor`) (`:127`).
- **Check:** Given purchase price 300,000 and market value 400,000, when market value is changed to 450,000, then purchase price is still 300,000.
- **Disposition:** preserve

### R-26  A field edited on Main is stored in the property's details, on Rent in its renting, on Expenses in its expenses

- **Workflow:** W-07, W-08
- **Evidence:** `property_service.dart:86-110` (route + tab index → node); `widgets/edit_fieldvalue_dialog.dart:158`; tab order `property_tabs.dart:13`
- **Status:** RESOLVED
- **Confidence:** high for the mapping. How it is decided (screen route + app-wide tab index) is the old mechanism and is free to change.
- **Edge cases:** an unresolved route writes nothing (R-32, R-52).
- **Check:** Given the Rent tab, when Rent amount is saved, then it is stored under the property's renting data.
- **Disposition:** transform

### R-27  Money is stored as amount + currency (USD or CAD) + frequency factor

- **Workflow:** W-07, W-08, W-11, W-15
- **Evidence:** `models/property/money.dart:5-40`, `money.g.dart:16-20`; currency choices `fields/form_currency_field.dart:57-65`; `expense.g.dart:20-27` (additional expenses use the same three plus name, date, id)
- **Status:** RESOLVED
- **Confidence:** high
- **Edge cases:** any currency other than `"USD"` is treated as CAD in conversion (`money.dart:22-27`).
- **Check:** Given a saved amount of 1,200 USD yearly, when read back, then it is amount 1200, currency "USD", factor 12.
- **Disposition:** preserve

### R-28  A money field without a frequency (market value, purchase price, sales cost, rent) is stored with factor 0

- **Workflow:** W-07, W-15
- **Evidence:** `edit_fieldvalue_dialog.dart:164` (`int.parse(value['factor'] ?? '0')`); the frequency radio is only shown when `showCycle` (`form_currency_field.dart:69`), which is false for these fields (`property_record.dart:26`, `market_value_tile.dart:54`). The model treats a missing factor as 1 (`money.dart:32`).
- **Status:** CONFLICT (C-10)
- **Confidence:** high
- **Edge cases:** in the traced paths these fields go through `calculateCAD` (no division), so no divide-by-zero happens today. Any future use of `calculateCADFactored` on them would divide by 0.
- **Check:** stage 3. Option A: Given market value 400,000 saved, when read back, then factor is absent or 1. Option B: factor is 0, as now.
- **Disposition:** transform

### R-29  Rent amount is a monthly figure

- **Workflow:** W-07, W-16
- **Evidence:** comment `overview_service.dart:121-122` ("rentTotal is based on Renting Amount which is monthly"); rent has no frequency control (`rent_tab.dart:26-32`)
- **Status:** ASSUMED
- **Confidence:** medium. A comment only; the screen label is just "Rent amount".
- **Edge cases:** —
- **Check:** Given rent 2,000, when cash flow is computed, then 2,000 counts as one month's rent.
- **Disposition:** preserve

### R-30  The rent increase limit the user enters is the one shown back

- **Workflow:** W-07
- **Evidence:** saved under field `rentIncreaseLimit` (`rent_tab.dart:83`); displayed from `rentIncreaseLimitPercent` (`rent_tab.dart:85`, `property_renting.dart:14`, `property_renting.g.dart:24-25`)
- **Status:** CONFLICT (C-11)
- **Confidence:** high
- **Edge cases:** as written, a saved limit never appears on screen; stored data may hold either name.
- **Check:** Given Rent increase limit set to 2.5, when Rent reloads, then it shows 2.5.
- **Disposition:** transform

### R-31  Field inputs are validated: amounts and numbers required and numeric; an empty date starts at today

- **Workflow:** W-07, W-08, W-11, W-12, W-15
- **Evidence:** `form_currency_field.dart:42-45`, `form_number_field.dart:25-28`, `form_expense_field.dart:33-35, 48-51`, `form_date_field.dart:17`, `form_radio_field.dart:26-28`
- **Status:** RESOLVED
- **Confidence:** high
- **Edge cases:** negative numbers pass the numeric validator; dates have no validation (`validator: (val) => null`).
- **Check:** Given an amount of "abc", when SAVE is tapped, then nothing is saved and a numeric error shows.
- **Disposition:** transform

### R-32  When the user saves a field, they are told it has been updated

- **Workflow:** W-07, W-08, W-15
- **Evidence:** dialog pops with "`<label>` has been updated." right after calling the service, which is not awaited (`edit_fieldvalue_dialog.dart:162-198`); unresolved routes write nothing (`property_service.dart:136-140, 203-210`)
- **Status:** CONFLICT (C-01)
- **Confidence:** high
- **Edge cases:** the message shows even when nothing was written or the write fails.
- **Check:** stage 3. Option A: Given the write fails or cannot be placed, when SAVE is tapped, then the user sees a failure message. Option B: as now.
- **Disposition:** transform

---

## W-08 Edit a standard expense

### R-33  Each property has four standard recurring expenses, each with an amount, currency and a required frequency (monthly or annually)

- **Workflow:** W-08
- **Evidence:** `tabs/expenses_tab.dart:62-109` (Property taxes, Maintenance fee, Property management fee, Insurance; `showCycleCurrencyField: true`); `form_currency_field.dart:69-80` (Frequency radio, required); `models/property/property_expenses.dart:7-11`
- **Status:** RESOLVED
- **Confidence:** high
- **Edge cases:** —
- **Check:** Given Property taxes with no frequency chosen, when SAVE is tapped, then nothing is saved and frequency shows a required-field error.
- **Disposition:** transform

### R-34  Frequency is stored as factor 1 = monthly, 12 = annually; the monthly equivalent is amount ÷ factor

- **Workflow:** W-08, W-09, W-11, W-16
- **Evidence:** `form_currency_field.dart:77`, `form_expense_field.dart:96` (Annually→'12', Monthly→'1'); `money.dart:31-34`, `expense.dart:37-40` (missing factor → 1)
- **Status:** RESOLVED
- **Confidence:** high
- **Edge cases:** factor 0 (R-28) would divide by zero.
- **Check:** Given taxes of 3,600 annually, when the monthly equivalent is computed, then it is 300.
- **Disposition:** preserve

### R-35  Each standard expense row shows its value for the selected period and currency

- **Workflow:** W-08, W-09
- **Evidence:** `expense_service.dart:32-48` (monthly equivalent in CAD × period, × 1/rate for USD); `expenses_tab.dart:67, 80, 93, 106`; `property_record.dart:51`
- **Status:** RESOLVED
- **Confidence:** high
- **Edge cases:** the returned Money keeps the original currency label while holding the converted amount (`expense_service.dart:45`); only the amount is shown.
- **Check:** Given insurance 1,200 CAD annually, the Annually period and CAD, when Expenses opens, then the row shows 1,200.00; with Monthly it shows 100.00.
- **Disposition:** transform

---

## W-09 View a property's expense totals

### R-36  Amounts are combined in CAD: USD is multiplied by the user's exchange rate; USD views divide by it

- **Workflow:** W-08, W-09, W-14, W-16
- **Evidence:** `money.dart:19-29`, `expense.dart:25-35`; rate from `users/{uid}.settings.exchangeRate` (`models/user/settings.dart:7`) in `expense_service.dart:35, 68`, `metric_service.dart:33`, `overview_service.dart:56`, `borrowing_service.dart:54, 81`
- **Status:** RESOLVED
- **Confidence:** high. Consistent across five services.
- **Edge cases:** `BorrowingService` and `ExpenseService.getFieldCalculatedValue` use a rate cached at sign-in (`borrowing_service.dart:35`, `expense_service.dart:29`); the others re-read per call.
- **Check:** Given rate 1.30 and a 100 USD expense, when totals are shown in CAD, then it counts as 130; in USD, a 130 CAD item shows as 100.
- **Disposition:** preserve

### R-37  When the user has not set an exchange rate, 1.42 is used

- **Workflow:** W-08, W-09, W-14, W-16
- **Evidence:** `expense_service.dart:35, 68`, `metric_service.dart:33`, `overview_service.dart:56`, `borrowing_service.dart:54, 81` (`?? 1.42`)
- **Status:** UNKNOWN (C-12)
- **Confidence:** high on behaviour; nothing says why 1.42 or which date it reflects
- **Edge cases:** —
- **Check:** Given no exchange rate in settings, when a 100 USD amount is shown in CAD, then it counts as 142.
- **Disposition:** transform

### R-38  A property's monthly expense total is its standard expenses + additional expenses + credit payments, all as monthly equivalents; annual = monthly × 12

- **Workflow:** W-09
- **Evidence:** `expense_service.dart:90-109`; the Expenses tab shows a "Credit payments" row ("Based on credit payments") beside the total (`expenses_tab.dart:48-57`)
- **Status:** RESOLVED
- **Confidence:** high
- **Edge cases:** see R-39 for currency handling of the additional expenses.
- **Check:** Given taxes 3,600/yr, insurance 100/mo, one additional expense 50/mo and credit payments 1,000/mo, when Monthly is selected, then the total is 1,450.00; Annually shows 17,400.00.
- **Disposition:** transform

### R-39  Additional expenses in another currency are converted before being added to a property's totals

- **Workflow:** W-09, W-16
- **Evidence:** converted in the "Other expenses" figure (`expense_service.dart:76-78`, `calculateCADFactored`) and the overview (`overview_service.dart:108-110`); **not** converted in the monthly/annual total (`expense_service.dart:91-93`, `amountFactored`, `expense.dart:17-23`)
- **Status:** CONFLICT (C-13)
- **Confidence:** high
- **Edge cases:** —
- **Check:** stage 3. Option A: Given rate 1.30 and one 100 USD/month additional expense, when the monthly CAD total is shown, then it includes 130. Option B: it includes 100, as now.
- **Disposition:** transform

### R-40  The period (monthly/annually) and currency (CAD/USD) chosen for a property's totals are applied to every figure on the Expenses tab

- **Workflow:** W-09
- **Evidence:** `expense_totals.dart:19-24, 76-79`; `expense_service.dart:50-64`; held on the app-wide `ExpenseService` (`:13-14`); the dropdown always starts at 'CAD' (`expense_totals.dart:84`), while the period radio starts from the stored value (`:34`)
- **Status:** CONFLICT (C-14)
- **Confidence:** high
- **Edge cases:** after leaving with USD selected, the totals come back in USD while the dropdown shows CAD.
- **Check:** stage 3. Option A (remember): Given USD was chosen, when Expenses is reopened, then the dropdown shows USD and the figures are USD. Option B (reset): both CAD.
- **Disposition:** transform

### R-41  Credit payments for a property are the sum of its borrowings' recurring payment amounts as monthly equivalents

- **Workflow:** W-09, W-16
- **Evidence:** `metric_service.dart:39-41`; `overview_service.dart:77`; shown as "Credit payments" (`expenses_tab.dart:51`)
- **Status:** ASSUMED
- **Confidence:** medium. How the payment's factor is set (borrowing screens) was not traced. `MetricService.init` is not awaited before `calculate` (`expense_service.dart:117-119`), a timing risk (README › Not traced).
- **Edge cases:** shown in the selected currency and period (`expense_service.dart:127-132`).
- **Check:** Given two borrowings paying 1,000/mo and 6,000/yr, when Monthly is selected, then Credit payments shows 1,500.00.
- **Disposition:** transform

### R-42  The unshown "expenses grand total" need not be computed

- **Workflow:** W-09
- **Evidence:** set at `expense_service.dart:83-87`, field `models/expense/expense.dart:4`; never read by any screen (grep)
- **Status:** RESOLVED
- **Confidence:** high
- **Edge cases:** —
- **Check:** Given the Expenses tab, when it renders, then no displayed value comes from `expensesGrandTotal`.
- **Disposition:** retire

---

## W-10 – W-13 Additional expenses

### R-43  The additional expenses list shows the expenses of the property currently being viewed

- **Workflow:** W-10, W-11, W-12, W-13
- **Evidence:** the screen reads `notifier.selectedPropertyId` (`expenses_other_details.dart:24-26`); the route passes `{"expenseId": "0"}` with comment `//TODO - change to propertyId` (`expenses_other_record.dart:45`)
- **Status:** ASSUMED
- **Confidence:** medium. Works only because the selected property is set when details are opened (`property_tile.dart:92`, `property_save_button.dart:55`); the TODO shows the author meant to pass the id explicitly.
- **Edge cases:** —
- **Check:** Given property 1's details are open, when "Other expenses" is tapped, then only property 1's additional expenses are listed.
- **Disposition:** transform

### R-44  An additional expense has a name, amount, currency, date and recurrence (monthly/annually); name, amount and recurrence are required

- **Workflow:** W-11, W-12
- **Evidence:** `fields/form_expense_field.dart:27-99`; new-expense defaults: amount 0, today's date, monthly (`expenses_other_details.dart:130-131`)
- **Status:** RESOLVED
- **Confidence:** high
- **Edge cases:** the expense date is recorded but no calculation uses it; every additional expense counts as recurring at its factor (`expense_service.dart:76-78`).
- **Check:** Given the add dialog with no name, when SAVE is tapped, then nothing is saved and the name shows a required error.
- **Disposition:** transform

### R-45  Adding creates a new expense on the property; editing changes that same expense in place

- **Workflow:** W-11, W-12
- **Evidence:** `property_service.dart:112-121` (id '0' → `add` + store id; otherwise merge into `additionalExpenses/{id}`)
- **Status:** RESOLVED
- **Confidence:** high
- **Edge cases:** id `'0'` is the "new" marker.
- **Check:** Given an expense "Plumber 400", when its amount is edited to 450, then the list still has one "Plumber" entry, now 450.
- **Disposition:** transform

### R-46  Additional expenses are listed most recently added first

- **Workflow:** W-10
- **Evidence:** `property_service.dart:234-237` (`orderBy('timestamp', descending: true)`, the creation time, not the expense date)
- **Status:** ASSUMED
- **Confidence:** medium
- **Edge cases:** an older expense date entered later appears above newer-dated ones.
- **Check:** Given expense A added before B, when the list opens, then B is first.
- **Disposition:** transform

### R-47  After adding an expense, the user is told it has been saved, by name

- **Workflow:** W-11
- **Evidence:** `expenses_other_details.dart:133-140`: the message uses the local placeholder's `expenseName`, which is never set (`:130`), and shows whether the dialog was saved or dismissed
- **Status:** CONFLICT (C-01)
- **Confidence:** high
- **Edge cases:** as written it reads "null has been saved", also after DISMISS.
- **Check:** stage 3. Option A: Given "Plumber" is added, when SAVE is tapped, then the message reads "Plumber has been saved", and no message shows after DISMISS.
- **Disposition:** transform

### R-48  An additional expense is deleted immediately, without confirmation

- **Workflow:** W-13
- **Evidence:** `expenses_other_details.dart:92-99`; contrast the property delete confirmation (`property_delete_button.dart:52-58`)
- **Status:** UNKNOWN (C-08)
- **Confidence:** high on behaviour; nothing says why
- **Edge cases:** —
- **Check:** stage 3. Option A: Given an expense, when Delete is tapped, then a confirmation is asked. Option B: it is removed at once.
- **Disposition:** transform

---

## W-14 View market value and loan-to-value

### R-49  Loan-to-value per property = total amount borrowed against that property ÷ its market value × 100

- **Workflow:** W-14
- **Evidence:** `borrowing_service.dart:88-112`; label "Loan-to-Value (%)" (`market_value_tile.dart:71-72`)
- **Status:** RESOLVED
- **Confidence:** high
- **Edge cases:** both sides are converted to the same currency first (`:101-107`). Zero market value: see R-50.
- **Check:** Given market value 500,000 CAD and borrowings of 300,000 CAD and 50,000 USD at rate 1.30, when Market Value opens in CAD, then LTV is (300,000 + 65,000) / 500,000 × 100 = 73.00.
- **Disposition:** transform

### R-50  A property with no market value has a defined loan-to-value display

- **Workflow:** W-14
- **Evidence:** `borrowing_service.dart:99-112`: market value defaults to 0.0 and the division has no branch for 0, giving Infinity (borrowings > 0) or NaN (no borrowings), formatted by `formatCurrency` (`market_value_tile.dart:72`, `property_record.dart:91-92`)
- **Status:** CONFLICT (C-16)
- **Confidence:** high
- **Edge cases:** —
- **Check:** stage 3 (once C-16 is decided): Given a property with no market value, when Market Value opens, then LTV shows the agreed placeholder.
- **Disposition:** transform

### R-51  Every borrowing records the user and property it belongs to

- **Workflow:** W-14, W-16 (and borrowing screens, out of scope)
- **Evidence:** written on create (`property_service.dart:31-37`) and update (`:46-51`); read by `borrowing_service.dart:62, 89, 139`
- **Status:** RESOLVED
- **Confidence:** high
- **Edge cases:** the market-value and borrowing-list views depend on these fields; the overview walks subcollections instead (`overview_service.dart:136-156`).
- **Check:** Given a new borrowing on property P, when it is stored, then it carries the user's id and P's id.
- **Disposition:** preserve

### R-52  The market value screen shows values in the currency the user chose

- **Workflow:** W-14
- **Evidence:** uses `settings.defaultCurrency` cached at sign-in (`borrowing_service.dart:35, 80`); the overview uses its own switcher (`overview_service.dart:16, 30-33`, `currency_switcher.dart:26-31`); the Expenses tab its own dropdown (R-40)
- **Status:** CONFLICT (C-14)
- **Confidence:** high
- **Edge cases:** switching the overview to USD does not change the market value screen.
- **Check:** stage 3, per the C-14 decision.
- **Disposition:** transform

---

## W-15 Edit market value from the market value list

### R-53  A market value edited on the market value screen is always saved to that property

- **Workflow:** W-15
- **Evidence:** `market_value_tile.dart:49-54`; route `/market_value_list/<selectedIndex>` (`edit_fieldvalue_dialog.dart:158`); only `/market_value_list/0` maps to `propertyDetails` (`property_service.dart:89-90`); any other index falls to `handleSettingsUpdates`, which writes nothing (`:136-138, 203-210`)
- **Status:** CONFLICT (C-17)
- **Confidence:** high on code path; not observed running
- **Edge cases:** if the last details tab opened was not Main (R-23), the edit is silently dropped and "Market value has been updated" still shows (`market_value_tile.dart:55-61`).
- **Check:** Given the last details tab opened was Rent, when a market value is changed to 450,000 here and saved, then the property's stored market value is 450,000.
- **Disposition:** transform

### R-54  After a market value edit, the list and its loan-to-value refresh

- **Workflow:** W-15
- **Evidence:** `market_value_tile.dart:62-63`, `borrowing_service.dart:25-28`, `market_value_list.dart:32-33`
- **Status:** RESOLVED
- **Confidence:** medium. The write is not awaited, so the refresh may read before it lands (README › Not traced).
- **Edge cases:** the refresh also runs after DISMISS (harmless).
- **Check:** Given LTV 60.00 at market value 500,000 with 300,000 borrowed, when market value is saved as 600,000, then LTV shows 50.00.
- **Disposition:** transform

---

## W-16 Portfolio overview totals

### R-55  The overview shows property count, total market value, equity, total credits, funds and cash flow, in CAD or USD, rounded to whole units

- **Workflow:** W-16
- **Evidence:** `overview_service.dart:59-124` (`formatCurrency(..., false)`); tiles `screens/home/widgets/overview.dart:30-114`
- **Status:** RESOLVED
- **Confidence:** high
- **Edge cases:** the CAD/USD switcher always starts at CAD (`currency_switcher.dart:36`) while the service keeps the last choice (`overview_service.dart:16, 30-33`). See C-14.
- **Check:** Given 2 properties worth 400,000 and 600,000 CAD, when Home opens in CAD, then Properties shows 2 and Value shows 1,000,000.
- **Disposition:** transform

### R-56  Equity = total market value − total borrowed

- **Workflow:** W-16
- **Evidence:** `overview_service.dart:82-84`; tile "Equity / Your assets equity" (`overview.dart:63-72`)
- **Status:** RESOLVED
- **Confidence:** high
- **Edge cases:** can be negative.
- **Check:** Given total value 1,000,000 and borrowings 650,000, when Home opens, then Equity shows 350,000.
- **Disposition:** preserve

### R-57  Cash flow = total monthly rent − total monthly expenses (standard + additional + credit payments)

- **Workflow:** W-16
- **Evidence:** `overview_service.dart:95-124`, comment `:120-122` ("expenses need to factor monthly aspect")
- **Status:** RESOLVED
- **Confidence:** high on the formula. The overview converts additional expenses, unlike the per-property total (R-39, C-13).
- **Edge cases:** rent is not factored (R-29).
- **Check:** Given rent 3,000/mo, taxes 3,600/yr, credit payments 1,500/mo, when Home opens, then Cash Flow shows 1,200.
- **Disposition:** preserve

### R-58  The overview is recalculated each time Home is opened

- **Workflow:** W-16
- **Evidence:** `home.dart:34-42` (`didPush` → `signal()` after 500 ms); re-init on sign-in change (`main.dart:69-72`)
- **Status:** ASSUMED
- **Confidence:** medium. The 500 ms delay has no stated reason.
- **Edge cases:** a "calculating" state is shown meanwhile (`overview_service.dart:40-43, 49, 126`).
- **Check:** Given a property's market value was just changed, when Home is opened, then Value reflects the change.
- **Disposition:** transform

---

## Unreachable code

### R-59  No behaviour depends on the unreachable property code

- **Workflow:** none
- **Evidence:** grep across `lib/` finds no caller for `widgets/property_tabs2.dart`, `property_details.dart.bak2`, `models/property/property_main.dart` (`PropertyMainModel`), `models/currency.dart` (used only by `PropertyMainModel` and a comment in `main_tab.dart:27-30`), `PropertyMock` (`models/property/property.dart`; only a comment in `property_service.dart:215-219`), `BorrowingMock` / `BorrowingService.watchAll` / `getBorrowing` (`borrowing_service.dart:158-169`), `config/utils.dart` `setDateField` / `setInputField` / `applyDefaultValues` (`:29-47, 73-78`), `buildCard` (`expenses_tab.dart:143-161`)
- **Status:** RESOLVED
- **Confidence:** high, within `lib/`. Nothing outside `lib/` was searched.
- **Edge cases:** —
- **Check:** Given the rebuild, when it is compared with the old app's reachable screens, then nothing in this list has a counterpart.
- **Disposition:** retire
