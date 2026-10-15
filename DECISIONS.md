# Design decisions

## Database indexes

I added a unique index for `employees.emp_code` and a unique compound index for `attendance_logs.emp_code` and `date`. These fields identify an employee and that employee's attendance record for a day. The database indexes also help prevent duplicate records when requests arrive at the same time. Other indexes support employee filtering, attendance history, and late-arrival reports. The explain endpoint returns MongoDB execution statistics so the query plan can be inspected.

## Two punch-in requests at once

Checking for an existing record before inserting is not enough, because two requests could pass the check together. The unique compound index is the final safeguard. One request can create the record; the other gets a conflict response (`409`).

## Ranking employees with the same late minutes

The late-arrival leaderboard uses competition ranking. For example, if two employees share second place, the next employee is ranked fourth. The results are sorted by late minutes, with employee code used to keep the output order consistent. The limit is applied to the ranks, so employees tied at the cutoff are not arbitrarily left out.

## Department summary

The monthly department summary starts with employees who had joined by the end of the selected month and then looks for their attendance records. This means an employee with no attendance record is still included in the headcount. Employees who joined after the month are excluded. Average work hours are calculated from the attendance records that qualify for the report.

## If the data grows significantly

For a dataset around 100 times larger, I would first inspect the explain output and measure the queries that take the most time. I would then adjust indexes based on the actual filters being used. Depending on the workload, I would also consider archiving older records or keeping precomputed daily summaries for expensive reports. Any cached or precomputed values would need checks to keep them in sync with attendance updates.
