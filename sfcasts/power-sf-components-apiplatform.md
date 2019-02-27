# Leverage the power of Symfony components within ApiPlatform

***TIP
SymfonyCon 2018 Presentation by [Antoine Bluchet](https://github.com/soyuka)

ApiPlatform is great for building API-first software. Even better, it's build on
top of Symfony! Did you hear about Symfony's Workflow component? Or the brand
new Messenger component? They offer powerful tools!

For example, a pizza order can take multiple statuses, each one in transition
with another (order, pay, wait, eat). This is a workflow.

While waiting for the product, it needs to be cooked! Every time the order is
paid, we have to send an instruction to the kitchen. Today, why don't we
automatically push that message?

Through a real-life use case, I will demonstrate how well these two fit on top
of our Symfony-based API!
***

Hello! Welcome to my talk about leveraging the power of Symfony within API
Platform.

## Hey soyuka (Antoine)!

So first thing first I'm going to introduce myself. So I'm Antoine Bluchet also
known as soyuka online. And I'm core contributor to API Platform, but also PM2,
which is a Node.js Process management software. And I'm developing or coding in
many languages, php, Node.js, also a bit of Rust. And I play Rocket League.

## What is API Platform?

Um, so which one you guys don't... doesn't know about API Platform here in this
room? Doesn't know. Okay. Yeah, that's good, a few hands. So "API Platform is
the most advanced API platform in any framework or language". So this is a quote
of Fabien Potencier from last year at the SymfonyCon.

So what is actually API Platform? It's basically a set of tools and these tools
are given to you, and give you the ability to build an API in really a few
minutes.

So for example, yeah, we have create, remove, update, delete resource
management. Uh, we have embedded filtering, sorting, we have the validation that
comes from the symfony/validator component. We have pagination, authentication,
for example, with the JSON Web Tokens or OAuth and things like this. In these
tools are also available the security. So when you think about an API, you don't
really think about security and you don't have to care about it because we did
all the work. We prepared this for you. What's important is that we are using
this JSON Linked Data format by default, but you can use any format that you
want CSV, XML, everything that the symfony/serializer component gives you. API
Platform has Hypermedia capabilities. I'm going back to this in a minute. And we
also have GraphQL support, cache abilities with Varnish for example.

Hypermedia capabilities means that we can generate things for you. For example,
this Open API, it used to be called SwaggerUI. Um, we have also this, uh, admin
panel you can put on API Platform, you mount your API, then you just run a few
commands and you have an admin built out of React Admin. We have code
generators, for example, for React, also React Native since a few weeks. Uh,
Angular interfaces for typescript. And it's production ready. Uh, the, there are
lots of companies, I'm sure you're aware of because you're using it, that are
using API Platform in production. And it does really work well.

For this to work, we have a Docker, a docker container that's all ready for you.
You just have to clone the repository launch it and you're ready for production.

So API Platform is also open source. So this is really, really great. Today, so
I've combined, we have two repositories, we have the core one and API platform
with, which is a bit front page. I combined them both and we have about 2,900
commits, 140 contributors. So thank you very much for this. Uh, about 3000 stars
and we just reached a million downloads a few weeks ago. So I was checking like
two days ago on the Packagist website and yeah, we just reached a million
downloads on the core repository. So this is a really great milestone.

## How API Platform Benefits from the Symfony Ecosystem

And now about this talk, it's the thing is with API Platform is that it benefits
from the Symfony ecosystem. So what does this mean? Actually? It means that
because it's built in a Symfony way, you can use every component you want within
API platform and really they combine with each other really, really in a simple
manner.

So basically now for those who didn't know about API platform before, I hope
that's what you're thinking right now and yeah, I just advise you to check API
Platform and just try to use it. If you don't like it, it's okay. But really I'm
sure you will.

## Building our Pizza Ordering App

So now let's order pizza because we all like it. To order pizza usually you take
your phone, I don't know, you give a call, whatever, you order. Then the pizza
gets prepared and then it gets delivered. So this is what I call a workflow. And
then about this workflow is when the user just sent out the command to call, to
order for a pizza, we need to notify the kitchen and say: "hey, prepare my pizza
please". The kitchen prepares the pizza and then it sends a message back. okay,
my pizza is ready, just now we need to deliver it. Now there is a second message
that goes through the delivery truck, for example, and says, okay, my pizza
needs to get to my client.

So, um, to do this, we're going to base our project on `symfony/skeleton`. So
this is what I did here. So I don't know if you heard about Symfony4, and I
insist on the 4 because we have now Flex recipes. What are Flex recipes? These
are kind of like Composer plugins that the third line here, the `composer require api`
is in fact, um, launching a Flex recipe that will install API
Platform, but also the doctrine-bundle, CORS support - Cross Origin Resource
Management - and everything you need to basically have an API started. So these
three steps, and I'm done, I've created an API. All I need to do is add some
stuff. So really you could have started the API Platform with our skeleton. So
check it out on Github it's the api-platform/api-platform project. But I prefer
to use the `symfony/skeleton` just to show you guys how things go together.

So now that we have API, I'm going to start by installing the Workflow
component. So I don't know if you guys assisted to this, the talk tomorrow, this
morning about Workflow. So yeah, basically this is a quote from the docs. And...
so, yea: just require it on top of our current API. So really nothing really new
here. Um, next thing we're going to use the Messenger component. He also, we had
a beautiful talk this morning by Sam about the Messenger component. And I
actually stole a slide from him, just like this afternoon. And yeah, we are
going to use really basic functionalities because all we need to do is send a
message to the kitchen and to the delivery truck. So we have a message, we have
the bus and the bus will manage the, the to send the message and then through
our transport, handlers will get this message. I will get back to this.

Yea, easy: `composer require symfony/messenger`. I remind you, I'm in a
`symfony/skeleton` project. I installed `api`, `workflow` and `messenger`. So
now, um, if I want to order pizza, I thought about it and I come, came out with
this graph here. So the first step on the top left is just create an order. To
do so we're just doing a POST on our API. Then the second step, when there is
this small message Emoji, it's actually a message that we are sending from our
API to the kitchen in the bottom. When the pizza is done, the kitchen will
answer: "okay, API, I'm done, I'm ready". The pizza is ready: `PATCH /orders/prepare`.
The fourth step is, then again, I'm sending a message to the
delivery truck. And when the order has been delivered, the delivery truck will
answer.

So let's do this. Um, just to remind, yea, this is the stack. So Symfony, API
Platform. I'm using RabbitMQ for the transport of my queues. Um, and SQLite
because I'm a lazy guy, I didn't want to set up a PostgreSQL database.

## Creating the Order Entity & ApiResource

So to do the first step, it's the easiest one. All we have to do is create an
entity. So in `src/Entity/Order.php`. basically most of you guys I hope know
about the annotation there. So we have basic Doctrine annotations and the most
important one on the top of the screen here is the `@ApiResource`. In my Order,
it's really simple: for now I just have a `status`. It was, yeah, you would have
many more fields like what pizza we want, and what the are products, etc.

And yeah, it's done. I have an API: I have my four routes. So API Platform
automatically creates these when I create my entity. I have GET on the
collection, then I can create a new one, gets an item that exists, update it or
delete it.

## Workflow and Messenger

So far so good. Now, the workflow. So to do the workflow, I hope you've seen the
conference this morning, it was really interesting. I want, I will not get into
details, but basically all you have to do is, to set up this configuration. What
is really great is when we did the `composer require symfony/workflow`, we have
a recipe behind it and it preconfigured everything: this file existed. I just
had to go in there and add my things. So I've added a few places and a few
transitions. Uh, the cancel one isn't really important, but yeah, just to show
you guys how easy this was.

And yeah, that's nice with the workflow as, I don't remember the speaker name
from this morning, but he showed this command as well. And he said something
really interesting is that: when you do this, you can actually see what you
previous configuration gives. And I did this many, many times and I'm, I
actually was okay: I will rename things because transitions were, had bad names,
etc. And this is the result of the previous command. So we can see we have a
first state is the order being ordered, then the preparation, order get... is
prepare. We deliver it and it's been delivered.

Now let's talk about the messages. So to do this: same thing, all we have to do
is just prepare this configuration file. On top of it you can see that I used,
yeah, I put out the environment variable that I'm using below and I'm specifying
an amqp queue in RabbitMQ. We, the small thing I added there is the serializer
configuration path, where I said: okay, I'm in an API, I actually want you to
serialize things with JSON-LD. The rest is just like in the documentation. And
to picture it, well I have two transports: one is my kitchen. So you really need
to picture this transport as different physically places. The kitchen is really
a kitchen and the delivery truck is a delivery truck and I chose to route my two
messages - the preparation message and the deliver message - to these two
transports.

Now that we have this, we can create our command but we still need to send this
message to the kitchen and say: Hey, I have an order now, please prepare my
pizza". To do this, I've chosen to add a Doctrine listener. So I added the
annotation there: `@EntityListeners` `OrderCreateListener`. So every time an
order gets created with a POST, on POST on my API, I'm just able to do things.
And here, I'm just dispatching the message: `PrepareOrderMessage`. So I injected
the `MessageBusInterface` and on `postPersist` I am saying dispatch my
`PrepareOrderMessage`.

So, this `PrepareOrderMessage` is just a wrapper around my `Order`. So, um, uh,
this morning, Samuel told us about the Messenger component and new things that's
evolved since the 4.1 version of Symfony. And so they might be, it may be a bit
simplified here. And next step: we have this Order handler. So this is actually
my kitchen. The kitchen will get messages in this `__invoke()` function, it will
prepare the pizza, and when it's done, it will just send an order via the API
route with a `PATCH`.

Let's do this actually. We don't have this `PATCH /orders/prepare` route. We
need to create it. So with API platform it's really easy to add a new routes. So
if you remember, at first I had four one's: one to get the collection, items, to
delete and to update. So I'm specifying `itemOperations` with my previous routes
and I'm giving a new one called `status` where I say: okay, this has a `PATCH`
method. And in the path I have my order, the id of this order, and the
transition I want to apply. And the last one, the most Important one, the
controller class.

And this is how my controller is. Um, it's a Symfony controller. I hope you all
know how it looks like. I've injected my workflow `Registry` and whenever I get
a `PATCH` on this new route, I'm saying: okay, get me the workflow please. And
just apply the transition I got in my arguments. Also I added some noise there
with the HttpException.

So we can now patch our orders. The kitchen can say: okay, I have done, just do
the prepare transition because you're done.

Now we are going to do the exact same thing with the delivery. And, sorry, what,
what's it... what is really good here is that we used a workflow to picture our
process. So what comes with the workflow is actually a lot of events that again,
we use behind it. And this is exactly what I'm going to do now. I'm going to
say, okay, whenever the pizza has been prepared, please send the message for the
delivery truck and say: okay, the pizza is done. Now you have to deliver. So
this is a listener, or can't be more simpler. And at the bottom you can see that
the subscribe events I say: `workflow.order.transition.prepare` call
`onPrepare`. And the `onPrepare` will just get back to order from the event and
we will use this bus to dispatch this new message: please deliver my command.

So then again, exact same thing. This is my delivery truck, this is where I'm
going to deliver my pizza. Drive a bit, get to the client and say, okay, here's
your pizza. And when it's done, I can get back to my API and say, okay, I'm
done. I'm done delivering.

## Live Demo

This was step five. So now I'm going to show you guys how this all works. If I
can find my screen. Perfect. Nice. So, as I said before, I actually have an
AMQP, a RabbitMQ service. So here you can see on the right, bottom right screen,
I have a RabbitMQ service that's actually running right now. And on the top left
of the screen I'm going to, uh, I want you to picture a kitchen. So the top left
of the screen is the kitchen and I'm going to say: okay, consume message. And
yes, please consume the message of my kitchen transport here. So previously I
routed the message, `PrepareOrderMessage`, and I said: okay, this message has to
go to the kitchen. So this is my kitchen. And on the right side of the screen
I'm going to start my delivery truck. Now, I'm just going to create an order. So
if you remind, I had only one field, it was `status` order. I'm hope you can see
alright. I can maybe increase the size a bit.

Yeah. So I'm using this tool `httpie`: it's just curl: `http POST` on my API
with new status `order`. So status order is the first step of my workflow. So
let's go and you can see the others done it. It got created the kitchen prepared
it and now the delivery truck is delivering it and he said, okay, I delivered.
So, I'm going to start this again with a few commands so that you can see
everything in action. So I've added, in my handlers, I've set up a sleep time
out with dots. And when it's ready, I just patched the order on my API.

Okay, so that just raises further questions indeed. Because here, I really
wanted to show you guys how easily things come together when we use Symfony and
API Platform. The thing is, um, we could have used the messenger and a lot more
configuration possible. And it goes the same with the workflow: it can be really
much more complicated than this. What I actually don't really like about what I
did here is to use transitions through the API. But yeah, this kind of things
can surely be improved. Um, also about the transport, I used here RabbitMQ on
the messenger transport. Uh, I've actually worked on the, on a new transport
with redis. And uh, I've opened up a pull request on Symfony. So yeah, lots of
thing that can be improved in this workflow.

## What's Next?

Um, actually what comes next now is that within API platform also, we will have
the Mercure support. So I don't know if you guys know about Mercure yet, but
it's something Kévin Dunglas worked on that, that gives you the ability to
receive live events when you update a resource. So for example, here, when I
updated my pizza, I could have received a live event in the kitchen just by
adding one simple configuration on my annotation, meaning `mercure` true. In API
Platform we also have, it's, it's being prepared, it's the MongoDB support. And
we also have people working on Elasticsearch read support within API Platform.
So lots and lots of new stuff. We work everyday on API Platform to provide you
with the best tools possible.

So yeah, this was it. So I want to thank the SymfonyCon for having me here. Uh,
obviously, uh, everyone who contributed to API Platform because I really learned
a lot by just reading you guys and working with you. Uh, also, Les-Tilleuls,
which will be hiring me in February. And yeah, this talk will be available on
GitHub soon, uh, at this URL. And you can follow me on GitHub and Twitter. So
now if you have any question I will be happy to answer them.

## Questions

Does someone have a question? I think this works.

Okay, I think it works. I was wondering more about how can you customize the API
Platform because it seems like it's like this, but perhaps not.

How you can customize API platform?

I mean, is this behavior that you showed us, is it the standard behavior? Can
you do other things?

It's not a standard behavior because I used other components that comes with
Symfony. But really when you have Symfony 4, with the autoconfiguration and
everything, it's so easy: you just add the good interfaces and everything works.
So, so I don't know.

Okay.

Someone else back there? Just send it and then. It's like non-breakable mic.

Okay. Uh, for me it's questionable for the monolith application, uh, to use some
additional services like RabbitMQ and Redis for the messenger. So I'm wondering
if the same workflow which you just showed can be achieved just in the monolith
with use of just Symfony events.

So you mean working with this but with Redis on, in back?

No, without Redis and without RabbitMQ: just with internal Symfony event system,
just in a monolith system, we are working in a once call, right? So no
additional services required usually in this case.

So actually, um, when you transport the messages, you can use whatever you like.
I mean, yes, it can work, with a bit of work, you'd have to dig a bit through
the components, but yeah, I don't see why it couldn't, could not work.

Someone else. Nope. Looks... thank you for your attention.
