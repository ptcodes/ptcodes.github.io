---
layout: post
title:  Go vs Crystal Performance
categories: [Go,Crystal,Performance,wrk]
image: /images/social/crystal-lang.png
---

![Go vs Crystal](/images/go-vs-crystal.png)

It's a follow up post to the previous [Ruby vs Crystal Performance](/ruby-vs-crystal-performance/).

I guess this time it will be a fair performance comparison as both languages are compiled and statically typed.


We will perform a couple of tests:
* Finding a number in the Fibonacci sequence as in the previous post
* Running an HTTP server locally and performing benchmarks with wrk

Language versions installed my machine are:
* go version go1.14.3 darwin/amd64
* Crystal 0.34.0 (2020-04-07)
 
I'm curious to find out how Go and Crystal perform in comparison to each other.

<!-- more -->

### Compilation

For the tests we will be running previously compiled programs. We will use the release flag to enable optimizations in Crystal:
```bash
crystal build --release program.cr
```

Go binaries don't have a release version and we won't be using any flags. So, it's just:
```bash
go build program.go
```

## Fibonacci

Alright, first we will write code to generate a Fibonacci sequence for a given number. Let's find the 47th number which is 2,971,215,073.

Go version:

```go
package main

import "fmt"

func fibonacci(n uint32) uint32 {
  if n < 2 {
    return n
  }
  return fibonacci(n-1) + fibonacci(n-2)
}

func main() {
  fmt.Println(fibonacci(47))
}
```

Crystal version:

```crystal
def fibonacci(n : UInt32)
  return n if n < 2
  fibonacci(n-1) + fibonacci(n-2)
  end

puts fibonacci(47)
```

Results on my machine (MacBook Pro 2.2 GHz Intel Core i7):

| **Language** | **Binary size** | **Run time** | **Memory usage**
| go      | 418k | 21.28s | 2.01M
| Crystal | 2.1M | 19.69s | 1.72M

Crystal is slightly winning here. 

A few observations here: 

Crystal's binary size is 5 times smaller than Go's. Though, they can be slightly reduced in size when we omit the debug information:
```bash
go build -ldflags="-w" fibonacci_golang.go
```
This way the binary size goes down from 2.1M to 1.7M.

Also, not in this particular example, but generally Go's compilation time is much much faster than Crystal's.

## HTTP Server

Now, let's create a simple HTTP server using standard libraries. Both Go's _net/http_ and Crystal's _http/server_ employ concurrency: Go uses goroutins and Crystal uses fibers.

Go version:
```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	http.HandleFunc("/", HelloServer)
	http.ListenAndServe(":8080", nil)
}

func HelloServer(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello from %s!", r.URL.Path[1:])
}
```

Crystal version:
```crystal
require "http/server"

server = HTTP::Server.new do |context|
  context.response.content_type = "text/plain"
  context.response.print "Hello from #{context.request.path}!"
end

puts "Listening on http://127.0.0.1:8080"
server.listen(8080)
```

For benchmarking we will be using [wrk](https://github.com/wg/wrk). If you're not familiar with this tool it's like a pretty well known ApacheBench (ab) but a modern version.

Here is how we can run a benchmark for 60 seconds, using 8 threads, and keeping 400 HTTP connections open:

```bash
wrk -t8 -c400 -d60s http://localhost:8080/hello
```

Results for the Go server:
```
Running 1m test @ http://localhost:8080/hello
  8 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     3.56ms    2.26ms  95.31ms   92.00%
    Req/Sec     8.77k     2.24k   15.75k    64.66%
  4190457 requests in 1.00m, 535.51MB read
  Socket errors: connect 157, read 100, write 0, timeout 0
Requests/sec:  69757.81
Transfer/sec:      8.91MB
```

Results for the Crystal server:
```
Running 1m test @ http://localhost:8080/hello
  8 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     2.89ms    0.97ms  19.01ms   80.34%
    Req/Sec    10.54k     3.41k   18.14k    60.85%
  5035284 requests in 1.00m, 513.82MB read
  Socket errors: connect 157, read 85, write 0, timeout 0
Requests/sec:  83917.26
Transfer/sec:      8.56MB
```

Results:

| **Language** | **Binary size** | **Memory usage** | **CPU usage** | **Throughput**
| go | 7.4M | 20.2M | 300% | 69,757
| Crystal | 966kb | 19.1M | 99% | 83,917

Crystal again shows better results.

CPU utilization over 100% in the table might seem confusing. But it simply means the system uses multiple cores. One core at max is 100%. 

My machine has 8 cores as it can be seen with the following command on macOs:
```bash
sysctl -n hw.ncpu
```

## Conclusion

Frankly speaking, we have only performed a couple of small tests to make any conclusions but I'm still excited for Crystal as a young language but showing great results. 


