# Writing Servers in Go

When writing servers in go, you'll probably make extensive use of the standard library.The net and net/http packages are some of the most widely used in the Go ecosystem, and come with some unique pitfalls. The source to the [net/http package](https://golang.org/pkg/net/http/) is surprisingly straightforward and readable, and if you're going to write servers, you should read through the docs and the source of this package at least once to see what is available.

## Running a server

You should not use the http.DefaultServeMux, in case other packages have decided to register handlers on it, and you expose those handlers without knowing about it.

Set the timeouts on your server explicitly to sensible defaults. You can read more about these timeouts in[ the complete guide to net/http timeouts](https://blog.cloudflare.com/the-complete-guide-to-golang-net-http-timeouts/) and [So you want to expose go on the internet](https://blog.cloudflare.com/exposing-go-on-the-internet/) by @filosottile at cloudflare. If you're interested in networking, the cloudflare blog has a lot of good articles on Go.

```
srv := &http.Server{  
    ReadTimeout:  5 * time.Second,
    WriteTimeout: 10 * time.Second,
    IdleTimeout:  120 * time.Second,
    TLSConfig:    tlsConfig, // set TLS config 
    Handler:      serveMux, // always set this to avoid using http.DefaultServeMux
}
log.Println(srv.ListenAndServeTLS("", ""))
```

## Implicit goroutines

The http server uses goroutines to run your handlers and serve multiple requests in parallel, so **each handler is in a new goroutine**. This means you have to be careful about sharing memory between handlers. If for example you have a global config struct or cache, this must be protected by a mutex. 

## Client Timeouts

The http client has no default timeout, which can be a problem. In this example problem the client waits 1 hour before exiting.

```go
package main
import (
  “fmt”
  “net/http”
  “net/http/httptest”
  “time”
)
func main() {
  svr := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    // handler sleeps for 10 minutes
    time.Sleep(10 * time.Minute)
  }))
  defer svr.Close()

  // Make a get request with default client
  fmt.Println(“making request”)
  http.Get(svr.URL)
  fmt.Println(“finished request”)
}
```

Instead, you should create a client explicitly with a timeout:

```go
// Create a client with a timeout of 10 seconds
var netClient = &http.Client{
  Timeout: time.Second * 10,
}
response, _ := netClient.Get(url)
```

## Handling Responses

### Closing the response body

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

### Serving Files

Don't perform **any** file operations on paths before you clean them, and root them at a known good path. If you're serving an entire directory of files, consider using http.FileServer instead of http.ServeFile:

```go
fileServer := http.FileServer(http.Dir("./public/static"))
http.Handle("/static/", http.StripPrefix("/static", fileServer))
```

If you are using os.Stat, ioutil.ReadAll, or [http.ServeFile](https://www.gitbook.com/book/kennygrant/go-bestiary/edit#) or similar functions with user input \(be that in the request url or params\), be sure to sanitise the file path first, and root it at a known public path. The default mux will usually strip .. from urls before presenting to handlers and ServeFile has some protections against directory traversal, but it is better to be very careful when accessing local files based on anything from user input.

If you wish to serve static content but present a custom 404 page to users, set up a file handler which checks if files exist and returns 404 or 401 in case of problems accessing the file, but otherwise calls ServeFile.

```go
// Clean the path and prefix with our public dir
localPath := "./public" + path.Clean(r.URL.Path)

// Check the file exists
s, err := os.Stat(localPath)
if err != nil {
    // If file not found return 404 page
    if os.IsNotExist(err) {
        renderNotFound()
        return
    }

    // For other file errors render unauthorised and return
    http.Error(w, "Not Authorized", http.StatusUnauthorized)
    return
}

// If not a file return 404 page
if s.IsDir() {
    renderNotFound()
    return
}

// Serve the file content
http.ServeFile(w, r, localPath)
```

### Bad Requests

The built in Go server will reject bad requests before they hit your handlers, so you will never see them. It will return a code 400 Bad Request to the client. There is at present no way to override this. Normally this isn't a problem but it's something to be aware of. For example this bad request would never hit your handlers:

```
curl http://localhost:3000/%3
```

and would simply return to the client:

> 400 Bad Request

## ListenAndServe

If you launch the http server in your main goroutine, don't expect control to return to your main function until the end of the program, because it blocks waiting for input until an error occurs.

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

If a handler panics, the server assumes that the panic was isolated to the current request, recovers, logs a stack trace to the server log, and closes the connection. So the server will recover from any panics in your handlers, but if your handlers use the go keyword, they must protect against panics** within any separate goroutines** they create, otherwise those goroutines can crash the entire server with a panic. See the errors chapter for more details.

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

## Profiling

The [http/pprof](https://golang.org/pkg/net/http/pprof/) package provides excellent support for profiling your server, and can be used to generate data for use in flame graphs or other graphical representations of your program.

To use it, simply import the package:

```
import _ "net/http/pprof"
```

**Beware **though if you register it there are hidden side effects. It will attach endpoints to the default serve mux during init. For this and performances reasons you should not import it in production. You can find out more about profiling in [Profiling Go Programs](https://blog.golang.org/profiling-go-programs).

