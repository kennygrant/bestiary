# Writing Servers in Go

When writing servers in go, you'll probably make extensive use of the standard library.The net and net/http packages are some of the most widely used in the Go ecosystem, and come with some unique pitfalls. The source to the [net/http package](https://golang.org/pkg/net/http/) is surprisingly straightforward and readable, and if you're going to write servers, you should read through the docs and the source of this package at least once to see what is available.


## Getting Started

Running a simple server is drop-dead simple in Go, and to get started all you need is a handlerFunc as the http package contains a default router and server you can use. A Handler in go is a function which responds to http requests, usually for a specific route on your server (thought they can be general, like an error handler or a static file handler). It is a very simple function with two arguments: the Request, and an http.ResponseWriter to write a response to. Except for reading the body, handlers should not modify the provided Request.

```go
// A handler function in go writes a response to the Request to the ResponseWriter
func HelloServer(w http.ResponseWriter, req *http.Request) {
	io.WriteString(w, "hello, world!\n")
}

func main() {
    // Attach our handler function to the default mux 
    http.HandleFunc("/hello", HelloServer)

    // Ask the http package to start a server at port 8080
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		log.Fatal(err)
	}
}
```

## Handlers 

When writing handlers, be aware that the http.Request.Body is an io.ReadCloser, which means it doesn't have to be read into memory all at once, but can be read as a stream. Consider using the io.Reader interface if you need to pass it to a decoder or receive large requests. The request Body will always be non-nil, and the server will close it, so handlers do not need to.

Similarly, the http.ResponseWriter is an io.Writer, so it can be written to in a stream, and io.Copy can be used to read from one source and write it to the output (for example when reading a file) without reading everything into a buffer first.

Try to use these interfaces if possible when reading from requests and writing responses. 

## Servers

You normally should not use the http.DefaultServeMux, in case other packages have decided to register handlers on it, and you expose those handlers without knowing about it. For example the pprof package by default will install handlers simply by being imported. 

Also set the timeouts on your server explicitly to sensible defaults as the default timeouts are not advisable, but cannot be changed because of the Go 1 promise. You can read more about these timeouts in[ the complete guide to net/http timeouts](https://blog.cloudflare.com/the-complete-guide-to-golang-net-http-timeouts/) and [So you want to expose go on the internet](https://blog.cloudflare.com/exposing-go-on-the-internet/) by @filosottile at cloudflare. If you're interested in networking, the cloudflare blog has a lot of good articles on Go.

```
// Setting up your own server, specifying some default timeouts
srv := &http.Server{  
    ReadTimeout:  5 * time.Second,
    WriteTimeout: 10 * time.Second,
    IdleTimeout:  120 * time.Second,
    TLSConfig:    tlsConfig, // set TLS config 
    Handler:      serveMux, // always set this to avoid using http.DefaultServeMux
}
log.Println(srv.ListenAndServeTLS("", ""))
```

### Implicit goroutines

The http server uses goroutines to run your handlers and serve multiple requests in parallel, so **each handler is in a new goroutine**. This means you have to be careful about sharing memory between handlers. If for example you have a global config struct or cache, this must be protected by a mutex. Always run your server programs with a race tester to catch bugs like this as they can creep in if you are not vigilant about finding them. 

### Listen and Serve Blocks

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

If your go server is behind a proxy like nginx or a load balancer those will also intercept such requests. 

### Panics in goroutines

If a handler panics, the server assumes that the panic was isolated to the current request, recovers, logs a stack trace to the server log, and closes the connection. So the server will recover from any panics in your handlers, but if your handlers use the go keyword, they must protect against panics **within any separate goroutines** they create, otherwise those goroutines can crash the entire server with a panic. See the errors chapter for more details.

One approach to this problem is never to make mistakes. However with unpredictable, malformed or downright malicious data coming from outside the application in parameters or files this can be difficult, so it may be worth protecting against panics in any goroutines launched from your handlers.

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

### Cookies

Cookies are just headers passed between client and server, so they're not complex, but go provides some utilities for manipulating them. The server sends cookies with SetCookie on the http.ResponseWriter (not on the request). NB that invalid cookies may be silently dropped when set or received. If you want to use cookies with invalid values including strings like space or @, you will need to use your own cookie implementation. 

```go
// handler sets a cookie on the request
func handler(w http.ResponseWriter, r *http.Request) {
	cookie := http.Cookie{
		Name:     "session_name",              // Set the name
		Value:    "cookie_value",              // Set a value on the cookie
		Domain:   "example.com",               // Always set the domain
		Path:     "/",                         // Always set the path
		Expires:  time.Now().AddDate(1, 0, 0), // Optional expiry time
		MaxAge:   0,                           // MaxAge=0 means no Max-Age
		HttpOnly: true,                        // Allow only the server to access the cookie
		Secure:   true,                        // Set this if using https
	}
	// Set the cookie on the response
	http.SetCookie(w, &cookie)
	// Write the rest of the response
	w.WriteHeader(http.StatusOK)
	io.WriteString(w, "hello")
}
```

If you want to store information in cookies you should try to keep it limited, and encrypt the information stored, as it is stored on the client machine outside your control. The [gorilla/sessions](http://www.gorillatoolkit.org/pkg/sessions) package allows you to use encrypted cookies. 


## Making http requests

Your program may need to fetch http resources, and the net/http package offers http.Get or http.Client in order to help with this, but there are some issues you should be aware of.

### Client Timeouts

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

### Accepting uploads

If accepting uploads via a multipart form, be aware that temp files created will not be automatically deleted by Go. You should therefore delete them with [Request.Form.RemoveAll](https://golang.org/pkg/mime/multipart/#Form.RemoveAll) after they have been used by the server.


### Check Status Codes

Always check the status code of the response when making a request with the Go http client. If the status is in the 200 range you can use the response as is. If it is in the 300 range it is a redirection. If it is in the 400 range you have a problem with your request which you should fix \(e.g. invalid headers, invalid URL\). If it is in the 500 range there was a problem on the server end, so you need to inform the user.

The error returned by http.Get and friends informs you if there was an error reading the response, not if the response was successful.

```go
url := "https://example.com"
resp, err := http.Get(url)
if err != nil {
    return fmt.Errorf("error getting %s: %v",url,err)
}

// Unexpected response, inform the user
if resp.StatusCode < 200 || resp.StatusCode > 299 {
    return fmt.Errorf("unexpected http status:%d",resp.StatusCode)
}

// No errors fetching, so the response is ok to use
```

## Utilities

Under the net/http package are several utility packages which make life easier when debugging or testing http Requests. 

In particular, the [httputil](https://golang.org/pkg/net/http/httputil) package provides:

#### [httputil.DumpRequest](https://golang.org/pkg/net/http/httputil/#DumpRequest)

DumpRequest outputs a nicely formatted version of the request passed to it, making the request content much clearer. 

#### [httputil.DumpResponse](https://golang.org/pkg/net/http/httputil/#DumpResponse)

DumpResponse outputs a nicely formatted version of the response passed to it, making the response content much clearer. 

#### [httputil.ReverseProxy](https://golang.org/pkg/net/http/httputil/#ReverseProxy) 

Provides a simple reverse proxy handler

And the [httptest](https://golang.org/pkg/net/http/httptest) package provides several useful functions and types:

#### [httptest.NewRequest](https://golang.org/pkg/net/http/httptest/#NewRequest)

Provides a new request suitable for sending to a Handler for testing. 


#### [httptest.ResponseRecorder](https://golang.org/pkg/net/http/httptest/#ResponseRecorder)

ResponseRecorder is a ResponseWriter which records the response data written by a handler so that it can be compared in tests to the expected response. 


## Profiling

The [http/pprof](https://golang.org/pkg/net/http/pprof/) package provides excellent support for profiling your server, and can be used to generate data for use in flame graphs or other graphical representations of your program.

To use it, simply import the package:

```go
import _ "net/http/pprof"
```

**Beware **though if you register it there are hidden side effects. It will attach endpoints to the default serve mux during init. For this and performance reasons you should not import it in production. You can find out more about profiling in [Profiling Go Programs](https://blog.golang.org/profiling-go-programs).

## Latency

When optimising servers it's worth being aware of the comparitive times for operations. If your database is in a different data centre for example on another continent, it doesn't matter how optimised your code is if it needs to spend 150ms for every database request. 

1 CPU cycle takes 0.0000003ms, Cache access 0.0000129ms, Solid state disk 0.05ms, Rotational disk access around 1ms and on internet request around 200ms across continents. So make sure you're optimising the right operations - avoiding network access will always be faster than any other optimisation.

