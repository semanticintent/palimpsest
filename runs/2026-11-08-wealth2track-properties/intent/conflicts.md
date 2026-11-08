# Conflicts and unknowns

Every place where the sources disagree, or where nothing says why. Stage 2 asks; stage 3 answers.

All paths are relative to `wealth2track/lib/`. Order: likely data loss or wrong numbers first, then UI behaviour, then unknowns.

### C-05  Property delete leaves its additional expenses behind

- **Rules:** R-17
- **What disagrees:** the delete service's own comment says it deletes the property's additional expenses (`firestore/data_delete_service.dart:16-19`), but the code deletes from a subcollection named `additionalProperties` (`:28`). Expenses are stored in, and read from, `additionalExpenses` (`property_service.dart:116, 120, 235, 264`; `expense_service.dart:145`; `overview_service.dart:171`).
- **Question:** when a property is deleted, must all of its additional expenses be deleted too?
- **Decision:** open (stage 3)

### C-06  Expenses already left behind by past property deletes

- **Rules:** R-17
- **What disagrees:** if C-05 is a bug, live data probably holds additional expenses whose property no longer exists. Nothing in `lib/` reads or cleans them up; no data sample was available to check.
- **Question:** should the rebuild's data migration find and remove (or keep) additional expenses whose property has been deleted?
- **Decision:** open (stage 3)

### C-17  Market value edits on the market value screen can be silently dropped

- **Rules:** R-53 (via R-23, R-26)
- **What disagrees:** the screen invites editing the market value and then says "Market value has been updated" (`screens/market_value/widgets/market_value_tile.dart:49-61`). The save only happens when the app-wide tab index is 0 (Main): `/market_value_list/0` → `propertyDetails` (`property_service.dart:89-90`). Any other last-opened tab leads to `handleSettingsUpdates`, which writes nothing (`:136-138, 203-210`).
- **Question:** should a market-value edit on this screen always be saved, whichever property tab was opened last?
- **Decision:** open (stage 3)

### C-13  USD additional expenses are not converted in the per-property monthly total

- **Rules:** R-39 (also R-38, R-57)
- **What disagrees:** the "Other expenses" figure and the overview convert additional expenses to CAD (`expense_service.dart:76-78`, `overview_service.dart:108-110`, `calculateCADFactored`). The per-property monthly/annual total adds them unconverted (`expense_service.dart:91-93`, `amountFactored`). The same expense therefore counts differently on the Expenses tab total and on the overview's cash flow.
- **Question:** should additional expenses entered in USD be converted at the user's rate before they are added to a property's monthly and annual totals?
- **Decision:** open (stage 3)

### C-11  Rent increase limit saved under one name, read under another

- **Rules:** R-30
- **What disagrees:** the Rent tab saves the field as `rentIncreaseLimit` (`screens/properties/widgets/tabs/rent_tab.dart:83`). The model and the display read `rentIncreaseLimitPercent` (`models/property/property_renting.dart:14`, `property_renting.g.dart:24-25`, `rent_tab.dart:85`), so a saved limit is never shown.
- **Question:** which stored field is the real rent increase limit (a percentage): `rentIncreaseLimit` (what is written) or `rentIncreaseLimitPercent` (what is read), and should existing values under the other name be carried over?
- **Decision:** open (stage 3)

### C-07  Property delete is not all-or-nothing

- **Rules:** R-18
- **What disagrees:** `deleteProperty` awaits each step in order (`data_delete_service.dart:22-24`), showing an intent to finish children before the parent. The child deletes are started inside a `.then` that is neither returned nor awaited (`:28, :36`), so the property can go first, and a failure part-way leaves partial data.
- **Question:** must deleting a property be all-or-nothing, so that an interrupted delete never leaves orphaned borrowings or expenses?
- **Decision:** open (stage 3)

### C-10  "No frequency" money values are stored with factor 0, but the model treats missing as 1

- **Rules:** R-28 (also R-34)
- **What disagrees:** the edit dialog stores `factor = 0` when no frequency control is shown (`widgets/edit_fieldvalue_dialog.dart:164`), e.g. for market value, purchase price, sales cost and rent. The model's factored calculation treats a missing factor as 1 (`models/property/money.dart:32`) and divides by it (`:33`).
- **Question:** for money values that have no frequency (market value, purchase price, sales cost, rent), should the stored factor be "none / 1" rather than 0?
- **Decision:** open (stage 3)

### C-16  Loan-to-value when a property has no market value

- **Rules:** R-50
- **What disagrees:** LTV is shown as a percentage (`market_value_tile.dart:71-72`), but with a market value of 0 or none the code divides by 0 (`firestore/borrowing_service.dart:99-112`). The result is Infinity (with borrowings) or NaN (without), passed straight to the number formatter.
- **Question:** what should loan-to-value show for a property with no market value recorded?
- **Decision:** open (stage 3)

### C-15  The property list shows market value as entered, unconverted and without currency

- **Rules:** R-04
- **What disagrees:** the list shows the raw stored amount with no currency label (`screens/properties/properties_list.dart:130-133`). The market value screen and overview convert to the user's currency (`borrowing_service.dart:99-107`, `overview_service.dart:64-69`). If details exist but no market value, the list shows `0.00` instead of the `<Market value>` placeholder (`config/utils.dart:51`, `property_tile.dart:84`).
- **Question:** should the property list show each market value converted to the user's chosen currency (with its label), or exactly as entered?
- **Decision:** open (stage 3)

### C-14  Three separate currency choices

- **Rules:** R-40, R-52, R-55
- **What disagrees:** the overview has its own CAD/USD switch, kept in the service but always displayed as CAD on return (`currency_switcher.dart:26-36`, `overview_service.dart:16, 30-33`). The Expenses tab has another, likewise kept but displayed as CAD (`expense_totals.dart:76-84`, `expense_service.dart:14`), while its period radio does show the kept value (`expense_totals.dart:34`). The market value screen ignores both and uses `settings.defaultCurrency` from sign-in (`borrowing_service.dart:35, 80`). So what a control shows can differ from the figures beside it, and screens can disagree.
- **Question:** should the app have one currency choice (the settings default) that every screen follows, or a per-screen choice, and if per-screen, should it be remembered between visits?
- **Decision:** open (stage 3)

### C-01  Success messages are shown whether or not anything was saved

- **Rules:** R-09, R-32, R-47
- **What disagrees:** the messages say the work was done, but:
  - "Success processing request." is shown before the property save completes, and errors are only printed (`buttons/property_save_button.dart:69-73`, `property_service.dart:24`).
  - "`<label>` has been updated." is shown without awaiting the write, including when the route resolves to no write (`edit_fieldvalue_dialog.dart:162-198`, `property_service.dart:136-140`).
  - "Market value has been updated" shows after DISMISS too (`market_value_tile.dart:54-61`).
  - "`null` has been saved" shows after adding an expense, also after DISMISS (`expenses_other_details.dart:130-140`).
- **Question:** must the user be told when a save fails or does not happen, and see a success message only after the data is actually stored?
- **Decision:** open (stage 3)

### C-02  After SAVE, the property edit form shows the values from before the save

- **Rules:** R-12
- **What disagrees:** SAVE reopens the edit form (`property_save_button.dart:38-39, 64-66`), which is built from the header loaded before the save (`property_edit.dart:49`, `changeNotifiers/property_tab.dart:9, 15-27`); nothing reloads it. The success message suggests the form now holds the saved values.
- **Question:** after SAVE on the property edit form, should the form show the values just saved?
- **Decision:** open (stage 3)

### C-03  CANCEL on the property form does not leave the form

- **Rules:** R-13
- **What disagrees:** the button is labelled "CANCEL" (`property_edit.dart:180`), but it only resets the fields and stays (`:184-186`). Leaving is by the back arrow (`:51`).
- **Question:** should CANCEL discard changes and leave the form, or only reset the fields as now?
- **Decision:** open (stage 3)

### C-04  Address lookup is Canada-only while the form allows other countries

- **Rules:** R-15
- **What disagrees:** lookup is restricted to Canada, with a US option commented out (`location_lookup_button.dart:31-34`). The form offers Canada, USA and Other (`property_edit.dart:163`), and money can be in USD.
- **Question:** should address lookup also find US (or any) addresses?
- **Decision:** open (stage 3)

### C-08  Deleting an additional expense asks for no confirmation (unknown why)

- **Rules:** R-48
- **What disagrees:** nothing disagrees outright, and nothing says why. Property delete asks for confirmation (`property_delete_button.dart:52-58`); expense delete removes at once (`expenses_other_details.dart:92-99`).
- **Question:** should deleting an additional expense ask for confirmation?
- **Decision:** open (stage 3)

### C-09  The details screen remembers the last tab across properties (unknown why)

- **Rules:** R-23 (feeds R-26, R-53 / C-17)
- **What disagrees:** nothing says whether this is wanted. The tab is app-wide state set only by tapping a tab (`property_tab.dart:10-11, 29-33`, `property_tabs.dart:30`), so opening another property, or creating one, shows the last tab used.
- **Question:** when a property's details open, should they start on Main, or on the tab last used?
- **Decision:** open (stage 3)

### C-12  Default exchange rate 1.42 (unknown why)

- **Rules:** R-37
- **What disagrees:** nothing says why. Five services fall back to `1.42` (USD→CAD) when the user has set no rate (`expense_service.dart:35, 68`, `metric_service.dart:33`, `overview_service.dart:56`, `borrowing_service.dart:54, 81`), with no date or source for the figure.
- **Question:** when the user has not set an exchange rate, what should be used: a fixed default (which value), a fetched current rate, or a prompt to set one?
- **Decision:** open (stage 3)
