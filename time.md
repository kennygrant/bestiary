# Time in Go

## AddDate

The AddDate function adjusts dates according to rollover rules. Adding years and days is relatively straightforward. If you add 7 years to a date, you'll find it has the same, unless that date doesn't exist in the year, in which case the day before is chosen. For example

2015+

You might get some unexpected results if you add months with it, because the rollover rules for days are used, so subtracting 1 month or adding 1 month may not always mean you end up in the month before or after your initial date. For example:

## Time Zones

## Monotonic time

Since `Go 1.9`, the [time](https://golang.org/pkg/time/) package now transparently tracks monotonic time in each Time value, making computing durations between two Time values a safe operation in the presence of wall clock adjustments.

If Times t and u both contain monotonic clock readings, the operations t.After\(u\), t.Before\(u\), t.Equal\(u\), and t.Sub\(u\) are carried out using the monotonic clock readings alone, ignoring the wall clock readings. If either t or u contains no monotonic clock reading, these operations fall back to using the wall clock readings.

Times may not contain a monotonic clock reading if they pass through functions like AddDate Round or Truncate, or if they come from parsed external sources.

# Time on Go Playground

The playground doesn't support certain operations, and functions which deal with time, 





