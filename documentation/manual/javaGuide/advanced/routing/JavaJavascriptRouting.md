# Javascript Routing

The play router is able to generate Javascript code to handle routing from Javascript running client side back to your
application.  This aids in refactoring your application, if you change the structure of your URLs or parameter names,
your Javascript gets automatically updated to use that new structure.

## Generating a Javascript router

The first step to using Plays Javascript router is to generate it.  The router will only expose the routes that you
explicitly tell it to, thus minimising the size of the Javascript code and also avoiding unnecessarily giving away all
the information about the structure of your application.

There are two ways to generate a Javascript router, one is to embed the router in the HTML page using template
directives, the other is to generate Javascript resources in an action that can be downloaded, cached and shared between
pages.

### Embedded router

**This method is currently not supported in Java, though will be in future**

An embedded router can be generated using the ``@javascriptRouter`` directive inside a Scala template.  Typically this
might be done inside the main decorating template.

    @helper.javascriptRouter("jsRoutes")(
      routes.javascript.Users.list,
      routes.javascript.Users.get
    )

The first parameter is the name of the global variable that the router will be placed in.  The second parameter is the
list of Javascript routes that should be included in this router.  In order to use this function, your template must
have an implicit RequestHeader in scope, for example this can be made available by adding ``(implicit req:
RequestHeader)`` to the end of your paramteer declarations.

### Router resource

A router resource can be generated by creating an action that invokes the router generator.  It has a similar syntax to
embedding the router in a template:

    public static Result javascriptRoutes() {
        response().setContentType("text/javascript");
        return ok(
            Routes.javascriptRouter("jsRoutes",
                controllers.routes.javascript.Users.list(),
                controllers.routes.javascript.Users.get()
            )
        );
    }

Having implemented this action, and of course adding it to your routes file, you can then include it as a resource in
your templates:

    <script type="text/javascript" src="@routes.Application.javascriptRoutes"></script>

## Using the router

The router uses jQuery by default to make requests.  Making a call is as simple as:

    jsRoutes.controllers.Users.get(someId).ajax({success: /*...*/, error: /*...*/});

The router also makes a few other pieces of information available, such as the ``url``, the ``method`` and the
``absoluteURL``, as properties on the route.

## Using a custom ajax method

If jQuery isn't your thing, or if you'd like to decorate the jQuery ajax method in some way, you can provide a function
to the router to use to perform ajax queries.  This function must accept the object that is passed to the ``ajax``
router method, and should expect the router to have set the ``type`` and ``url`` properties on it to the appropriate
method and url for the router request.

To define this function, in your action pass the name of this method as the second parameter, eg:

        return ok(
            Routes.javascriptRouter("jsRoutes", "myAjaxMethod",
                ...
            ));

