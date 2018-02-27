---
title: "Golang Configure Cors Log"
date: 2018-02-27T16:45:26-05:00
draft: false
---

# Intro 
I recently ran into some CORS issues while creating a REST API in Go. Because I spend the bulk of my time programming in JavaScript, I had at first assumed there would be an easy catch all CORS package that I could configure and never think about again. Then I remembered that part of Go's beauty is it's standard library, so I decided to solve the problem by myself, and while doing so learned a lot about HTTP and CORS.

# Background

For those who don't know CORS stands for Cross Origin Resource Sharing. It's essentially a mechanism for restricting access to a resource from a server of a different origin. Sounds straightforward right? In reality it's pretty complex, and I'd recommend reading more about it on [Mozilla's site](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) before reading the takeways.

The key takeaways of CORS for an API developer are as follows:

* CORS Pre-Flight is triggered on any requests with a *content-type* or *authorization* header
* If you run into CORS Pre-Flight, an OPTIONS request will be sent to your API .

This means that using a Gorilla mux router in Golang, unless you specifically handle the OPTIONS request, you will get an error code. Most likely `405`.

# How to handle the CORS requests in Golang

The following function sets the neccessary headers in the response to enable CORS. The comments explain each header.

```

func setHeaders(h http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        //anyone can make a CORS request (not recommended in production)
		w.Header().Set("Access-Control-Allow-Origin", "*")
        //only allow GET, POST, and OPTIONS
		w.Header().Set("Access-Control-Allow-Methods", "GET, POST, OPTIONS")
        //Since I was building a REST API that returned JSON, I set the content type to JSON here.
		w.Header().Set("Content-Type", "application/json")
        //Allow requests to have the following headers
		w.Header().Set("Access-Control-Allow-Headers", "Origin, Content-Type, Authorization, cache-control")
        //if it's just an OPTIONS request, nothing other than the headers in the response is needed.
        //This is essential because you don't need to handle the OPTIONS requests in your handlers now
		if r.Method == "OPTIONS" {
			return
		}

		h.ServeHTTP(w, r)
	})
}
```

With Gorilla mux router, you can use this for all requests by doing the following:

`	http.ListenAndServe(":3333", setHeaders(myMuxRouter))`



Hope you enjoyed and are now able to handle that pesky CORS mechanism in Golang!


