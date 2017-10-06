# Writing Servers in Go

When writing servers in go, you'll probably make extensive use of the standard library.

The net and net/http packages are some of the most widely used in the Go ecosystem, and come with some unique pitfalls.

## Handling Responses

Don't close the response body before you check if there was an error.

## Serving Files

A common anti-pattern which has sprouted up to serve files in Go is: 



## Working with JSON

JSON only handles floats, so if you Marshal json you'll have to be aware of this.

## Server Timeouts

Set the timeouts on your server explicitly. 

## Client Timeouts

Set the timeouts on http client. 

