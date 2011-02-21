Crack
=====

 - Crack is crazy
 - Crack is [Rack][] for the console
 - Crack should not be smoked

Crack is **Console** **Rack**

What the hell?
--------------

So ... I was recently working on a CLI tool and I really wished that I had Rack Middleware.

I wanted to be able to allow people to build extensions that could do things like:

 - handle command line options
 - change the arguments before our executable sees them
 - modify the response before it gets printed to the console

I also wanted to find a *clean* way of handling global command line options that 
people add via extensions.

What does this have to do with Rack?
------------------------------------

Rack does a lot of things, it:

 - defines a standard Web request
 - defines a standard Web response
 - defines a Web application
 - defines Web middleware
 - and lots more

Crack does very similar things, it:

 - defines a standard Console request
 - defines a standard Console response
 - defines a Console application
 - defines Console middleware

One of the benefits of Rack is being able to host any application on any type of server. Crack doesn't have 
any known advantages like that (yet).  It just helps you write CLI applications that are easy to test, 
easy to extend, make use of middleware, etc.

Specification
-------------

### Request

A console request consists of:

 - `string[] args`
 - `IDictionary<string,object> data`

Initially, a request was just a `string[] args`, which is very simply and I like it.  I might go back to that.

However, I discovered that it was useful to have some other "request data" that middleware can setup and pass 
to other middleware or the application.  That's why we added a Dictionary of data.

**NOTE**: STDIN is an obvious candidate for a console "request" but it's a pain in the ass to work with, so I haven't touched it (yet).

### Response

A console response consists of:

 - `STDOUT`
 - `STDERR`
 - `ExitCode`

Pretty standard!  Instead of writing directly to `Console.Out` as you would in a normal CLI application via `Console.WriteLine`, you 
can call `response.Out.WriteLine` to write to STDOUT or `response.Error.WriteLine` to write to STDERR.

The `Response.Out` and `Response.Error` properties are `TextWriter` instances, meant to reflect `Console.Out` and `Console.Error`.

Behind the scenes, we persist the STDOUT and STDERR to `StringBuilder` instances which you can access directly via `Response.STDOUT` and 
`Response.STDERR`

    // Write to a response
    var response = new ConsoleRack.Response();

    // These do the same thing behind the scenes.  You can work with the TextWriter or work with the StringBuilder, directly
    response.Out.Write("hi");
    response.STDOUT.Append("hi");

### Application

An application is nothing more than a public static method that takes a `Request` and returns a `Response`

    [Application]
    public static Response MyApp(Request req) {
        return new Response("Called by app with: {0}", req.Arguments); // <--- shortcut to write to STDOUT
    }

You don't have to decorate an application method with the `[Application]` attribute, you can manually instantiate an application 
with a `MethodInfo` for your method.  But using the `[Application]` attribute will let Crack auto-detect your application method;

### Middleware

A middleware is nothing more than a public static method that takes a `Request` and an `Application` and returns a `Response`

    [Middleware]
    public static Response WrapResponse(Request req, Application app) {
      var response = app.Invoke(req);
      
      // let's wrap the response with stars ...
      response.Prepend("***");
      response.Append("***");

      return response;
    }

Just like Rack middleware, a Crack middleware can perform some action before and/or after invoking its internal 
application.  You can also return directly from the middleware, eg.

    [Middleware]
    public static Response Version(Request req, Application app) {
      // ofcourse this could use an option parsing library
      if (req.Arguments.Length > 0)
        if (req.Arguments[0] == "-v" || req.Arguments[0] == "--version")
          return new Response("MyApp version 1.0.5.9");
    
      return app.Invoke(req);
    }

Using that middleware with out application, calling `MyApp.exe --version` would return automatically, 
displaying the version number.  You main application code would never even be called!

## Is this shit really useful?

To me: YES.

To you: UNKNOWN.

If it looks interesting, give it a try.  You don't even have to use the Crack library, necessarily.  The main 
idea of this library is the use of middleware in CLI applications as a way to easily organize and extend functionality.

Crack also makes it *really* easy to test your CLI applications because you don't need to have your tests run the 
actual .exe executable.  You can manually invoke your application and inspect the Response you get back, without 
executing any external processes.

License
-------

Crack is released under the MIT license.

[rack]: http://rack.rubyforge.org/