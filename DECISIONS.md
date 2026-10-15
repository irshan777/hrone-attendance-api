1. **Indexes.** I added unique indexes on `emp_code` and `(emp_code, date)` to avoid duplicates. For faster aggregations, I added indexes on `department`, `(department, joined_on)`, `(date, late_minutes)`. I declined to add text indexes on employee names, since the API only searches by code.

2. **Punch-in race.** Both requests will pass the Python layer, because they only perform a unique database insert. The first one will succeed, because it will finish the insert and return 201. The second will throw a `DuplicateKeyError` at the database layer, which will be caught by my `except` handler to return a `409 Conflict`.

3. **Ties.** The query will return both employees, even if it exceeds the `limit`. Using Standard Competition Ranking (1, 2, 2, 4), the `limit` applies to the number of ranks, not rows. This avoids the scenario where someone who tied for 10th would be pushed to 11th due to the limit.

4. **Headcount.** The aggregations always originate from the `employees` set (truth source). I'm using `$lookup` to join logs, then `$unwind` with `preserveNullAndEmptyArrays: True` to keep employees who have no logs.

5. **One thing you would change:** I would introduce a cache layer (Redis) for `department_summary` and `trend`. Since historical data is static, it's a waste of CPU to run `$densify` calendars on the fly for 100x data sets; I'd offload that to materialized views.