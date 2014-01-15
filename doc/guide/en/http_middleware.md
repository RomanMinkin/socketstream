# HTTP Middleware

SocketStream provides a stack of Connect HTTP middleware which is used internally to serve single-page clients, asset files and static files (e.g. images) in `client/static`.

The Connect `app` instance is accessible directly via the `ss.http.middleware` variable within `app.js`. The stack is loaded in this order:


Connect middleware order
---

Any prepended middleware is loaded here, then we load:

- compress
- cookieParser
- favicon
- session

Any appended middleware is loaded here, then we load:

- eventMiddleware (handles loading client files when running in development mode)
- static
- staticCache (optional)

Prepending custom middleware
---

SocketStream allows you to prepend new middleware to the top of the stack (to be processed BEFORE SocketStream middleware) using:

    ss.http.middleware.prepend()

For example you could add:

    ss.http.middleware.prepend( ss.http.connect.bodyParser() );

Appending custom middleware
---

Because SocketStream adds `connect.cookieParser()` and `connect.session()` to the stack, if the middleware you're wanting to use requires sessions support (i.e. access to `req.session`) it will need to be appended to the bottom of the stack AFTER SocketStream middleware has been loaded as so:

    ss.http.middleware.append( everyauth.middleware() );

Apart from determining where the middleware should be added, the `prepend()` and `append()` functions work in exactly the same way as `connect.use()`.

Loading staticCache middleware
---

In order to avoid making repeated fs.readFile requests for serving static files, you can load Connect's staticCache middleware, by passing this:

    ss.http.set({
      staticCache: {
        maxObjects: 128,       // The number of objects to store in cache
        maxLength: 1024 * 256 // The file size limit for storing a file in cache, 256Kb
      }
    });

This will load the StaticCache middleware. You will notice that when you do this, you'll see this warning when loading the app:

    connect.staticCache() is deprecated and will be removed in 3.0 use varnish or similar reverse proxy caches.

As SocketStream 0.3 currently uses 0.2 of connect, this is an issue that we will address when we come to upgrading connect to 0.3.

We did try to look at using <code>st</code> as an alternative, but found that it was ~ 2x slower at serving static files than using connect's static + staticCache middlewares. It is something that we will look at again in the near future. 

Passing custom HTTP middleware settings
---

You can pass custom http middleware settings via <code>ss.http.set</code>:

    ss.http.set({
      static: {
        maxAge: 30 * 24 * 60 * 60 * 1000 // cache static assets in the browser for 30 days
      },
      staticCache: {
        maxObjects: 128,       // The number of objects to store in cache
        maxLength: 1024 * 256 // The file size limit for storing a file in cache, 256Kb
      }
    });

At the moment, you can use this to set custom options for the following SS middleware:

- static
- staticCache
- session (secure flag only)

We will find a way to make all of the middleware customisable in the near future.