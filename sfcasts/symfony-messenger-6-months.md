# Symfony Messenger: 6 months already and more to come

***TIP
SymfonyCon 2018 Presentation by [Samuel Roze](https://github.com/sroze)
The Messenger component brings asynchronous processing to Symfony: using message bus(es) you are able to decouple your application and route some (or all) of these "messages" to transports such as the built-in AMQP transport. It’s been more than 6 months since we’ve merged the new Messenger component in Symfony. We will explore the different use cases it has been used for so far and how it will continue to involve in order to facilitate even more.
***

Hello, good morning. So, roughly six months ago we released the Symfony 4.1. And
that was the first time that we actually introduced the Symfony Messenger
Component. So today what I want to talk about is obviously this component. I
want to quickly introduce it and that's going to be the first part of the talk.
And then sort of like explain what's, what has changed in the, during the
transition from the 4.1 and the version 4.2, that was actually released a few
days ago I think.

## Hey Samuel!

Before that, I'll introduce myself very quickly. I'm Samuel Rozé, I'm part of
the Symfony core team, the ApiPlatform as well and I've created a bunch of open
source things like ContinuousPipe and Tolerance. I don't have any time to
present them but you should check these out, it's quite interesting.

Very quickly, the startup I'm working with, we are on a mission in trying to
help the old people, the elderly to stay healthier and longer at home instead of
going to retirement houses. It's, to me a very interesting project. You should
check the website, it's birdie.care. Obviously as most of the technology
companies we are actually hiring. So if you're interested, talk to me.

## Symfony Messenger? The 5 Concepts

So Symfony Messenger. So what it is? There are five main concepts within the
actual, within Symfony Messenger. The first one is messages. You could have
guessed it. So it's your PHP object. It's a PHP class that you will create that
you will dispatch somewhere and you will dispatch this message to a message bus.
Then once the message has been dispatched to the bus, the bus's responsibility
is to actually call some things that are going to contain the business logic,
your business logic, and these things are called message handlers. It's a bunch
of classes, no specific requirement, it's, um, what you want to execute when you
receive, when the code receive the message basically. You can have zero to N
handlers for a given message.

Just these three points are really super valuable. And if you're into, um, sort
of design patterns or sort of like interesting things like CQRS if you've heard
about it, the idea is that we can actually structure an entire application
around the bus. And we can actually structure an entire application around *a*
bus or multiple buses. But the point is that we can actually do really, really,
um, decouple who is dispatching a message from who is actually handling the
message. I won't go that much into the details. The important takeaway here is
that just these 3 concepts, without talking about asynchronous, without talking
about queue all of that, is already extremely valuable and you should actually
try to play with them.

Turns out, that also we didn't have a solution for asynchronous things in
Symfony. So one of the other concept in the Symfony Messenger component is the
notion of transport. So we can actually say, well, instead of actually directly
calling the message handler or your message handler, well, the bus can actually
switch to something else. We can actually route the message to actual, to a
transport. And the transport is really something a bit abstract, but something
is responsible of talking to your, to a third party. The most famous one is
RabbitMQ being a third party, so queuing mechanism. But technically it can be
actually anything, can be Gearman if you want to do some distributed processing
for your messages, can be an API, there is no real limit. And the last missing
dot here is that if you actually dispatch the message and you route it to a
transport, it's in the queue, but nothing will happen. So the fifth sort of
concept is this notion of worker. So it's just something that is going to say,
or ask the transport: "hey, tell me when I receive a message" And when a message
arrives, it will actually call your message handler.

## Messenger in Diagram

The same thing as a sort of a graphical representation. You've got a message,
you dispatch a message to a message bus, that is handled by one or zero or N
handlers.

If you really want to, you can route this message to a subset of transports: to
one or multiple transports, instead of directly calling the handler. These
transports are responsible for talking to these guys, and we've got the worker
that is responsible for receiving the message from the transport and dispatching
them back to the message bus. Now the important thing here is that actually, if
you actually route the message to a transport within your HTTP server - or your
like main PHP thread - um, you actually dispatch the message and your handler
won't be called. It will be super fast, you will hand it to the queue. And you
need to run this worker somewhere else. So you won't be on your web server,
might be somewhere else on another machine, but the code that your handler
contains, or your message handler contains, is going to be executed outside of
the PHP context or the web context. That's the main, that's the main concept.

## Using Messenger!

So now I'm going to show you exactly how from a user perspective or developer
perspective, how can we use it.

You all have seen, I guess what a Symfony controller looks like. Now it's fairly
recent, but it's a few versions already, we can inject services directly in the
action arguments. So in this example here, I'm going to inject a
`MessageBusInterface` that is already configured and wired for you if you have
the component installed on your Symfony framework application. What you can do,
as I mentioned, you can actually dispatch using, calling dispatch method, on the
bus, you can dispatch your message. There is no requirement. I mentioned that
already, but there is no requirement: that's how your message looks like. It's
your own php class. So whatever you want to do. If you want to actually send a
notification, you can call it a ` SendNotificationMessage`. If you want to, I
don't know, buy pizza, it can be `BuyPizzaMessage`. You can put the properties
you want, you can put the method, the getters, the setters you want it is
completely your own code. There's nothing... to compare with the
event-dispatcher for example, where the event needs to extend the `Event` class,
well, we don't need that anymore. At least in this component.

Once you've dispatched your message, guess what? You need a message handler.
Same idea: there is no specific requirement on the message handler. The only
thing is it needs to be a PHP callable. So typically it can be an anonymous
function. In our beautiful world of classes and all of that, it is a class that
has a method, a magic method named `__invoke()`. If you type-hint your message,
type-hint your message, as the first argument of the `__invoke` method, then
Symfony will automatically understand that it is this message that you want to
handle. And when you dispatch the message this handler is going to be called.

There is a slight tweak here. It's just that in order for Symfony to
automatically configure your handler, and automatically add the tag required to
actually discover your handlers, you can use this marker interface, and this
marker interface is called `MessageHandlerInterface` and contains nothing. You
don't have any requirements, but just to say to Symfony: "Hey, can you wire that
handler into the message bus?" Okay, cool.

Now, if you run that code, your handler, whatever your domain logic is will be
run synchronously. So that's the simplest way of registering a message handler.
You've got another way which is very similar to the... what we've done with
event dispatcher component. We've got a `MessageSubscriberInterface`. What you
can say is that, you can say, well, this handler or this class will actually
handle this message and this message and this message. You can handle multiple
messages and you can also give it a very interesting configuration. So you can
say that, so the syntax it's a bit different, it's using yield instead of
returning an array. There is a technical reason to that. But the main thing here
is the return value or the value of your yield can be a set of options. So you
can actually say instead of the default `__invoke` method, you can say, well,
let's call the `doSomething()` method. You can also configure the bus. So if
you've got multiple buses, for most of, you are using an event bus or command
bus or this kind of stuff, you can actually say, well, this subscriber or this
handler will actually just listen for this message on this specific bus. You've
got a notion of priority as well, etc.

An interesting point of using the `yield` instead of returning just an array is
that we can yield multiple time the exact same key. So we can have multiple
configuration for the same message, so we can actually, we can have the handler
be called twice if we use the same message key but different priorities as the
value. It's very interesting. Anyway, I'm not sure you're going to use that all
the time, but that's another way to register the message subscribers.

Now, very important, by default, every single message is handled synchronously.
And I think it's a pattern that starts to be, we can start to see if you've
watched or listened to the Symfony Mailer, announcement from Fabien a few
conferences ago, actually the idea would be that the more and more we ship the
more, maybe, you would just be able to dispatch a message into the bus. And that
would be a sort of a nice way to provide extension points from library
perspective: here's all the messages you can dispatch and that is going to do
the job.

## Async & Transports

Now if you actually have a message handler that is a bit too slow - and that is
the main use case. If it is too slow, what you can say is that either you wait.
Or you can, well you can configure this message bus to say, I'm going to route
this message somewhere else. And to this message you're going to route it to a
transport. So how are you going to do it?

Well, first thing you're going to create a transport. So transport is something
that can be created from a DSN or URL here. If you.... Actually, who here is
actually is using Doctrine? I was expecting more people. So half of the room
actually admit they use Doctrine. So the idea is simple: you've got a database
URL in Doctrine, so you can configure whether it's MySQL or Postgres database.
Here is the same example. The transport: you give it a name, you give it the
name rabbit, it's completely up to you give it the name you want, and the main
configuration is the DSN. When you create the transport, you run the same code
it's going to be as slow as before. Because creating a transport doesn't do
anything. What *does* something is when you route the message... explicitly
routing a message to a specific transport. So here you say your
`App\Message\YourMessage` is going to be routed to the `rabbit` transport. Here
I just put the actual class, but it can be a parent class or an interface, that
works as well, so we can route a specific set of messages. And if you rerun the
application, wow! Now super fast.

## Workers: Handling the Async Messages

The only problem is... Nothing happened, obviously. Because this message has
been sent to a queue, in this example RabbitMQ queue, but we didn't consume it,
so we didn't run the actual handlers. And that's the fifth point, which is just
the worker. The worker look like a... just a Symfony command. So you run
`bin/console messenger:consume-messages` into another machine, on another
machine, and this will listen for messages and consume them. That's a
long-running process. It's got a bunch of options if you want it to be... to
auto kill after like X time or X hours, etc. But that's the main thing you need
to do. And here your handlers are going to be called for every single message.
And that's it.

That's the one on one of the Symfony Messenger. That's how it works and that's
mostly all we want to know about it.

## What Happened During the Last 6 Months?

Now, I mentioned it's six months already, so I'm going to just do very quick
rewind of like what happened and what we changed. So, six months ago Fabien
tweeted this things like, yay! We merged it! So probably a year ago I started to
create a Symfony Message pull request. We actually broke Github because it was
too many comments. And finally somehow it got merged. Super cool. We actually
renamed it to Messenger. And the ignorant me was saying: "yay, job's done!".
Turns out that actually that was just the beginning of like a lot of work.

So since then, so what happened? Very quickly, um, we only had 33 issues. Super
cool eh? It worked super well. But we actually had 150 merged pull request in
the component. And if you mention the, sort of, not necessarily the pain but the
complexity of getting pull requests and the review process and the Symfony code
base. That's a lot of work. Out of these pull requests and other kind of side
commits, more than a hundred of them were actually for the preparation of 4.1
release. And 91 commits for the release 4.2. Probably out of this 90, 91, is
probably 50 of Nicolas because he destroyed the component recently.

But the good thing is it's actually coming from 21 different contributors and
thank you very much to them. That is the beauty of the... Thank you. That's
really the beauty of Symfony I guess is that everybody's just here to make it
better. Super cool.

## Improved in 4.2

So as of a few days ago, it's actually already 100,000 plus downloads. So it's
actually been getting used. And now we are actually starting to have much more
small tweaks and small features on this component. And I'm going to present
three different things that have changed or have been improved in the version
4.2, the new one.

## Envelopes

The first one is the notion of envelopes. So I didn't talk about it yet and this
is the best moment to introduce it. The notion of the envelope is to say, well,
you've got your message, we didn't want to put any pressure on your side - do
anything special with the messages... although, when we started to go
asynchronous, there is a lot of things actually we... that can go wrong and
there is a lot of information we actually want to carry around.

Well it's the same as the beautiful analogy of the postal message. You wrap that
into an envelope and you put a bunch of stamps. That's the exact same thing. We
have envelopes within Symfony. It's a PHP object, it contains a message and
guess what? It has some stamps. So before there were actually named Items,
that's the main thing that we actually renamed to make sure that the analogy
makes sense for everybody, but it's really that. So we can mark the envelope
with a bunch of information. Built-in within the component, we've got three
stamps. We've got the `ReceivedStamp`, we've got the `SentStamp` and we've got
the `HandledStamp`. So we can actually know when we get an envelope, we can know
whether the message has been received from a transport or actually we just sent
it to a transport, or we actually, the handler has been called. That's the main,
the really important information.

So how we are going to use it? I'm going to show you the... so that's the
message, the interface, the main interface you're using. You're going to use
every day - the `MessageBusInterface`. That actually changed in the 4.2. So the
main thing is you can dispatch a message. But actually, if you look at the
param, param PHP doc, you can dispatch a message or an `Envelope`. So you can
yourself, on your side, create... wrap your own message into an `Envelope` and
add a bunch of stamps, if you want to. But main thing is it actually returns an
`Envelope` now. Before, it was actually returning the return value of the actual
message handler. Um, and that guy, Nicolas did... So basically he said, well:
Symfony Messenger sounds very interesting, I'm gonna actually look at it. And he
found a bunch of things to fix. And one of them was actually a long, long, long,
long, long discussion over multiple weeks: how can we manage this return value
of the message bus. Beginning was a mixed return value. That sort of made no
sense because it's super unpredictable, when you dispatch a message, you don't
know what's going to come back. And that is a real problem. So then, we decided
to say, okay, let's return the `Envelope`. And that kind of makes sense.

Now by doing that, what you can do is when you dispatch the message, you get the
`Envelope` back. And on the `Envelope`, you can get a, you've got a bunch of
methods. And the one that is used here is the `last()` method. So you can get
the last stamp of a specific type. So first, obviously it implies that we can
have multiple stamps of the same type.

But the first example is *if* the last stamp of type `SentStamp` is not null,
well it actually means that we sent the message to the transport. Just right now
when we called `$bus->dispatch()`, it has been sent to a transport. And each
stamp contains a bunch of information. The second example is that we actually
use the same same thing: we get the last stamp of type `HandledStamp`. But you
actually get the stamp into a `$handled` variable. And what we can say is that,
well, the message has been handled. So a message handler has just been called.
And we can, on the stamp, call the `getResult()` method to get much more
information. So we can get the result of the handler. Actually, because multiple
handlers can be called, you can have multiple `HandledStamp` and then you can
get all the results. Same idea for if you send it to multiple transport at the
same time, you can have multiple `SentStamp`. So that's how you would use the
`Envelope` within your code base.

That's the advanced track. So the stamps themselves, they are automatically
added, they are added by the Symfony Messenger - the few of them that I've
mentioned. Now, you can very easily create your own. So you can just create a
class, implement the empty `StampInterface` and create your own thing. So you
can carry much more information if you want to have a unique identifier for that
message, then you can carry it across your application. That you can do. If you
want to carry the user context, you can completely do that outside of the
message. You can carry much more information.

And the very important thing is the stamps will survive the serialization. So
that's, so if when you start to do asynchronous processing, that's where the
real troubles comes in, is the serialization process. So your message, your php
class, when it's within the php memory, super cool, we can put whatever we want.
Now, if we want it to be consumed by another php binary somewhere else, we need
to serialize it. We need to put it in a way so that we can put it somewhere as a
string and deserialize it into the other php process. So these stamps are going
via the serialization process.

So very quickly, that's the detail of the AMQP transport, the default
serialization, or the way your message is actually sent, is using the Symfony
Serializer using the JSON format.

***TIP

In Symfony 4.3, the default serialization has been changed to use PHP's native
serialize() and unserialize() function. It *is* still possible to serialize to
JSON, but the new default serialization is much easier to use for most
use-cases.

***

So that is how a message would look like within the RabbitMQ. This payload is
literally just your message. So within the payload of... which is in the message
of RabbitMQ is just your thing with no constraints. No Symfony, neither php
related things. So what it means that you can very easily use the messenger to
also communicate with the other types of systems, other language, other
frameworks. Now, to carry a bit more information, we use the headers. So we
basically put the type of the header to add the type header, to say, well that
is the type of message, so we can deserialize the right class. And also we
can... another set of headers for each stamp. That's how we carry this
information.

### Middleware

Very, very related: the second concept is the concept of middleware. And
actually there's a moment where I said that the message bus itself, the class
`MessageBus`, which is the only implementation of the `MessageBusInterface`, is
I think it's 10 lines of useful code. The rest is just comments and stuff like
that. The *real* logic of every single bus actually comes from the middleware. I
cannot say middlewares because in english we don't say that, it's just
middleware, there is no plural apparently. So that's the middleware, multiple of
them. You may have seen this kind of stuff. Um, that's the graph that comes from
CakePHP that describes how they use middleware, the middleware mechanism to
handle the request and the response, the HTTP request and response. The idea is
very simple: every single middleware is in a stack. So there is an ordered list
of middleware and each middleware is responsible for calling the next one. And
they can do specific actions before or after. The Symfony Messenger bus as the
exact same way of working.

So there is our middleware, the Symfony Messenger one, and there are two main
important middleware. The two or... the two in the middle. The first one, which
is basically because... the one in the middle, the last of the stack, so the
last to be called is the `HandleMessageMiddleware`. So that's the middleware
that's responsible of saying: okay, I got this message, I am going to find out
which handlers are registered for this message and call them.

There is a middleware above that, which is `SendMessageMiddleware`. And this
middleware is responsible for sending the message to a transport. So if this
middleware decide that, well I need to send the message to a transport, it won't
call the handle message middleware: it will just return the `Envelope`. So the
entire behavior of a message bus is actually within the middleware.

And you can create your own. So in order to create your own, that's just a class
that implements a `MiddlewareInterface`. It has just one method. I kind of
mentioned it: it takes as an input an `Envelope`, and as an output another
`Envelope`.

It also takes the stack as - the middleware stack as an argument so that it can
call the next one. So an empty middleware that's is completely transparent, will
look like that. The main thing is return stack: from the stack and get the next
middleware, and call the handle method again. I just call the next middleware.
And here, I can actually add some behavior, I can add some things, things that
are going to happen before and some things that are going to have to happen
after. After what? Or before what? After and before the next middleware. And if
you look at the previous slide, it actually means it is going to be running
something before or after either the message has been sent or the message has
been handled.... your message handler has been called. Yep?

So, I'm gonna show you an example middleware, and I'll walk you through. The
main thing here is that line at the bottom, which return is, `return
$stack->next()->handle()`. That's super important: as a middleware, I need to
call the next ones. Here, the concept is to say: well, I can create a middleware
that's responsible of auditing what are the commands or what are the messages
that went through. So the first thing that I'm going to do is to get the message
from the envelope. So that's your php class. Get the message so that after, I
can display a bunch of messages that contains the class of the message. So it's
going to say: well, either I received the message or I started to dispatch a
message. To get the class at the end of the line, just get the class of the
message. Now there is one thing that if - and you've seen that already with the
two other stamps, but it's the same mechanism here - if from the envelope we've
got the stamp that says `ReceivedStamp`, it actually means that we are in the
consumer. We are under, `bin/console messenger:consume-messages` side. So in the
logs, I won't put a "started with message", I would just put, like, "received
message" because that's the context.

So when you look at your log a few days later or something, the future you will
actually thank you very much - that you didn't put the same message and that you
don't know when things have happened. We don't know exactly, um, what was the
context of the message execution. Because the middleware they were called when
we received the message from the transports or, when we actually dispatched it
from the web application. Because the messages all the time go from the bus.

Now, there is this notion of stamps and this custom stamp. So here is a simple
example where you can create your custom stamp - here I have named it
`AuditStamp`. It will just contain an identifier. So we can actually stick an
identifier to this given specific envelope, and you can stick this identifier so
that in your log you can correlate the log that have happened on the web server:
when you created the message, when you dispatch and send the message and the
logs that are happening in the CLI, in the background process, because you're
going to have the same identifier for this specific message. Because this stamp,
if it doesn't exist - that's the first line - if the `AuditStamp` does not
exist, you add the stamp on the envelope. And because these stamps are like,
carried around with the serialization process, you're going to have your
`AuditStamp` when you consume the message. And that's this middleware. So this
notion of stamps and envelopes - coupled with the middleware - in my perspective
are super, super valuable because we can do such things.

When you create your own middleware, well the only thing you need to do is to
register it. So you need to explicitly say this bus is going to have this
middleware. This example is saying you can create multiple buses, this is just
an example of creating multiple buses. And here one of them that we called
`command_bus` will have this `AuditMiddleware` that we've created. So every
message that we dispatch into this `command_bus`, will go through our new
`AuditMiddleware`. while the rest, or if the message is going to the
`event_bus`, won't go through this middleware. That's an example a bit more
complex because we've got multiple buses, but imagine and replace, just remove
the `event_bus` and that's the normal behavior of just having one bus in your
Symfony application.

Now there's just a small clarification: the stack. Because that's the main thing
as well that's changed in the `MiddlewareInterface`. Before we had a sort of
magic second parameter. Instead of the stack, there was "next", so we could just
call `$next()` parenthesis and call the next middleware. Well the problem was,
by doing that, the next, the function that was behind the next variable was
actually within the message bus class. So what it meant is that the stack trace
was super, super horrible. And if you're using a profiler or something like
that, you will always see the middleware going back to the message bus, going
back to the next middleware, etc, etc. That way it's actually much more explicit
as well: you get the stack and you get directly the next middleware within your
own middleware. So you just go: `$nextMiddleware->handle()` and that's it. It
directly called the next middleware. The stack trace is very good or very simple
when you get exceptions. And that's very useful.

So that's the sort of, um, details of what's changed and how, about the
middleware.

## Transports

Now, the last thing I want to talk about quickly are the transports. The first
thing, again, and I think it's really important to reiterate that, is that you
don't need transports to benefit from the Messenger component. By itself, the
structure that it offers you is really interesting for you to use. So we don't
need transports. And I think it's been built in a way so that we can have very
nice or an easy migration path. As in, we do synchronous as much as possible.
The reason is, asynchronous, for a lot of reasons, including the fact that you
might have inconsistencies between when you run the message and when the message
has been dispatched, with the fact that you may have something that actually
needs to return the value from the synchronous background process. We need to
have sort of like asynchronous notifications via email or whatever. There is
much more complexity.

So you can start synchronous. And *then* you can configure the transport and
then you can write a message to be asynchronous. That is sort of like the great
path, as has been planned. So the transport, we have a built-in AMQP transport.
So if you've got, the um, the only requirement is the AMQP php extension. If
you've got that, you can actually directly use RabbitMQ, and that's it.

Now there is an amazing php library is called Enqueue , which has 1+ transports.
So if you're using AWS, SQS transport, if you are using google, you've got
Google Pub/Sub, so using more exotic things like Kafka, you can actually use
them. Even just a file as well, this kind of stuff. There is an integration with
Enqueue there is an adapter so that you can directly create using the
EnqueueBundle all these transports and use them directly in the messenger. So
you can directly route the messages to an Enqueue transport. So it means that
already right now you've got dozens of actually transport you can use.

One thing that we changed, which is very important for you if you want to create
transports, is that we simplified the serialization. Before we had the notion of
encoder and decoder. Now it's very simple. You've got just one class, that is
responsible of serializing and unserializing the messages. You can, so I
mentioned by default, using Symfony Serializer, if you want to use protobuf, if
you want to use csv, you can easily create a class that implements the
`SerializerInterface` and roll your own.

## What Now? What's Next?

So what about now? So now the Symfony Messenger component, it's still
experimental in 4.2. So what it means is that maybe in 4.3 we're going to change
things as well. So the idea is to say for new components, because we - and
especially this one which touches more than the scope of just the php process -
there was potentially things we didn't see coming. And maybe there are design or
structure in the component that we need to tweak to make sure you actually
benefit for everybody. I think it's very unlikely, but it might change from 4.2
to 4.3. It doesn't mean that it is not usable, it just means that if you, when
you will upgrade from 4.2 to 4.3, you might have some slight things to change.

It is much better than before. And really I think the massive work of the
community has been great in a way that: it's super easier. The error messages
are like, make much more sense, and you've got this notion of stamps that
actually to me seems very, very simple to grasp.

Now, the documentation deserves a lot more love. It's a super hard problem,
especially for components moving super fast like this one. If you feel you can
help, that's an amazing contribution to do, and it will help a lot of people. So
on that. Thank you very much.
