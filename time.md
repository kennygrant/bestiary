# Time in Go

Go includes a good set of primitives for dealing with time in the standard library. There are a few oddities detailed below.

# Time on Go Playground

The playground starts with the same time for every run to make sure output is deterministic, so examples including time may not function as you expect on the playground. For more details see [Inside the Go Playground.](https://blog.golang.org/playground). 

## Time Formatting

Datetime formatting in Go is rather unusual. It uses a format string which functions as an example, unfortunately in practice this is rather difficult to remember. The Parse and Format functions take this format layout:

```
Mon Jan 2 15:04:05 -0700 MST 2006
```

the default time format for dates is and the time package uses the memorable [constant](https://golang.org/src/time/format.go?s=15291:15333#L66) `time.RFC3339` for timestamps in international date format ISO 8601:

```
"2006-01-02T15:04:05Z07:00"
```

You may want to define some constants for any date formats you normally use, to avoid constantly referring to the time package, for example:

```go
const (
   Date     = "2006-01-02"
   DateTime = "2006-01-02 15:04"
)
```

## AddDate

The AddDate function adjusts dates according to arbitrary overflow rules to ensure that results are reversible. If adding Years or days, the results will generally be as expected, but if adding months, the results may not be as users expect and does not match the output of popular programs like excel. 

For example subtracting one month from October 31 yields October 1st

```go
// Oct 31st minus 1 month is Oct 1st, 
// not Sept 30th according to Go
input := time.Date(2016, 10, 31, 0, 0, 0, 0, time.UTC)
output := time.Date(2016, 9, 30, 0, 0, 0, 0, time.UTC)
result := input.AddDate(0, -1, 0)
if result != output {
    fmt.Printf("got:%v want:%v\n", result, output)
}
```

The results will be unpredictable and depend on the day chosen and the number of days in the target month. If the number of days of the start month and the end month do not match, results can be unexpected. This is important if your users expect to add 9 months and 1 day to a given date to conform with accounting rules for example - typically this means 9 calendar months without rollover and then 1 day to be added. There is no way round this except writing your own code to handle adding months in a more predictable way. 

## Time Zones

You should **strongly** prefer to store times as **UTC**, and convert them for display. This makes comparisons and calculations straightforward, and allows you to customise display for the user's current location at the time of viewing. When creating times or comparing with the current time, always use the t.UTC\(\) function to be sure you compare the UTC time.

If you need to convert times, you can convert times from a given zone to a location easily enough:

```go
// Load a set location 
l, err := time.LoadLocation("America/Mexico_City")

// Set the time zone
now := t.In(l)
```

## Monotonic time

Since `Go 1.9`, the [time](https://golang.org/pkg/time/) package now transparently tracks monotonic time in each Time value, making computing durations between two Time values a safe operation in the presence of wall clock adjustments.

> If Times t and u both contain monotonic clock readings, the operations t.After\(u\), t.Before\(u\), t.Equal\(u\), and t.Sub\(u\) are carried out using the monotonic clock readings alone, ignoring the wall clock readings. If either t or u contains no monotonic clock reading, these operations fall back to using the wall clock readings.

Times may not contain a monotonic clock reading if they pass through functions like AddDate Round or Truncate, or if they come from parsed external sources, so don't assume they always will have.

## Comparing time

When comparing time you should usually use the [Time.Equal](https://golang.org/pkg/time/#Time.Equal) method rather than ==, as the == operator also compares both Location, which sets the time zone offset but not the absolute time, and the monotonic clock reading, which can be stripped in some circumstances. 

> In general, prefer t.Equal(u) to t == u, since t.Equal uses the most accurate comparison available and correctly handles the case when only one of its arguments has a monotonic clock reading.

## Formats vs Values 

Despite their beguiling appearance, time formats are not time values, so if testing parsing, be aware that using a format as a value is often invalid due to formatting directives in the format string which are not valid in a time:

```go
// Time formats are not always valid values
_, err := time.Parse(time.RFC3339, time.RFC3339)
fmt.Println("error", err) // RFC3339 is not a valid time
```

> error parsing time "2006-01-02T15:04:05Z07:00": extra text: 07:00
