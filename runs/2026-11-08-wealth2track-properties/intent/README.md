# wealth2track · properties and their money — intent

> Read backwards with Palimpsest on 2026-11-08. Status: stage 2 of 4.

**For agents:** before changing anything in this module, read `rules.md`. Keep every RESOLVED rule; anything not covered by a rule is free to change. If a change would break a rule, or a rule seems wrong, add an entry to `conflicts.md` instead of guessing.

## Scope

All paths are relative to `wealth2track/lib/`.

- In scope:
  - Screens: `screens/properties/properties_list.dart`, `property_edit.dart`, `property_details.dart`, `expenses_other_details.dart`; `screens/properties/widgets/` → `property_tile.dart`, `property_tabs.dart`, `property_header.dart`, `property_record.dart`, `edit_fieldvalue_dialog.dart`, `expense_totals.dart`, `expenses_other_record.dart`, `tabs/main_tab.dart`, `tabs/rent_tab.dart`, `tabs/expenses_tab.dart`, `buttons/property_save_button.dart`, `buttons/property_delete_button.dart`, `buttons/location_lookup_button.dart`, `fields/*.dart`; `screens/market_value/` (all three files); `screens/home/home.dart`, `screens/home/widgets/overview.dart`, `screens/home/widgets/currency_switcher.dart` (overview totals only).
  - State: `changeNotifiers/property_tab.dart` (`PropertyTabModel`).
  - Services: `firestore/property_service.dart`, `expense_service.dart`, `data_delete_service.dart`, `metric_service.dart`, `overview_service.dart`, `borrowing_service.dart` (only `getMarketValues`, `getBorrowings`, `getProperties`, `refreshMarketValues`, `init`).
  - Models: `models/property/*` (header, details, renting, expenses, expense, money, address, property_type + generated `.g.dart`), `models/expense/expense.dart` (`ExpenseModel`), `models/marketValue/market_value_model.dart`, `models/metric/metrics.dart`, `models/borrowing/borrowing_header.dart`, `borrowing_details.dart` (read-only use), `models/user/settings.dart` (exchange rate, default currency).
  - Helpers: `config/utils.dart` (`formatCurrency`, `formatDate`, `calculateAgeInYears`, `calculateRenewalDate`, `handleText`), `screens/shared/fullAppbar.dart` (back navigation).
- Calls out to:
  - Firestore paths: `users/{uid}` (reads `settings.exchangeRate`, `settings.defaultCurrency`); `users/{uid}/properties/{id}`; `users/{uid}/properties/{id}/additionalExpenses/{id}`; `users/{uid}/properties/{id}/borrowings/{id}` (read, and deleted on property delete); collection group `borrowings` filtered by `userId`; `users/{uid}/funds` (read by overview only).
  - Google Places autocomplete + `geocoder` (`buttons/location_lookup_button.dart`).
  - `services/auth.dart › AuthService.getUser` (from `PropertyTabModel.setPropertyHeader`).
- Called by: `config/routes.dart` (named routes), `main.dart:41-81` (Provider wiring: `PropertyService`, `DataDeleteService` rebuilt per `AppUser`; `MetricService`, `OverviewService`, `ExpenseService`, `BorrowingService` re-`init`ed per `AppUser`), Home overview tiles.
- Left out (not in this scope): borrowing create/edit/delete (Credits tab, `borrowing_details.dart`, `borrowing_edit.dart`, `screens/borrowing/`), funds, auth/login, onboarding/intro, settings, profile.

## Sources read

- Code: the `lib/` files listed under Scope, plus `main.dart`, `config/routes.dart`, `config/globals.dart`, `screens/home/widgets/app_drawer.dart` (entry links only), `screens/properties/widgets/tabs/borrowing_tab.dart` (navigation lines only), `models/index.dart`.
- `pubspec.yaml` (package versions: `cloud_firestore ^0.14.4`, `provider ^4.0.4`, `flutter_form_builder ^4.0.2`, `flutter_slidable ^0.5.4`, `flutter_google_places`, `geocoder`).
- Grep across `lib/` for cross-references (who calls which service method).
- No screens run, no data samples, no logs, no people asked.

## Workflows

| ID | Workflow | Entry point | Purpose (stage 2) |
| --- | --- | --- | --- |
| W-01 | List properties | Home › "Properties" tile → `PropertiesListScreen` (`/properties_list`) | See all my properties at a glance, with their market value, and get to each one. |
| W-02 | Create a property | Properties list › "+" FAB → `PropertyEditScreen` › SAVE AND NEXT | Record a new property I own so I can track its money. |
| W-03 | Edit a property's header (address, description, building type) | Properties list › slide "Edit", or tap property name → `PropertyEditScreen` › SAVE / popup | Correct or update where a property is and what kind of building it is. |
| W-04 | Look up an address | `PropertyEditScreen` › "Address Lookup" button | Fill in a property address without typing it, from a lookup. |
| W-05 | Delete a property and its related data | Properties list › slide "Delete" → dialog › CONTINUE | Remove a property I no longer want tracked, together with everything recorded against it. |
| W-06 | View a property's details | Properties list › tile arrow → `PropertyDetailsScreen` (`/property_details`) | See everything recorded about one property: purchase, value, credits, rent and expenses. |
| W-07 | Edit a property detail or rent field | Details › Main or Rent tab › tap a value → edit dialog › SAVE | Record or update one fact about a property (value, purchase, type, rent terms). |
| W-08 | Edit a standard expense (taxes, maintenance, management fee, insurance) | Details › Expenses tab › tap a value → edit dialog › SAVE | Record what a property costs me in recurring taxes, fees and insurance. |
| W-09 | View a property's expense totals (monthly/annual, CAD/USD) | Details › Expenses tab › `ExpenseTotals` (load, radio, currency dropdown) | Know what one property costs me per month or per year, in CAD or USD, including credit payments. |
| W-10 | Open the additional ("other") expenses list | Expenses tab › tap "Other expenses" value → `ExpensesOtherDetailsScreen` | See the irregular or extra costs recorded against a property. |
| W-11 | Add an additional expense | Expenses (Other) › "+" FAB → edit dialog › SAVE | Record another cost of a property that is not one of the standard expenses. |
| W-12 | Edit an additional expense | Expenses (Other) › tap amount → edit dialog › SAVE | Correct a recorded additional cost. |
| W-13 | Delete an additional expense | Expenses (Other) › slide "Delete" | Remove an additional cost recorded by mistake. |
| W-14 | View market value and loan-to-value per property | Home › "Value" tile → `MarketValueListScreen` (`/market_value_list`) | See how much each property is worth and how much of it is borrowed against (loan-to-value). |
| W-15 | Edit market value from the market value list | Market Value › tap amount → edit dialog › SAVE | Update a property's market value quickly while looking at loan-to-value. |
| W-16 | Compute portfolio overview totals | `HomeScreen.didPush` → `OverviewService.signal`; currency switcher | See my whole portfolio in one place: count, value, equity, credits, funds and cash flow. |

## Traces

### W-01 List properties

1. Home "Properties" tile pushes `/properties_list` — `screens/home/widgets/overview.dart:31-35`.
2. `initState` subscribes to `PropertyService.watchProperties()` — `properties_list.dart:32-36`.
3. Stream of `users/{uid}/properties` ordered by `timestamp` desc, mapped to `PropertyHeaderModel` — `property_service.dart:66-75`.
4. Empty → "No Property Added Yet" — `properties_list.dart:72-74`; else one `Slidable` per property — `properties_list.dart:113-157`.
5. Tile shows `propertyName`, `buildingType`, and `propertyDetails.marketValue.amount` formatted, or `N/A` when `propertyDetails` is null — `properties_list.dart:125-133`, `property_tile.dart:84`.
6. `propertyName` is not stored; it is a getter built from address as `line1, city, province, postal` with `<...>` placeholders — `models/property/property_header.dart:20-28`.
7. Ends: screen.

### W-02 Create a property

1. FAB calls `PropertyTabModel.setPropertyHeader('0')` then replaces route with `/property_edit`, args `{propertyId: "0"}` — `properties_list.dart:49-55`.
2. `setPropertyHeader('0')` gets the user from `AuthService`, sets `propertyHeader` to `getPropertyEmpty()` — `changeNotifiers/property_tab.dart:15-27`; empty model has `address.country = 'Canada'`, all else null — `property_service.dart:245-250`.
3. Edit screen title "Add New Property" when `propertyId == '0'` — `property_edit.dart:35`; form built from `notifier.propertyHeader` future — `property_edit.dart:49-80`.
4. Fields and validators: building type (dropdown of 5, required), description, address line 1 (required), line 2, city (required), postal, province (required), country (Canada/USA/Other, required) — `property_edit.dart:83-167`. `initialValue` sets a `property_name` key with no matching field — `property_edit.dart:71`.
5. Optional address lookup → W-04 — `property_edit.dart:105`.
6. CANCEL resets the form only — `property_edit.dart:184-186`.
7. "SAVE AND NEXT" → `_handlePropertySave(..., '/property_details')` — `property_save_button.dart:26, 36-37`.
8. `saveAndValidate()`; maps form into `PropertyHeaderModel(null, desc, address, buildingType)` — `property_save_button.dart:49, 122-139`.
9. `addProperty`: `add({...toJson(), timestamp: now})` to `users/{uid}/properties`, then `updateData({id: documentID})` — `property_service.dart:23-28`. `toJson` includes `id: null` and null `propertyDetails`/`propertyRenting`/`propertyExpenses` — `property_header.g.dart:39-47`. Add error is printed (`catchError(print)`) — `property_service.dart:24`.
10. On success: `setSelectedProperty(newId)` and replace route with `/property_details`, `{propertyId: newId}` — `property_save_button.dart:53-58`.
11. Snackbar "Success processing request." is shown right after the save call is started, not after it completes — `property_save_button.dart:69-73`.
12. Ends: Firestore write (property doc) + Property Details screen.

### W-03 Edit a property's header

1. Entry A: slide "Edit" → `setPropertyHeader(prop.id)` → `/property_edit` `{propertyId}` — `properties_list.dart:136-146`. Entry B: tap property name → same — `property_tile.dart:35-41`. Neither calls `setSelectedProperty` — (contrast `property_tile.dart:92`).
2. `setPropertyHeader(id)` → `PropertyService.getProperty(id)` one-off read — `property_tab.dart:23-25`, `property_service.dart:221-223`.
3. Title "Edit Property" — `property_edit.dart:35`. Same form as W-02 step 4.
4. Save button label "SAVE" plus a popup with "SAVE AND GO TO DETAILS" / "SAVE AND GO BACK TO LIST" — `property_save_button.dart:26, 32, 77-106`.
   - SAVE → after-save route `/property_edit` — `property_save_button.dart:38-39`.
   - Popup 1 → `/property_details`; popup 2 → `/properties_list` — `property_save_button.dart:95-100`.
5. `updateProperty`: `toJson()` minus `propertyDetails`, `propertyRenting`, `propertyExpenses`, then `updateData({...mapped, updatedAt: now})` — `property_service.dart:55-64`. Written keys: `id`, `buildingType`, `propertyDesc`, `address` (whole map) — `property_header.g.dart:39-43`.
6. Then replace route with chosen screen, `{propertyId}` — `property_save_button.dart:60-67`; snackbar as W-02 step 11.
7. `notifier.propertyHeader` is not refreshed by the save; the edit screen reads that future — `property_save_button.dart:45-75`, `property_edit.dart:49`.
8. Ends: Firestore update (property doc) + chosen screen.

### W-04 Look up an address

1. "Address Lookup" button → `PlacesAutocomplete.show` with country component `ca` and a key literal in source — `location_lookup_button.dart:11-13, 24-36`.
2. Selected prediction → `Geocoder.local.findAddressesFromQuery(description)`, first result used — `location_lookup_button.dart:42-51`.
3. Builds `PropertyHeaderModel` with id from `notifier.selectedPropertyId` (null if `'0'`), current desc and building type, and the geocoded address (line1 = `subThoroughfare thoroughfare`, line2 = `subAdminArea`, city = `locality`, province = `adminArea`, country = `countryName`) — `location_lookup_button.dart:56-73`.
4. `notifier.setPlaceAddress(model)` replaces `propertyHeader` with a completed future and notifies — `property_tab.dart:40-47`; edit screen rebuilds from it — `property_edit.dart:33, 49`.
5. Ends: in-memory state + screen (nothing written until W-02/W-03 save).

### W-05 Delete a property and its related data

1. Slide "Delete" (`PropertyDeleteButton`) → `showAlertDialog` — `properties_list.dart:135`, `property_delete_button.dart:17-25`.
2. Dialog "Deleting Property?" / "This operation can't be undone…", CANCEL closes slidable and pops — `property_delete_button.dart:32-38, 52-65`.
3. CONTINUE → `DataDeleteService.deleteProperty(propertyId)`, then pop via `appGlobals.scaffoldKey` context — `property_delete_button.dart:40-48`.
4. `deleteProperty` awaits, in order: `deleteAdditionalExpenses`, `deleteBorrowings`, then deletes `users/{uid}/properties/{id}` — `data_delete_service.dart:21-25`.
5. `deleteAdditionalExpenses` reads subcollection **`additionalProperties`** and deletes each doc — `data_delete_service.dart:27-33`. Expenses are written to and read from **`additionalExpenses`** elsewhere — `property_service.dart:116, 120, 235, 264`; `expense_service.dart:145`; `overview_service.dart:171`.
6. `deleteBorrowings` reads `borrowings` and deletes each doc — `data_delete_service.dart:35-41`.
7. In both, the `get().then(...)` future is not returned or awaited, so `deleteProperty`'s `await` does not wait for the per-doc deletes — `data_delete_service.dart:28, 36`.
8. List updates through the W-01 stream — `property_service.dart:66-75`.
9. Ends: Firestore deletes (borrowings, property doc; `additionalProperties` docs if any).

### W-06 View a property's details

1. Tile arrow → `setSelectedProperty(id)` → `/property_details` `{propertyId}` — `property_tile.dart:90-96`.
2. Each `build`: `watchProperty(id)` stream and `MetricService.calculate(id)` — `property_details.dart:36-41`, `property_service.dart:225-227`.
3. `MetricService.calculate` sums `borrowingDetails.recurringPaymentAmount.calculateCADFactored` over the property's borrowings and writes to `metrics` — `metric_service.dart:31-53`. No screen reads `MetricService.metrics` (grep: only `property_details.dart:40` references `MetricService` in `screens/`).
4. Header (description, building type) + tab bar `Main | Credits | Rent | Expenses` — `property_details.dart:67-92`, `property_tabs.dart:13`.
5. Tab tap → `notifier.setSelectedTab(name, index)` — `property_tabs.dart:27-31`, `property_tab.dart:29-33`. Selected tab/index live on the app-wide notifier and are not reset when a different property is opened (only setter call site: `property_tabs.dart:30`).
6. Main tab rows: market value, purchase date, purchase price, sales associate cost (editable); property age = days since purchase / 365 (read-only); property type radio (Primary Residence / Secondary Residence / Investment) — `main_tab.dart:36-100`, `utils.dart:80-91`, `property_type.dart:9-22`.
7. Rent tab rows: rent amount, start date, term in months, renewal date = start + term months (read-only), last rent increase, rent increase limit — `rent_tab.dart:26-88`, `utils.dart:60-67`.
8. Credits tab → `BorrowingTab` (out of scope) — `property_details.dart:79-80`.
9. Back arrow → `/properties_list` — `property_details.dart:58`, `fullAppbar.dart:15-21`.
10. Ends: screen.

### W-07 Edit a property detail or rent field

1. Tap a value in a `PropertyRecord` with non-null `controlType` → `editFieldValueDialog(...)` — `property_record.dart:57-69`.
2. Dialog captures the current route name and args (`/property_details`, `{propertyId}`) — `edit_fieldvalue_dialog.dart:19-21`; renders a field by control type (Currency, Datepicker, Number, Text, Radio, Expense) — `edit_fieldvalue_dialog.dart:97-140`.
3. SAVE → `saveAndValidate()`; builds `route = '<routeName>/<notifier.selectedIndex>'` — `edit_fieldvalue_dialog.dart:155-158`.
4. Per type — `edit_fieldvalue_dialog.dart:162-185`:
   - Currency: `Money(amount, currency)`, `factor = int.parse(value['factor'] ?? '0')` → `updateMoneyField`. Main/Rent currency fields do not show the frequency radio, so `factor` is stored as `0` — `property_record.dart:26`, `form_currency_field.dart:69`.
   - Datepicker → `updateDateField`; Radio → `updateInputField`; Number → `updateNumberField` (`double.parse`).
5. Service resolves node from route: `/property_details/0` → `propertyDetails`, `/property_details/2` → `propertyRenting`, `/property_details/3` → `propertyExpenses`, `/borrowing_details/1` → `borrowingDetails`, `/market_value_list/0` → `propertyDetails`, `/expenses_other_details` → `additionalExpenses`, contains `/settings/` → `settings` — `property_service.dart:86-110`.
6. Property nodes: `setData({node: {field: value}}, merge: true)` on the property doc — `property_service.dart:123-201` (e.g. money: `{amount, currency, factor}` at `:127-131`). Unresolved node → `handleSettingsUpdates`, which writes only if route contains `/settings/` — `property_service.dart:136-138, 203-210`.
7. Rent "rent increase limit" saves under field name `rentIncreaseLimit` — `rent_tab.dart:83`; the model reads `rentIncreaseLimitPercent` — `property_renting.dart:14`, `property_renting.g.dart:24-25`.
8. Service call is not awaited; dialog pops with "`<label>` has been updated." — `edit_fieldvalue_dialog.dart:168-198`; record shows it as a snackbar — `property_record.dart:71-77`.
9. Details screen re-renders from the W-06 `watchProperty` stream.
10. Ends: Firestore merge-write on the property doc.

### W-08 Edit a standard expense

1. Expenses tab rows for `propertyTaxes`, `maintenanceFee`, `propertyManagementFee`, `insurance`, control `Currency`, `showCycleCurrencyField: true` — `expenses_tab.dart:62-109`.
2. Displayed value is `ExpenseService.getFieldCalculatedValue(field)` when available, else the raw field — `expenses_tab.dart:67, 134-141`, `property_record.dart:51`. Calculation: `calculateCADFactored(rate) × selectedExpensePeriod`, × `1/rate` when currency is USD; result keeps the field's original currency label — `expense_service.dart:32-48`.
3. `calculateCADFactored` = amount (× rate if `USD`) / `factor` (null → 1) — `money.dart:19-34`. Rate = `users/{uid}.settings.exchangeRate`, default `1.42` — `expense_service.dart:35`.
4. Dialog shows amount (required, numeric), currency (USD/CAD, default CAD), and Frequency radio Annually=`12` / Monthly=`1` (required) — `form_currency_field.dart:34-80`.
5. Save as W-07 steps 3-6 with route `/property_details/3` → node `propertyExpenses` → `setData({propertyExpenses: {field: {amount, currency, factor}}}, merge)` — `property_service.dart:98-99, 127-131`.
6. Ends: Firestore merge-write on the property doc.

### W-09 View a property's expense totals

1. `ExpenseTotals.didChangeDependencies` → `ExpenseService.calculateExpenses(notifier.selectedPropertyId)` — `expense_totals.dart:32-40`.
2. `calculateExpenses` — `expense_service.dart:66-114`:
   1. Reads user doc for rate (default 1.42), the property doc, and all `additionalExpenses` — `:67-72, 137-150`.
   2. `additionalExpensesTotal` = Σ additional `calculateCADFactored` × period; USD variant × 1/rate — `:75-80`.
   3. `expensesGrandTotal` = that sum (unmultiplied) + the four standard expenses factored — `:83-87`. Not displayed by any screen (grep).
   4. `expenseMonthlyTotal` = Σ additional `amountFactored` (amount/factor, no currency conversion — `expense.dart:17-23`) + four standard expenses `calculateCADFactored` + monthly borrowing payments — `:90-105`.
   5. Monthly borrowing payments: new `MetricService`, `init(user)` (not awaited), `calculate(propertyId)` → Σ borrowings' `recurringPaymentAmount.calculateCADFactored` — `:116-120`, `metric_service.dart:31-41`. Sets `borrowingPaymentTotal` (monthly or ×12, CAD or USD) — `:121-132`. Called twice — `:96, :111`.
   6. Annual = monthly × 12; USD variants × 1/rate — `:106-109`. `notifyListeners()` — `:113`.
3. Display: big number = monthly or annual total in the selected currency — `expense_totals.dart:55-69`; "Credit payments" row = `borrowingPaymentTotal` (read-only) — `expenses_tab.dart:48-57`; "Other expenses" row = `additionalExpensesTotal(USD)` — `expenses_tab.dart:114-120`.
4. Monthly/Annually radio → `setSelectedExpensePeriod(1|12, id)` → recalculates — `expense_totals.dart:19-24, 102-113`, `expense_service.dart:50-56`.
5. Currency dropdown (USD/CAD) → `setSelectedCurrency` → recalculates — `expense_totals.dart:76-79`, `expense_service.dart:58-64`.
6. Period and currency are fields on the app-wide `ExpenseService` (initial 1 / `CAD`) — `expense_service.dart:13-14`; the dropdown's `initialValue` is always `'CAD'` — `expense_totals.dart:84`.
7. Ends: screen (no writes).

### W-10 Open the additional expenses list

1. Tap the "Other expenses" value → replace route with `/expenses_other_details`, args `{expenseId: "0"}` — `expenses_other_record.dart:42-46`.
2. Screen uses `notifier.selectedPropertyId` (not the route args) → `watchAdditionalExpenses(id)` — `expenses_other_details.dart:24-26`.
3. Stream of `.../additionalExpenses` ordered by `timestamp` desc → `Expense` — `property_service.dart:234-243`.
4. Empty → "No Expense Added Yet" + FAB — `expenses_other_details.dart:39-44`; else rows with name, date (`d-MMM-y`), amount — `expenses_other_details.dart:79-109`, `expenses_other_record.dart:66-99`.
5. Back arrow → `/property_details` `{propertyId: selectedPropertyId}` — `expenses_other_details.dart:41, 78`.
6. Ends: screen.

### W-11 Add an additional expense

1. FAB → local `Expense(id: '0', amount: 0, expenseDate: now)`, `factor = 1` → `editFieldValueDialog(..., 'Expense', ...)` — `expenses_other_details.dart:129-133`.
2. Dialog fields: expense name (required), amount (required, numeric), currency (default CAD), expense date, Recurrence Annually=`12` / Monthly=`1` (required) — `form_expense_field.dart:27-99`.
3. SAVE builds `Expense(id: '0', amount, currency, expenseName, expenseDate)`, `factor` → `updateExpenseField(selectedPropertyId, expense)` (not awaited) — `edit_fieldvalue_dialog.dart:186-195`.
4. `id == '0'` → `add({...json, timestamp: now})` to `.../additionalExpenses`, then `updateData({id})` — `property_service.dart:112-119`. Stored keys: `id, amount, currency, expenseName, expenseDate, factor` — `expense.g.dart:20-27`.
5. After the dialog closes (SAVE or DISMISS), snackbar "`${field.expenseName}` has been saved" using the local object, whose `expenseName` was never set — `expenses_other_details.dart:130, 135-140`.
6. Ends: Firestore add on `additionalExpenses` + list refresh via stream.

### W-12 Edit an additional expense

1. Tap amount on a row → `editFieldValueDialog(context, name, expense, 'Expense', ...)` — `expenses_other_record.dart:86-87`.
2. Dialog prefilled from the expense — `form_expense_field.dart:29, 44, 64, 84, 92`.
3. SAVE → new `Expense` with the existing id → `updateExpenseField` — `edit_fieldvalue_dialog.dart:186-195`.
4. Non-'0' id → `setData(json, merge: true)` on `.../additionalExpenses/{id}` — `property_service.dart:120`.
5. Snackbar with dialog result unless `'null'` (DISMISS) — `expenses_other_record.dart:89-96`.
6. Ends: Firestore merge-write on the expense doc.

### W-13 Delete an additional expense

1. Slide "Delete" → `PropertyService.deleteExpense(expense.id, propertyId)`; no confirmation — `expenses_other_details.dart:92-99`.
2. `delete()` on `.../additionalExpenses/{id}` — `property_service.dart:263-265`.
3. Ends: Firestore delete + list refresh via stream.

### W-14 View market value and loan-to-value

1. Home "Value" tile → `/market_value_list` — `overview.dart:49-52`.
2. Each `build` (Provider `listen: true`) calls `BorrowingService.getMarketValues()` — `market_value_list.dart:32-33`.
3. `getMarketValues` — `borrowing_service.dart:79-119`:
   1. `defaultCurrency` and `exchangeRate` from `dbUser.settings` loaded at `init` (defaults `CAD`, `1.42`) — `:80-82`, `:30-36`.
   2. All borrowings via collection group `borrowings` where `userId == uid` — `:84, 136-144`; all properties ordered by `timestamp` desc — `:85, 146-156`.
   3. Per property: loan = Σ its borrowings' `borrowingAmount.calculateCAD` (USD variant × 1/rate) — `:88-97`.
   4. Market value = `propertyDetails.marketValue.calculateCAD` (× 1/rate if default currency is not CAD), 0 if absent — `:99-107`.
   5. `loanToValuePercentage = loan / marketValue × 100`; no branch for marketValue = 0 — `:112`.
   6. Emits `MarketValueModel(propertyId, propertyName, amount, LTV, currency, marketValueOrig)` — `:109-115`.
4. `FutureBuilder` → empty "No Data Added Yet", else one `MarketValueTile` per property — `build_list_view.dart:8-71`.
5. Tile shows name, "Market Value (`<currency>`)" amount, "Loan-to-Value (%)" — `market_value_tile.dart:39-74`.
6. Ends: screen (no writes).

### W-15 Edit market value from the market value list

1. Tap amount → `setSelectedProperty(propertyId)` → `editFieldValueDialog(context, 'marketValue', marketValueOrig, 'Currency', ..., showCycle false)` — `market_value_tile.dart:49-54`.
2. Route captured as `/market_value_list`; on SAVE becomes `/market_value_list/<notifier.selectedIndex>` — `edit_fieldvalue_dialog.dart:19-21, 158`.
3. Only index `0` resolves to `propertyDetails` — `property_service.dart:89-90`; then `setData({propertyDetails: {marketValue: {amount, currency, factor: 0}}}, merge)` — `property_service.dart:127-131`, `edit_fieldvalue_dialog.dart:164`. Any other index → `handleSettingsUpdates`, no write — `property_service.dart:136-138, 203-210`. `selectedIndex` is the last Property Details tab chosen — `property_tab.dart:29-33`.
4. After the dialog closes (SAVE or DISMISS): snackbar "Market value has been updated" and `refreshMarketValues()` → `notifyListeners()` → screen rebuilds → W-14 re-runs — `market_value_tile.dart:55-63`, `borrowing_service.dart:25-28`.
5. Ends: Firestore merge-write on the property doc (when index is 0) + screen.

### W-16 Compute portfolio overview totals

1. `HomeScreen.didPush` → after 500 ms `OverviewService.signal()` → `init(user)` — `home.dart:35-42`, `overview_service.dart:35-38`. Also `init` on every `AppUser` change — `main.dart:69-72`.
2. `init` — `overview_service.dart:48-128`:
   1. Rate from user settings (default 1.42) — `:55-57`.
   2. Property count — `:60-61`.
   3. Market value total = Σ `marketValue.calculateCAD` — `:64-69`.
   4. Borrowing total = Σ `borrowingAmount.calculateCAD`; recurring payments = Σ `recurringPaymentAmount.calculateCADFactored` (per-property subcollection reads) — `:72-80, 136-156`.
   5. Equity = market value − borrowing — `:83-84`.
   6. Funds total (out of scope, read only) — `:87-93`.
   7. Expenses = Σ four standard expenses factored + Σ additional expenses factored + recurring payments — `:96-112`.
   8. Rent total = Σ `rentingAmount.calculateCAD` (not factored) — `:115-118`.
   9. Cash flow = rent − expenses — `:120-124`.
3. Values formatted without decimals; CAD and USD (× 1/rate) variants — `:68-124`, `utils.dart:49-52`.
4. Tiles: Properties, Value, Equity, Credits, Funds, Cash Flow; currency switcher picks CAD/USD variant (`setSelectedCurrency`, display only) — `overview.dart:18-116`, `currency_switcher.dart:26-31`, `overview_service.dart:30-33`.
5. Ends: screen (no writes).

## Not traced

- **Borrowing CRUD** (Credits tab, `borrowing_details.dart`, `borrowing_edit.dart`, `screens/borrowing/`): out of scope. Borrowings are only read here (W-06, W-09, W-14, W-16) and deleted in W-05. Field edits on borrowings share the W-07 path (`/borrowing_details/1` → `borrowingDetails`, `property_service.dart:92-93, 132-135`) and were not followed further.
- **Firestore platform behaviour**: whether deleting a document removes its subcollections, security rules, and whether the collection-group `userId` query (`borrowing_service.dart:137-140`) has an index. None of this is in `lib/`.
- **Timing / ordering**:
  - Writes from the edit dialog are not awaited before the dialog pops (`edit_fieldvalue_dialog.dart:168-195`). Whether W-15's refresh reads before or after the write lands was not traced.
  - The inner deletes in W-05 are not awaited (`data_delete_service.dart:28, 36`). The order in which Firestore applies them relative to the property-doc delete was not traced.
  - `MetricService.init` is not awaited inside `ExpenseService._getBorrowingPayments` (`expense_service.dart:117-119`). `init` sets `_propertiesCollection` after an await (`metric_service.dart:23-27`), and `calculate` uses it (`:56`). Which one finishes first was not traced.
  - The 500 ms delay before overview refresh (`home.dart:38`) and the re-`init` of four services on every `AppUser` emission (`main.dart:65-80`).
  - `BorrowingService`/`ExpenseService` cache `dbUser` (rate, default currency) at `init` (`borrowing_service.dart:35`, `expense_service.dart:29`), while some methods re-read it per call (`expense_service.dart:67`, `metric_service.dart:32`, `overview_service.dart:55`). Behaviour after a settings change was not traced.
- **Depends on app-wide state or data**:
  - Route-to-node resolution depends on `PropertyTabModel.selectedIndex`, which carries over between screens and properties (W-07, W-15).
  - `selectedPropertyId` is set only in `property_tile.dart:92`, `property_save_button.dart:55` and `market_value_tile.dart:52`, so W-04, W-09, W-10, W-11 read whatever was last set.
  - Stored `factor` values of `0`/null and their effect wherever `calculateCADFactored` is applied to Main/Rent fields (it is not applied to them in the traced paths).
  - Documents written by older code versions (field names, missing `timestamp` for the `orderBy` queries).
  - Market value of 0 in W-14.
- **Framework behaviour**: whether `FormBuilder` re-applies a changed `initialValue` after W-04's `setPlaceAddress`; `Slidable.of(context)` called from a dialog context (`property_delete_button.dart:35, 43`); `Scaffold.of(context)` after async gaps.
- **External calls**: Google Places autocomplete and `Geocoder.local` responses (`location_lookup_button.dart:27-50`). The API key is a literal in source (`:11`). Firebase Analytics/Messaging setup in `main.dart` was not followed.
- **Code present but not reached from any traced entry point** (per grep): `property_tabs2.dart`, `property_details.dart.bak2`, `models/property/property_main.dart` (`PropertyMainModel`), `models/currency.dart` (only from `PropertyMainModel` and a comment), `PropertyMock`, `BorrowingMock` / `BorrowingService.watchAll` / `getBorrowing`, `utils.dart` `setDateField` / `setInputField` / `applyDefaultValues`, the `buildCard` widget in `expenses_tab.dart:143`, `ExpenseModel.expensesGrandTotal` (computed, never shown), and `MetricService.metrics` from the Provider instance (computed in W-06, never read).
