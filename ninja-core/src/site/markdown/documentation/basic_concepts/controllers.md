Controller
==========

Introduction
------------
Controllers are the "C" in the MVC paradigm. Controllers will do actual stuff.
They get requests or part of requests, retrieve stuff from databases and
cause Ninja to return certain results.

A really simple controller
--------------------------

A basic controller looks like:

<pre class="prettyprint">
package controllers;

@Singleton
public class AppController {

    public Result index() {

        return Results.html();
    }


}
</pre>

The important conventions are that a controller method must return a <code>Result</code>. 
A result is in short just an object that holds response codes, 
content to render, headers and cookies that should be returned to the visitors of your website.

<code>Results</code> with the "s" at the end is a convenience class 
that allows you to generate a preconfigured <code>Result</code>.
For instance <code>Results.html()</code> will tell Ninja to render a html page. 
But you can also render json and other results.

By convention controllers live in the package "controllers".


Getting parameters into your controllers
----------------------------------------

A controller usually not only renders stuff, 
but also takes some inputs and does something with them.

You can get parts of the url inside your controller via two annotations:
<code>@Param</code> and <code>@PathParam</code>. Via @PathParam you can get 
variable parts of a route (described in more details in the routing section).
And @Param allows you to get Query or Form parameters.

Let's say and the user visits the following Url...

<pre class="prettyprint">
/user/12345/my@email.com/userDashboard?debug=false
</pre>

... and we got a route definition like that:

<pre class="prettyprint">
router.GET().route("/user/{id}/{email}/userDashboard").with(AppController.class, "userDashboard");
</pre>

We can then get all variable parts of this url via the following method controller
definition:

<pre class="prettyprint">
package controllers;

@Singleton
public class AppController {

    public Result userDashboard(
            @PathParam("id") String id, 
            @PathParam("email") String email, 
            @Param("debug") String debug) {

        //do something with the parameters...

    }

}

</pre>


More on objects Ninja can automatically provide you at controller level
-----------------------------------------------------------------------

Have a look at that method:

<pre class="prettyprint">
package controllers;

@Singleton
public class ApplicationController {

    public Result index(
            @PathParam("id") String id, 
            @PathParam("email") String email, 
            @Param("debug") String debug,
            Context context,
            MyObject myObject) {

        //do something with the parameters...
    }

}
</pre>

Ninja can not only inject PathParam and Param objects. But also the Context.
The context is a request scoped object that holds all informations of the current
request - parameters, headers and so on...

Even better: Ninja will parse an arbitrary object given in the method.
In the above case MyObject will be automatically parsed by Ninja. The way
it will parsed (Json, Xml, PostForm) will be determined via the content type request header.

Therefore you don't have to worry if
input is for instance Xml or Json. You simply get a parsed object.


## Ninja and content negotiation

In the last section we have already seen that parsing of objects works via content 
negotiation.

Rendering of objects works the same way and evaluates the Accept header for that.

Therefore a controller method like...

<pre class="prettyprint">
public void Result getUser() {
    User user = userDao.getUser();
    return Results.ok().render(user);
}
</pre>

...will render xml when the request accept header is "application/xml" or json when
the accept header is "application/json".

While content negotiation is pretty cool it sometimes breaks down in
reality. 

Especially if clients send strange headers or if you want to
be sure that a certain controller method always renders a return format. 

Then you can simply call use <code>return Results.json().render(user)</code> or 
<code>return Results.xml().render(user)</code> to enforce the rendering of a certain return format.




Rendering html
--------------

Rendering by convention has been already explained in the getting started part. But in short consider the
following class:

<pre class="prettyprint">
package controllers;

public class ApplicationController {       

    public Result index() {
        return Results.html();

    }
}
</pre>

This renders by convention a view in directory <code>views/ApplicationController/index.ftl.html</code>.

But you can specify your own html view, too:

<pre class="prettyprint">
package controllers;

public class ApplicationController {       

    public Result index() {
        return Results.html().template("views/AnotherController/anotherview.ftl.html");

    }
}
</pre>

In conclusion you can use convention over configuration in most cases - but you can also specify
a view to render explicitly.
