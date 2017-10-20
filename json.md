# Handling JSON

The Go Standard Library offers templating of both text and html content. The html templates offer a superset of the text template features, including contextual escaping, but the interface can be a little confusing.  

## JSON doesn't have integers

All numbers in json are floats. This can cause some weird side effects if you're not aware of it, as the precision of the floats is limited, and floats do not behave the same as integers in all cases.


## Decoding arbitrary JSON 

The json package uses map[string]interface{} and []interface{} values to store arbitrary JSON objects and arrays; it will happily unmarshal any valid JSON blob into a plain interface{} value. The default types unmarshalled are bool, string and nil for their respective JSON counterparts, and float64 for JSON numbers (which are always floats). 

Full example here of decoding 

