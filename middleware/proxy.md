# Proxy

Proxy middleware for [Fiber](https://github.com/gofiber/fiber) that allows you to proxy requests to multiple servers.

## Signatures

```go
func Balancer(config Config) fiber.Handler
func Forward(addr string) fiber.Handler
func Do(c *fiber.Ctx, addr string) error 
```

## Examples

Import the middleware package that is part of the [Fiber web framework](https://github.com/gofiber/fiber)

```go
import (
        "github.com/gofiber/fiber/v2"
        "github.com/gofiber/fiber/v2/middleware/proxy"
)
```

After you initiate your Fiber app, you can use the following possibilities:

```go
app.Get("/gif", proxy.Forward("https://i.imgur.com/IWaBepg.gif"))
// Make request within handler
app.Get("/:id", func(c *fiber.Ctx) error {
	url := "https://i.imgur.com/"+c.Params("id")+".gif"
	if err := proxy.Do(c, url); err != nil {
		return err
	}
	// Remove Server header from response
	c.Response().Header.Del(fiber.HeaderServer)
	return nil
})

// Minimal config
app.Use(proxy.Balancer(proxy.Config{
	Servers: []string{
		"http://localhost:3001",
		"http://localhost:3002",
		"http://localhost:3003",
	},
}))


// Or extend your balancer for customization
app.Use(proxy.Balancer(proxy.Config{
	Servers: []string{
		"http://localhost:3001",
		"http://localhost:3002",
		"http://localhost:3003",
	},
	ModifyRequest: func(c *fiber.Ctx) error {
		c.Set("X-Real-IP", c.IP())
		return nil
	},
	ModifyResponse: func(c *fiber.Ctx) error {
		c.Response().Header.Del(fiber.HeaderServer)
		return nil
	},
}))
```

## Config

```go
// Config defines the config for middleware.
// Config defines the config for middleware.
type Config struct {
	// Next defines a function to skip this middleware when returned true.
	//
	// Optional. Default: nil
	Next func(c *fiber.Ctx) bool

	// Servers defines a list of <scheme>://<host> HTTP servers,
	//
	// which are used in a round-robin manner.
	// i.e.: "https://foobar.com, http://www.foobar.com"
	//
	// Required
	Servers []string

	// ModifyRequest allows you to alter the request
	//
	// Optional. Default: nil
	ModifyRequest fiber.Handler

	// ModifyResponse allows you to alter the response
	//
	// Optional. Default: nil
	ModifyResponse fiber.Handler
}
```

## Default Config

```go
// ConfigDefault is the default config
var ConfigDefault = Config{
	Next: nil,
}
```

