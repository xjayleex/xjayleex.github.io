---
layout: post
comment: false
title: HTTP/2 in Go
date: 2020-08-14
category: "back-end"
tags: [go http2]
update: 2020-09-10
author: Jaehyun Lee
---

> ### Contents
[**Introduction**](#introduction)  

#### HTTP/2 Server
---
- [*'http2 doc'*](https://godoc.org/golang.org/x/net/http2)

```go
package main

import (
	"log"
	"net/http"
)
func main() {
	srv := &http.Server{Addr: ":8000", Handler: http.HandlerFunc(handle)}

	// Start the server with TLS, since we are running
	// HTTP/2 it must be run with TLS.
	// Exactly how you would run an HTTP/1.1 server with TLS connection.
	log.Printf("Serving on https://0.0.0.0:8000")
	log.Fatal(srv.ListenAndServeTLS("/path-to/server.crt",
		"/path-to/server.key"))
	//log.Fatal(srv.ListenAndServe())
}

func handle(w http.ResponseWriter, r *http.Request) {
	// Log the request protocol
	log.Printf("Got connection: %s", r.Proto)
	// Send a message back to the client
	w.Write([]byte("Hello"))
}
```


