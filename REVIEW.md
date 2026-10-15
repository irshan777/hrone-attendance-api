# REVIEW.md

List every defect you found in the starter `app/main.py` (helpers and endpoints). For each one:

| # | Where (function / line) | What is wrong | How you'd notice it (test, input, or symptom) | How you fixed it |
|---|---|---|---|---|
| 1 | `create_employee` & `punch_in` | Time-Of-Check to Time-Of-Use race condition. It uses `find_one()` then `insert_one()`. | Sending parallel identical requests creates duplicate employees or double punch-ins for the same day. | Created `unique=True` database indexes and used `try/except DuplicateKeyError` to catch conflicts. |
| 2 | `create_employee` & `punch_in` | Uses naive `datetime.now()`. MongoDB assumes UTC, shifting the time by 5.5 hours. | A punch-in at 3:00 AM IST Tuesday logs as 9:30 PM UTC Monday, breaking all date analytics. | Explicitly passed the `IST` timezone to all datetime instantiations: `datetime.now(IST)`. |
| 3 | `punch_in` | Blindly attempts to read `emp["shift_start"]` without verifying the employee exists. | Submitting an unknown `emp_code` results in a `TypeError` and a 500 Internal Server Error crash. | Added an `if not emp:` check to safely raise a `404 Not Found` before any dictionary lookups. |
| 4 | `list_attendance` | Pulls the entire collection into Python memory (`list(find())`) to sort and slice. | Server memory bloats and response times spike severely when testing against 100,000+ records. | Offloaded the sorting and pagination to the database using cursor chaining (`.sort().skip().limit()`). |
| 5 | `list_employees` | Skip math is `page * page_size`. Also, `count_documents({})` ignores the active filter. | Requesting Page 1 skips the first 20 records. Total count ignores department search filters. | Corrected the formula to `(page - 1) * page_size` and passed the active query `q` into `count_documents(q)`. |

**What I looked at and decided was NOT a defect:**
I examined the `compute_late_minutes` helper function and determined it was completely correct. It perfectly implements Business Rule R2 (the 10-minute grace period). By checking `if minutes > 10` and returning the total minutes, it correctly calculates the full floor minutes since `shift_start`, rather than subtracting the grace period from the penalty.