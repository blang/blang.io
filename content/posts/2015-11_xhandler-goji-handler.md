+++
date = "2015-11-12T20:51:49+01:00"
title = "Use xhandler inside goji router"
+++

I found it a good way to test multiple golang http routers using [xhandler](https://github.com/rs/xhandler) as a uniform handler type. That way you already use a proper handler with the ability to transport request/route based values without binding to a specific framework.


<!--more-->

xhandler as goji handler
========================

For example you could use the [goji framework](https://github.com/zenazn/goji/).

```go
func Index(ctx context.Context, w http.ResponseWriter, r *http.Request) {
	params := ctx.Value("urlparams").(map[string]string)
    ...
}

func handle(ctx context.Context, handlerc xhandler.HandlerC) web.Handler {
	return web.HandlerFunc(func(c web.C, w http.ResponseWriter, r *http.Request) {
		newctx := context.WithValue(ctx, "urlparams", c.URLParams)
		handlerc.ServeHTTPC(newctx, w, r)
	})
}

// Middleware
c := xhandler.Chain{}
c.UseC(xhandler.TimeoutHandler(2 * time.Second))
// Router
mux := web.New()
mainContext := context.Background()
mux.Get("/:name", handle(mainContext, c.HandlerC(xhandler.HandlerFuncC(Index))))
```
