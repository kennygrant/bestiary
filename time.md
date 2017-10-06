# Time in Go

Go includes a good set of primitives for dealing with time in the standard library. There are a few oddities detailed below.

## Time Formatting

Datetime formatting in Go is rather unusual. It uses a format string which functions as an example, unfortunately in practice this is rather difficult to remember. The Parse and Format functions take this format layout:

```
Mon Jan 2 15:04:05 -0700 MST 2006
```

the default time format for dates is and the time package uses the memorable [constant](https://golang.org/src/time/format.go?s=15291:15333#L66) `time.RFC3339` for timestamps in international date format ISO 8601:

```
"2006-01-02T15:04:05Z07:00"
```

There is no format just for these dates unfortunately, so you may want to define some constants for the formats you normally use, to avoid constantly referring to the time package, something like:

```go
const (
   Date     = "2006-01-02"
   DateTime = "2006-01-02 15:04"
)
```

## AddDate

The AddDate function adjusts dates according to rollover rules. Adding years and days is relatively straightforward. If you add 7 years to a date, you'll find it has the same, unless that date doesn't exist in the year, in which case the day before is chosen. For example

2015+

You might get some unexpected results if you add months with it, because the rollover rules for days are used, so subtracting 1 month or adding 1 month may not always mean you end up in the month before or after your initial date. For example:

## Time Zones

You should **strongly** prefer to store times as **UTC**, and convert them for display. This makes comparisons and calculations straightforward, and allows you to customise display for the user's current location. When creating times or comparing with the current time, always use the t.UTC\(\) function to be sure you compare the UTC time.

If you need to convert times, you can convert times from a given zone to a location easily enough:

```go
// Load a set location 
l, err := time.LoadLocation("America/Mexico_City")

// Set the time zone
now := t.In(l)
```

## Monotonic time

Since `Go 1.9`, the [time](https://golang.org/pkg/time/) package now transparently tracks monotonic time in each Time value, making computing durations between two Time values a safe operation in the presence of wall clock adjustments.

If Times t and u both contain monotonic clock readings, the operations t.After\(u\), t.Before\(u\), t.Equal\(u\), and t.Sub\(u\) are carried out using the monotonic clock readings alone, ignoring the wall clock readings. If either t or u contains no monotonic clock reading, these operations fall back to using the wall clock readings.

Times may not contain a monotonic clock reading if they pass through functions like AddDate Round or Truncate, or if they come from parsed external sources.

# Time on Go Playground

The playground starts with the same time for every run, so output is deterministic, so be aware examples including time may not function as you expect on the playground. For more details see [Inside the Go Playground.](https://blog.golang.org/playground)

