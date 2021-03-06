# Using Symfony Forms with Rich Domain Models

***TIP
SymfonyCon 2018 Presentation by [Christopher Hertel](https://github.com/chr-hertel) &
[Christian Flothmann](https://github.com/xabbuh)

With the popularization of DDD people started shifting from anemic models with
only getters and setters to a rich model describing the state changes in specific
methods. This way of designing models does not play well with Symfony forms. User
provided input is inherently invalid while we want to maintain certain invariants
in our domain model. A common approach to overcome these limitations is to create
data transfer objects our forms are then bound to. This can lead to lots of mapping &
glue code that might be cumbersome to write and maintain.

But couldn’t we do better? In this talk we will discuss the different aspects of a
rich domain model that makes it hard to use it in conjunction with the Form component.
We will then look at the possibilities to hook into the data flow of the form handling
and discover how we can modify it to interact seamlessly with our model.
***

Welcome. Hi. Hello. The stage is huge, it's intimidating, but we're pretty
happy that so many of you joined us. And we're talking about Symfony forms. So
who have you used Symfony forms before? Oh, that's an easy one. Right? And
we're talking about Symfony forms using rich domain models. Who's familiar with
the term rich domain model. So quite a few and uh, but we are starting with
introducing ourselves.

## Hello Christian and Christopher

So let's do this. That is Christian, Christian is working for SensioLabs, in
Germany and uh, besides being a software developer at SensioLabs he's working
as a core member of Symfony software and Symfony docs.

And this guy next to me is Christopher, he's working at SensioLabs in Germany
too. And he's also one of the organizers of Symfony user group in Berlin. So if
you ever come to Berlin and you're lucky to, to attend the user group, you may
meet him there too.

## The Use Case (Starting Code)

And Christopher will now guide you through a simple example application that we
created to show you the concepts of our talk. So in order to make it easier to
understand everything.

Yeah. Well, so we're doing it hands on with a simple use case and our use case
is a simple product CRUD. CRUD is an abbreviation for create, read, update,
delete. I think you're familiar with that principal, a principal and a, it
looks like that. It's a simple interface. Looks a lot like bootstrap I guess.
And there are two forms in it. So, one about the category and one about the
product. And the domain is a product with a name, a category, and a price.
Pretty simple. On the category itself has a name and an optional parent
category pretty simple as well. And the price object has an amount tax rate and
a currency.

So, uh, that's the workflow of the forms and we're talking just about the
product form with a create and edit. So you're quite familiar with that. I
guess it's a very simple action that handles, uh, both steps the create and
edit. So let's see, at the very basic solution, that's pretty much a, like the
best practice document in the documentation. So we're having a product
controller with a form action and that's just creating the form type, handling
the request, binding request to the form type and then the very typical, if
it's submitted and it's valid structure with a redirect at the end or the
rendering of the form optional with data and errors or plain.

The form type itself, is very straightforward as well. So we're having a
product type with a name category, price amount, price tax and price currency.
So we're not having a price object on this level, but three price attributes
and the product type. And we're binding the product product class to the type.

Next up, the `Product` class. So the model, which is the focus of our talk
today, right? The model. And we're having a simple, a simple product entity
with a private properties with assertion annotations on it and simple getters
and setters, right? So pretty straightforward.

## The Data Flow

So, um, what's happening now when we are getting our forms. So, um, this is the
form as you will see it in your browser and you will enter data and so on, and
your browser will submit an HTTP request. You can see on the headers and then
you have the body of the POST request with all the data coming from the form.
And inside your your front controller, the incoming request will be transformed
into a request object which is part of the http foundation component, which is
an object oriented layout on top of the super global. It's basically what PHP
offers you to handle HTTP requests and this is what it looks like if we dumped
the structure. So you basically have an object around a structured array. And
then inside your controller on this side, you have the request object on top,
which is then passed down into the handle request method, which means that the
request is handled by the form component and we can later check if form was
submitted or not. Yes, right here. And here the form is taking into account
that we are binding our form to the product class.

This was the final object, all the data that was coming into our controller was
bound by the form and is now part of the form object.

## Vendor code, Model code, Glue Code

So let's take a look at three columns of our three types of code that are
involved to solve our problem. First, there was third party code that we just
want to use that we don't want to maintain that we don't want to write
ourselves, which means the front controller, which is just bootstrap code from
a application. We have the HttpFoundation component as the object oriented
layout on top of HTTP. We have the form component of course. And we have the
validator component which we have seen in the product model than we have added
assertions to our model to, to ensure that domain roots are taken into account.
Then we have our domain layout, which means the core part of our business.
Everything that's individual for our application, which is basically the
product class the category class and the price class or the price price object.

And then we need to write some glue code to somehow wrap all these things
together. And this is the controller, which means we are handling the request
that we are dealing with a request and returning a response. We need to create
a form type which describes how we map our request to our model and if we take
a look at this from a graphical point of view, we have the data flowing in
from, from the left and going through all the layouts in our model, our model
is bound to the form which is then handling the request and putting data back
into the model. So usually we want to focus on these yellow blocks, which is
the core part of our business. And we usually want to reduce the green part. So
the part that is just there for coupling our domain to the framework to the
least minimum.

Okay. So that's the basic setup. Probably nothing new for you.

## Anemic Domain Models vs Rich Domain Models

And the next step is that we look at our model, um, and we need to
differentiate about anemic domain model and rich domain model. What we saw, the
product type and the product is a dynamic domain model which is focused on
data. So there's a structured way of having data in an object. It's very easy
to implement and to maintain even you can generate it using helpers, like
MakerBundle or a or the PhpStorm has some utilities as well. And it contains
little or no logic at all, so maybe no constructor, no validation logic and
setters, and it's not guaranteed that the data that is in their object is valid
or is consistent. So that might be a problem.

Rich domain model combines data and logic and it is valid by design. So we have
mandatory constructor arguments, for example. So we need to give a category
name and category can't exist without a name. It's quite easy to test. Same is
true for the anemic domain model, but some might argue it's not really sensible
to test the anemic domain model. And we can now create state transitions by
name and test them and um, do assertions on transitions.

And that's the last point, defined state transitions and we can, um, the
concept is to be more explicit, explicit on the names of state transitions and
not just have set name, but uh, maybe rename process.

Important small disclaimer, we don't want to lecture you about using rich
domain models is the best way to do it. It's sensible to choose between them.
Right? And that's, that's up to you. And uh, this guy, Martin Fowler wrote in
2003 that anemic domain models are anti-pattern and then stack overflow was a
bit curious if it's an anti-pattern or not. And then there was discussions how
they differ and how to avoid anemic domain models, how to find the right
balance between using both maybe. Um, someone came up with this idea. It's not
an anti-pattern anymore. Someone says it's SOLID design. Some said it's not
SOLID, rich domain is SOLID and stack overflow was very confused. So that's the
basic bottom line at stack overflow article about it and how not to agree about
software design. And we just want to point out their differences and reasons to
use one of them.

Anemic models are good for prototyping, very, very easy to use, easily
generated and rich models are more about clean code and object oriented, um,
testability and yeah, we can design our domain in it including using name
transitions and process. Um, but it's your choice, right? But the next point is
we want to show you how to use rich domain models with Symfony.

## Rich Models in Symfony Forms

So if we take a look at our former model, we didn't have any defined state
methods since we had just setter and getter methods so you would have to call
them however you like and then you would have to run validation after. That's
not how we want put that. So the first step is to, to, um, require every
attribute to be passed when we are constructing the product. So we have to put
the name, the category and the price. For category and price we can be sure
that both are already valid because these routes apply there too. But for the
name we need to, to perform some, some validation. We need to, to make sure
that that the constraints that we defined are still applying here. And you can
also see that we have removed the validation assertions, so this is something
that is now happening inside our, our model. The same, happens here when we are
renaming our product and you can also see now that we don't have a set named
method anymore, but we have a rename method now so this is more like, like
describing what we are doing with our model.

And here's our validateName method, which was just basically checking the
length of our name. The price is a bit special because the price itself is not
identified by an id or something like that. Um, price, just, well having a
value. And this is what makes it so special because one price object can now be
shared between different products. But if we are doing that now we want to
compare them. So the value must not change. And this means that our price
object has to be immutable. We will see that later, what that means for our
forms after that. So we're doing all the validation again. And the same applies
for the category. So if we are now using this model without making any other
changes, we would run into many different exceptions and well that's not really
funny. Um, so we need to do something about that to make our forms usable with
our new model.

## Solution 1: Data Transfer Objects

So the first solution that we came up with is using data transfer objects or
DTOs. That's quite easy and looks like this. That's the DTO having public
properties and the validation assertions on it. And then we are having some
kind of mapping methods where we map the rich domain model to an DTO and back
again to an entity. So right here we can check if the DTO, if there are changes
and we can explicitly call those rename, moveToCategory or even the Costs
method and apply those changes only when we need them. So that's pretty easy to
do. And the code looks a bit like the model before, right? Um, the product type
is actually the same besides the small change down there, changing the data
class. The controller is a bit different because we have to map the DTO from
entity and to get the DTO from the form and map to the entity again before
persisting it. So quite simple steps, but we now can use rich domain models.
Right?

Okay. From the user's point of view, that's not so much that is just changing
the interface still looks the same as before the data that is sent is still the
same, it's still transformed into the request object, as before. And, the
interesting changes are happening here now because as Christopher already
explained, the entity is first transformed into the DTO because we need to, to
unbind the form to this DTO. We're handling the request here now and here we
see the binding to the DTO again, we have the reside here this is the DTO and
we need to um, transform this back into the entity. So we are catching the data
from form and parking it here. Um, the entity so we can now persist this entity
again.

This three column model that we have already seen before. And what changes here
now is the right column, because we now have additional glue code. We have
written DTOs and this is more code than before, um, which you can see here too.
So before, we only had our model here and now we also have a green rectangle on
the left and on the right and well basically we want to reduce this part. Now
we have have added some, so um, maybe that's not the best solution we can come
up with. But the solution enables us to use rich domain models by decoupling
the model from the form type and it's very easy to implement and maintain as
well. So the DTO was quite simple. Um, but it's additional glue code. Right?
And I don't know if you saw that, but we have redundant validation rules,
right? We have validation in our rich domain model and we need to have
validation in our DTO. So this one has an impact on the maintainability. Next
solution we came up with is harder.

## Solution 2: Data Mapper

Okay. Who of you has implemented a data mapper for forms before? Okay, one,
two, three. Okay. 10 or 20 or so. So I'm not so many and yes, I think it's
really the hardest solution, but we want to explain it nonetheless to show you.
Um, and I think it's, if you've never used it before, it might be quite fast,
but it's not the point to explain data mappers, we just want to show that it's
possible to use data mappers as a solution to decouple and use rich domain
models. Yes. So our controller now looks again the same as it was in our first
solution when we were using the anemic domain, which basically means that we
are now again binding the product entity to the form.

And the form is the part where things changed now. First we introduced a new
price object which is just extracting the price related properties from the
product and put it into a new model. And now we have a data mapper, so a custom
data mapper. The form component ships with a default one, but we cannot use
that. Um, so we add a new one here and we just say, okay, The data mapper that
we want to use is implemented inside, this product type. You could also move it
into another class that doesn't really matter here and a data mapper has two
methods, methods. The first one is this one is called mapDataToForms and it's
populating the form with the initial initial data. So we, basically here take
the subtypes and set the data based on the model that was passed here. So the
data variable is our product entity.

And after the form has been submitted, we must remap the data from the form
into our entity and this was the part where things are a little bit more
complicated now because we don't have any setter methods anymore. Um, so we are
first getting the data from the form, comparing them to the data inside our
entity, and then we call our state transitions, state transitioning methods. So
for example, rename here or here, we are moving our product into a different
category or the price changes and we are calling this method in this case.

When we are creating a new product, we need to make sure that the form is as
populate a new product entity is populated with the submitted data and we are
doing this here with the empty data option. Passing it a closure where we get
the form and we can extract all the data from the form and pass it to the
constructor of our product entity. We need to make sure here that we catch all
the exceptions that could occur so that the user does not see an internal
server error, but we can present them some kind of a nicer error message.

The data flow, it's still the same, or basically the same. Here's a little bit,
a little difference that you can see because we now have the price as an
additional nested element, but that's just a small detail here. Yeah, we moved
the price values to a lower type, right, so we have a price type right now.
Submitted data is still the same. The user does not change what they want to
enter there and you can see here that the controller now is the same as in the
first example. The request was handled again and here is the difference, we are
now not not, the FormType still is bound to the product entity, but the
important part now is that we have created our own data method that is used
here. And the resulting Product, that's the important part is still maintained.
We are basically changing what's happening when our form is handled, but we're
not changing our model.

Here our three column model that we are familiar with now. Changes a bit. Now,
we don't have DTOs, but we have data mappers and the validator component is
gone too because in the process, step one, we introduce in the second step when
we introduced our rich domain model, we are implementing our domain assertions
by by doing the validation inside our methods and throwing exceptions and now
that we don't have DTOs anymore, we also do not need the validator component
anymore.

But the glue code part still makes a big part of our application. We have now
data mappers that we need to maintain, that we need to change where we need to
think about how can we test them, which is yeah, massively different story
because testing data mappers really is not fun and um, but you probably want to
do that because this is an important part of your application.

So the bottom line with data mappers enabled, enables us to use rich domain
models and that requires advanced knowledge about the form component and the
extension points of the form component. And this is additional glue code as
well. Like it's not a with redundant validation rules we got rid of that but
it's quite hard to test and it's quite hard to understand if you're getting in
a new project and there's data mappers all over the place.

## sensiolabs-de/rich-model-forms-bundle

So, DTO's, redundant data mappers are hard.

Maybe we can, we can ask questions. So who have you after just having heard
this, um, would like to use rich domain models with Symfony forms? Oh, okay.
That's not so many. Would you prefer. So I think it was five or four people.
Would you use DTOs? Or would you use data mappers? Oh, okay. Half and half I
think. Okay. But uh, yeah, I mean it's clear that this is maybe not the best
solution yet.

We have another one, RichModelFormsBundle. Yeah, we could, we could work on the
name. Okay. But it's already out there. So it's RichModelFormsBundle. So, um,
you can easily install it. It's still experimental. So we're working on the
idea and uh, today is once again a kick off for a discussion, maybe. The
installation is quite easy with a tool called Composer.

### Bundle Features

Um, and now we're talking about the features. So yeah, basically the idea
behind this bundle is, and that's why we have shown you the third solution that
the, the data mapper approach maybe could be generalized to something where you
do not need to write the code yourself, but you would just configure this
generic data mapper so that it does what you want based on your use cases. And
um, yeah, now it's your turn. Thanks. Again.

First feature different write and read property path, there is an option
already property paths, whether you can configure how the property accessor,
um, talks to your model, but the new thing is that we can use, read and write
property path, um, different. Because the problem with the initial, um, within
default data mapper is that the getters and setters must really be named
getName and setName.

In this case, we'll look at this example and um, yeah, we didn't want to have a
setName method because in our domain it's called rename. So, um, we thought,
okay, why don't we split the property path in, read and write. And yeah, that's
the name, read_property_path and write_property_path. So in our case we are
saying, okay, when you're reading the value, you just call getName and please
call rename when we want to change the value. You could also pass a closure
there if you need to to maybe call different methods based on the value of a
checkbox, for example, because you maybe have, enable and disable and not
something like set enabled with true and false.

A quick side note, the bundle implements a couple of form options, additional
form options that you can use on your form type. So, exception handling. So our
domain model throws exceptions on invalid input or on maybe in constructor,
like this was the price object, right? Yes. And what you can now use is the
expected_exception option. We are not really satisfied with this name because
it reminds us more what we're used to when we are, um, using php unit, but in
fact we don't want the exception to be thrown. We just want the bundle to
intercept here. So yeah, if you have ideas for a better name, we would really
welcome any suggestion here. Um, but I think the idea should, should be pretty
clear so you can, can define which exceptions can occur here and which are
valid to be thrown because they might be thrown by domain validations here and
we want to show something to the user, some kind of of nice error message to
show them what they did wrong.

Yeah, I guess the, the error handling of the domain is quite important part. We
need to tackle because the exception message is not very good for the
interface, so we need to, uh, to decouple that. And maybe there are some other
validation concepts out there, not using exceptions, but using a message bags
and something.

One thing we didn't mention yet is that all type errors that may occur because
you passed the wrong data type to the method will automatically caught being
caught too. So if you, um, have a model, you will often have realized that you
will have to allow null values being there, and this will be intercepted as
well.

### Mandatory Constructor Arguments

Next one, mandatory constructor arguments. The form component, uh, creates
objects, right? And, um, there's an empty data option and you can hand over
already created objects. But in this case where we have an immutable value
object or a value object, it's quite hard. This also applies for product
entity, and yes, we could work around that by writing this closure ourselves,
but that's um, 10 lines of code ourselves for every form type. So, um, let's
define a new option which is called factory and factory can be the name of a
class, which means, um, all the, um, values of this type would be passed to the
constructor of this class. You could also, um, pass a closure on any PHP
callable, um, so maybe if setting method on another class or so, and this would
then return a new object. But only for cases where the form is populated
without any initial data object. So when you maybe are creating a new product
or so.

And the names of the properties, uh, the names of the fields are mapped to the
names of the arguments so you can easily change them, um, to, to have a
different order, something.

### Immutable Value Objects

Next one, immutable value objects. So once again, the same code, um, but we
need to create a new object on change because we can't um change the basic, uh,
the base data object, right?

And um, okay, we already have the opportunity to define a factory. So let's
just add another option and let's define it immutable because that's what we
are dealing with here. Immutable objects and if we set this option to true
every time you submit new data a new object will be created and the initial
data object that was bound to the form is still the same as it was before we
were handling the form.

So about the implementation, controller still looks the same as in the first
solution. The form type is a bit different now because we have added all these
options that we have shown you before, but that's the part where we now need to
make adjustments. From the point of view of the model we are now still using
the rich domain model, which is the main difference to the first solution where
we were using the anemic domain model. In the data flow there's not so many
differences here, here the request is coming in and behind the scenes the
important part is now happening as part of a RichModelFormsBundle. And the
code, um, we had our custom data mapper implementation before this is now gone
because we have the new bundle. And the important part here is the switch or
the movement from the right column, which is what we have to maintain to the
left column, where there is code that hopefully just works as it is supposed to
do and does not need to be maintained by us. So, um, we are here and we can now
focus again on our core domain and we don't have to deal with stuff that is not
part of our business. And well, we are not making money with it probably.

It generalizes the previous approach of the data mapper, so behind the scenes,
um, the bundle implements a couple of data mappers and creepy stuff. Um, but we
have additional data options form options and um, they are, the target is using
rich domain models so we can improve, uh, the goal is to improve that and it's
released, but we're still working on it. I think you tagged 0.1 yesterday or
today. Yes.

Some say there are some installations on production already, but I wouldn't do
that maybe. It's not feature complete. Rich domain models can be very different
because they belong to your domain. And we just, we just targeted those cases
we came up with. So maybe you have other targets and use cases and you could
easily share them with us on GitHub. That's, that's your part to do maybe, um,
yeah, that's a lie. But, uh, I don't know the project, maybe that's not that
big. So it's your turn to help us give us input about, um, maybe the namings of
the options and maybe your use cases. Just maybe just try it in your
application yet, not in production maybe, but, but on your test server, or on
your local development machine. Try it out and give us feedback, what works,
what maybe does not work and be nice and where you need to write your own glue
code once again. So we want to get rid of the glue code, right?

So, DTOs to sum it up, um, it's easy to implement, duplicated validation, it's
additional glue code. Data mappers, hard to implement, hard to test, and
additional glue code as well. New Bundle, no glue code on your side, it's
experimental and maybe we can bring some of those ideas to the core in the
future.

Okay, thanks.

That's it. Thank you.

## Questions

Three minutes for questions. Please don't ask hard stuff.

Oh sorry. Does this work? Hello? Can you go to a slide where you actually put
the data array the exception in and stuff?

Sorry, I don't hear anything.

Can you go to the slide where you put your extra property paths and everything
in there to the form type?

Um, maybe you can ask the question without the mic and we'll just repeat it. I
think it's better to understand. Maybe we can. No, we cannot.

This one?

Yeah.

So the question is if you can use...You mean for this or for that? Oh yeah,
that's how you use the form component. That's okay.

Can you repeat the question please?

That's, that's part of the interface that we cannot change here because that's
coming from the, from the core. So the question was, do we need to pass an
array here or can we pass an object? So yeah, that's not, that's not possible.

Yeah. But then we would have to work around much more stuff. Yeah, that's
probably not what you want to do.

Alright. Christian and I will be around conference social as well, so see you
there. Thank you.
