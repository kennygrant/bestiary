# Writing Servers in Go

When writing servers in go, you'll probably make extensive use of the standard library.The net and net/http packages are some of the most widely used in the Go ecosystem, and come with some unique pitfalls.

## Goroutines

The http server uses goroutines to run your handlers, so **each handler is in a new goroutine**. This means you have to be careful about sharing memory between handlers. If for example you have a global config struct or cache, this must be protected by a mutex.

## Handling Responses

#### Closing the response body

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

## 

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

The built in Go server will reject bad requests before they hit your handlers, so you will never see them. It will return a code 400 Bad Request to the client. There is at present no way to override this. Normally this isn't a problem but it's something to be aware of. For example this bad request would never hit your handlers:

```
curl http://localhost:3000/%3
```

and would simply return to the client:

> 400 Bad Request

## ListenAndServe

After spawning a server with http.ListenAndServe convenience function, don't expect control to return to your main function until the end of the program.

```go
func main() {
    http.HandleFunc("/hello", HelloServer)
    err := http.ListenAndServe(":12345", nil)
    if err != nil {
       log.Fatal(err)
    }
    log.Println("this line will never execute")    
}
```

## Panics in goroutines

If a handler panics, the server assumes that the panic was isolated to the current request, recovers, logs a stack trace to the server log, and closes the connection. So the server will recover from any panics in your handlers, but if your handlers use the go keyword, they must protect against panics** within any separate goroutines** they create, otherwise those goroutines can crash the entire server with a panic. 

## Cryptography

If you're trying to work with cryptography in go, definitely view this talk from [George Tankersley](https://www.golangnews.com/stories/1469-video-crypto-for-go-developers-gophercon-crypto) at CoreOS on Crypto for Go developers, which comes with [cryptopasta](https://github.com/gtank/cryptopasta) example code.

### Random Numbers

If generating random numbers for a server, you probably want them to be unpredictable, so use crypto/rand, not math/rand.

```go
// Example by George Tankersley
// NewEncryptionKey generates a random 256-bit key for Encrypt() and
// Decrypt(). It panics if the source of randomness fails.
func NewEncryptionKey() *[32]byte {
    key := [32]byte{}
    _, err := io.ReadFull(rand.Reader, key[:])
    if err != nil {
        panic(err)
    }
    return &key
}
```

### Comparing Passwords

If you're comparing passwords or other sensitive data, to avoid timing attacks, make use of the [crypto/subtle](https://golang.org/pkg/crypto/subtle/) subtle.ConstantTimeCompare, or better still use the [bcrypt](https://godoc.org/golang.org/x/crypto/bcrypt) package library functions bcrypt.CompareHashAndPassword.

## 



