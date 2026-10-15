# Code review notes

The implementation addresses the following issues identified while reviewing the starter API against the assignment requirements.

| Area | Issue | Change made |
|---|---|---|
| Health check | The endpoint could report the service as healthy even when MongoDB was unavailable. | It pings MongoDB and returns a service-unavailable response if the check fails. |
| Employee creation | Concurrent requests could create duplicate employee codes. | A unique database index protects the code, and duplicate-key errors return `409`. |
| Employee validation | Some fields and dates were not validated consistently. | Request models validate field constraints and calendar dates. |
| Employee listing | Pagination, department counts, and result ordering could be incorrect. | The query uses the requested filters, a corrected page offset, a matching count, and a stable sort. |
| Punch-in | Unknown employees and invalid input could cause poor error responses; timestamps also needed consistent timezone handling. | The endpoint checks the employee, validates the input, and handles UTC/IST conversions explicitly. |
| Concurrent punch-ins | A separate duplicate check could allow two requests to create the same day's record. | The unique `(emp_code, date)` index prevents duplicate attendance records. |
| Attendance date | Overnight shifts need special handling around midnight. | Attendance dates are resolved using IST and the shift-end rule. |
| Late minutes | The grace period and rounding of late minutes need to follow the stated rules. | Late minutes are calculated using the 600-second threshold and whole-minute flooring. |
| Work hours | Python's default rounding can round halfway values differently from the expected rule. | Work hours use decimal `ROUND_HALF_UP` to two decimal places. |
| Overtime | Overnight shifts and the minimum overtime duration need to be accounted for. | The calculation resolves the shift end date and applies the 30-minute minimum. |
| Attendance listing | Loading all matching records into Python is inefficient for paginated requests. | Filtering, sorting, counting, and pagination are performed in MongoDB. |
| Punch-out | Two simultaneous requests could both appear to punch out successfully. | A conditional atomic update closes an open record; a repeated request returns `409`. |
| Regularization | Updates need validation and an audit trail. | The updated record is validated, derived fields are recalculated, and a history entry is added with concurrency protection. |
| Analytics | Reports need to apply the attendance rules consistently. | MongoDB aggregation pipelines calculate the requested summaries using the stored attendance values. |
| Response formatting | MongoDB-specific values and older records with missing fields should not leak into API responses. | Response serializers normalize IDs, dates, and default values. |
| Indexes and explain | Query performance needed to be inspectable. | Indexes are created at startup, and the explain endpoint returns execution statistics. |

A few data-handling details are worth noting: `(emp_code, date)` is the natural key for a daily attendance record; older records without `history`, `half_day`, `late_minutes`, or `overtime_minutes` are handled with sensible defaults; and `ABSENT` and `LEAVE` are not treated as attendance presence statuses.

This document describes the implementation choices. Run the API against MongoDB and exercise the endpoints with the assessment data before relying on it in production.
