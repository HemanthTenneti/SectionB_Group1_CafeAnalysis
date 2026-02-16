# Data Cleaning Documentation

## The Problems We Found

### Item

- 333 rows have no item listed (blank)
- 292 rows just say "ERROR"
- 344 rows say "UNKNOWN"

### Quantity

- 170 rows show "ERROR"
- 171 rows show "UNKNOWN"
- 138 rows are blank
  The problem is we can't know if someone bought 1 item or 10 items. However, for the 138 blank rows, we might be able to recover the data. If we know the total spent and the price per unit, we can assume quantity. For the ERROR and UNKNOWN rows though, there's nothing to work with.

### Price Per Unit

- 190 "ERROR" entries
- 164 "UNKNOWN" entries
- 179 blank entries
  Again, blank cells might be recoverable if we have quantity and total spent. The garbage data ("ERROR" and "UNKNOWN") can't be fixed.

### Total Spent

- 164 "ERROR" entries
- 165 "UNKNOWN" entries
- 173 blank entries
  The blank ones are potentially fixable by multiplying quantity × price per unit. The rest are stuck.

### Payment Method

- 2,579 missing values (25.79% of the entire dataset!)
- 306 "ERROR" entries
- 293 "UNKNOWN" entries
  The valid payment methods are Digital Wallet (2,291), Credit Card (2,273), and Cash (2,258). That large chunk of missing payment data is a huge problem for any analysis that needs to understand payment patterns.

### Location

- 3,265 missing values (32.65%!)
- 358 "ERROR" entries
- 338 "UNKNOWN" entries
  There are only 2 valid locations: Takeaway (3,022) and In-store (3,017). The missing location data is a critical gap. We could probably generalize it with the current split between Takeaway and In-store but the amount of data missing is an issue.

### Transaction Date

- 159 blank entries
- 142 "ERROR" entries
- 159 rows where the date shows "UNKNOWN"
  Looking at the valid dates, we should expect everything from 2023. Any date outside that range or any row labeled "ERROR" or "UNKNOWN" needs to be removed.

## How to Clean

### Step 1: Remove Records with Unrecoverable Core Data

- Start by deleting rows where the fundamental transaction information is missing or corrupted beyond repair. These create too much uncertainty:
- Delete rows where Item is blank, "ERROR", or "UNKNOWN"
- Delete rows where Payment Method is blank, "ERROR", or "UNKNOWN"
- Delete rows where Location is blank, "ERROR", or "UNKNOWN"
- Delete rows where Transaction Date is blank, "ERROR", "UNKNOWN", or outside of 2023

### Step 2: Recover What We Can

- If Quantity is blank but Total Spent and Price Per Unit exist, calculate: Quantity = Total Spent / Price Per Unit
- If Price Per Unit is blank but Total Spent and Quantity exist, calculate: Price Per Unit = Total Spent / Quantity
- If Total Spent is blank but Quantity and Price Per Unit exist, calculate: Total Spent = Quantity × Price Per Unit

### Step 3: Verify the Math

Now check that every row makes sense mathematically. Quantity × Price Per Unit should equal Total Spent (allowing for tiny rounding differences). Any row where this doesn't work should be flagged for review or deleted.
Make sure all quantities are positive whole numbers, all prices are positive, and all dates are valid.

### Step 4: Final Cleanup

- Convert columns to the right data types (numbers as numbers, dates as dates, text as text)
- Check that payment methods only contain "Digital Wallet", "Credit Card", or "Cash"
- Check that locations only contain "Takeaway" or "In-store"
- Make sure no duplicates exist based on Transaction ID
- Verify the date format is consistent across all rows

## A Note on Data Recovery

When we calculate missing values (like recovering Quantity from Total ÷ Price), we're making an assumption that those numbers are correct. If the Total or Price was wrong to begin with, the recovered Quantity will be wrong too. That's acceptable risk for some analyses, but just we should be aware of it.
