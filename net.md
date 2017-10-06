# Writing Servers in Go

When writing servers in go, you'll probably make extensive use of the standard library.

The net and net/http packages are some of the most widely used in the Go ecosystem, and come with some unique pitfalls.

## Handling Responses

Don't close the response body before you check if there was an error.

```go
 r, err := http.Get("https://example.com")
 // Don't defer close here
 if err != nil {
   return err
 }
 // It's safe to defer close once you know there was no error
 defer r.Body.Close()
```

## Serving Files

If using [http.ServeFile](https://golang.org/pkg/net/http/#ServeFile) to serve files, make sure you sanitise the path first and check the file exists, then serve:

```go
p := filepath.Join("./public/static/", filepath.Clean(r.URL.Path[1:]))
http.ServeFile(w, r, p)
```

don't perform file operations on paths before you clean them, and root them at a known good path. If you're serving an entire directory of files, consider using http.FileServer instead of http.ServeFile:

```go
fileServer := http.FileServer(http.Dir("./public/static"))
http.Handle("/static/", http.StripPrefix("/static", fileServer))
```

## Working with JSON

JSON only handles floats, so if you Marshal json you'll have to be aware of this.

#### Private fields in JSON

Private fields will not be marshalled or unmarshalled, so make fields public if you want them to show up in JSON. 

#### Hiding fields from export

You can use struct tags for this

## Server Timeouts

Set the timeouts on your server explicitly.

## Client Timeouts

Set the timeouts on http client.

## Bad Requests

The built in Go server will reject bad requests before they hit your handlers, so you will never see them. It will return 400 Bad Request to the client. There is at present no way to override this. 

## Errors

The server will trap any panics in your goroutines. Will it catch panics within subgoroutines?

