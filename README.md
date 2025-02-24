# Timeout

[![Run Tests](https://github.com/gin-contrib/timeout/actions/workflows/go.yml/badge.svg?branch=master)](https://github.com/gin-contrib/timeout/actions/workflows/go.yml)
[![codecov](https://codecov.io/gh/gin-contrib/timeout/branch/master/graph/badge.svg)](https://codecov.io/gh/gin-contrib/timeout)
[![Go Report Card](https://goreportcard.com/badge/github.com/gin-contrib/timeout)](https://goreportcard.com/report/github.com/gin-contrib/timeout)
[![GoDoc](https://godoc.org/github.com/gin-contrib/timeout?status.svg)](https://pkg.go.dev/github.com/gin-contrib/timeout?tab=doc)
[![Join the chat at https://gitter.im/gin-gonic/gin](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/gin-gonic/gin)

Timeout wraps a handler and aborts the process of the handler if the timeout is reached.

## Example

```go
package main

import (
	"log"
	"net/http"
	"time"

	"github.com/gin-contrib/timeout"
	"github.com/gin-gonic/gin"
)

func emptySuccessResponse(c *gin.Context) {
	time.Sleep(200 * time.Microsecond)
	c.String(http.StatusOK, "")
}

func main() {
	r := gin.New()

	r.GET("/", timeout.New(
		timeout.WithTimeout(100*time.Microsecond),
		timeout.WithHandler(emptySuccessResponse),
	))

	// Listen and Server in 0.0.0.0:8080
	if err := r.Run(":8080"); err != nil {
		log.Fatal(err)
	}
}
```

### custom error response

Add new error response func:

```go
func testResponse(c *gin.Context) {
	c.String(http.StatusRequestTimeout, "test response")
}
```

Add `WithResponse` option.

```go
	r.GET("/", timeout.New(
		timeout.WithTimeout(100*time.Microsecond),
		timeout.WithHandler(emptySuccessResponse),
		timeout.WithResponse(testResponse),
	))
```

### custom middleware

```go
package main

import (
	"log"
	"net/http"
	"time"

	"github.com/gin-contrib/timeout"
	"github.com/gin-gonic/gin"
)

func testResponse(c *gin.Context) {
	c.String(http.StatusRequestTimeout, "timeout")
}

func timeoutMiddleware() gin.HandlerFunc {
	return timeout.New(
		timeout.WithTimeout(500*time.Millisecond),
		timeout.WithHandler(func(c *gin.Context) {
			c.Next()
		}),
		timeout.WithResponse(testResponse),
	)
}

func main() {
	r := gin.New()
	r.Use(timeoutMiddleware())
	r.GET("/slow", func(c *gin.Context) {
		time.Sleep(800 * time.Millisecond)
		c.Status(http.StatusOK)
	})
	if err := r.Run(":8080"); err != nil {
		log.Fatal(err)
	}
}
```

### extended timeout on certain paths

```go
package main

import (
	"log"
	"net/http"
	"time"

	"github.com/gin-contrib/timeout"
	"github.com/gin-gonic/gin"
)

func testResponse(c *gin.Context) {
	c.String(http.StatusRequestTimeout, "timeout")
}

func extendedTimeoutMiddleware() gin.HandlerFunc {
	return timeout.New(
		timeout.WithTimeout(200*time.Millisecond),          // Default timeout on all routes
		timeout.WithExtendedTimeout(1000*time.Millisecond), // Extended timeout on pattern based routes
		timeout.WithExtendedPaths([]string{
			"/ext.*", // List of patterns to allow extended timeouts
		}),      
		timeout.WithHandler(func(c *gin.Context) {
			c.Next()
		}),
		timeout.WithResponse(testResponse),
	)
}

func main() {
	r := gin.New()
	r.Use(extendedTimeoutMiddleware())
	r.GET("/extended", func(c *gin.Context) {
		time.Sleep(800 * time.Millisecond)
		c.Status(http.StatusOK)
	})

	if err := r.Run(":8080"); err != nil {
		log.Fatal(err)
	}
}
```