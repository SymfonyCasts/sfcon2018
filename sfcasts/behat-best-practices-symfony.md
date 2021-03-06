# Behat Best Practices with Symfony

***TIP
SymfonyCon 2018 Presentation by [Ciaran McNulty](https://github.com/ciaranmcnulty)
Behat is widely used as part of a Behavior Driven Development lifecycle, but
it's also widely misused.

In this talk I'll explain the BDD process, and show the best practices for
using Behat including: writing good scenarios, driving service development from
scenarios, fast UI testing, and using Behat and the Symfony2Extension.
***

Hi everyone. Welcome. Let's get started. Maybe we can close the doors. Hi. I'm
Ciaran, and the subject of this talk is Behat and the best practices around
Behat. I'm a consultant around areas like agile, quality, communication in
teams being productive and most of the time the thing I introduced to
organizations that helps them the most is this thing called behavior driven
development.

## Business Problems vs Technical Solutions

So generally in software we start with some kind of problem maybe we express
the problem as a project goal or maybe you express the problem as an epic.
We've got some sort of business problem that we want to solve and we caught
some ideas about how we're going to solve these problems and policies we're
going to put into place some things we're going to let users do that they
couldn't do before, some things we're going to stop people from doing.

We have we, we generate some kind of business rules. And then at some point we
implement technical solutions that implement those business rules or enable
those, those new policies and we can launch the product and real people can
start using it. And as we go through this progression from that side to that
side, when you're looking at it that way, the cost gets higher. So the cost of
each of these things increases. Having a ... identifying a business problem is
cheap. Coming up with some ideas for how we solve it. It's relatively cheap. We
have to talk to people and then technical solutions and building stuff is
expensive. So shifting left is about maybe trying a lot of BDD practice is
about trying to test these ideas earlier, testing before we commit them into
code. So a lot of it is about communication, so it's called behavior driven
development, Behat is a behavior driven development tool. The name comes from
behavior testing, but that's not documented anywhere that I know about. So it's
a secret for you.

## BDD: Talking about what the System should do

So, what's behavior Liz Keogh says BDD is when you use examples in
conversations to illustrate behavior. Just about having a conversation. Number
one, difficult for developers sometimes. So we need people in the teams who can
enable this kind of process and help break down some barriers. We need to talk
about behavior or define behavior in a minute. What, what should the system do,
you know, not, not the technical details, how should it behave in different
situations? We use concrete examples. We give very specific examples of how it
should behave in different cases or talk about why that's important.

## Examples

So what's an example? An example is something that illustrates a rule. So maybe
the important thing is understanding business rules, but the idea is we give
examples as a very efficient way of making sure everyone understands them the
same way. So naturally in a conversation if you don't understand something, to
ask for an example, a concrete real example. So, we've got these business rules
we're going to implement and we're going to illustrate them with examples.
Examples are going to help us understand the business rules, help us check that
we all are aligned on the same business rules. They have some value in
themselves, but mostly it's to make sure we all, we're all talking about the
same thing.

So here's a business rule. We charge our customers sales at a tax rate of 20
percent. Quite common in different countries to do this. Put your hand up if
you understand this rule. Someone wrote that in a user story, you'd kind of
have no further questions, you can just implement it. So, but there's an
argument that sometimes a business rule is so obvious that we don't have to
really have a conversation. But an example is a really cheap way, even with a
simple rule of just validating, yes, we understand. We really did understand
it. So I might ask the developer who's from the US and they say, yeah, I
understand this completely. So something's price is $10, we charge $10 and we
had $2 tax. So we take $12 from the customer.

Put your hands up people if that's what you thought it meant, not everyone,
right? So even in a simple business rule, we can have a misunderstanding about
how it works. We'll come back to that. So the person on the left has, has this
idea in their head, this rule, and if they just expressed the rule, they
explain the algorithm or the general policy that covers lots of cases it's
possible for the person on the right to have a different, entirely valid
understanding of the same rule.

If instead they give concrete examples. So this is the rule and that means in
this situation, this is what will happen. It gives the other person, an
opportunity to kind of run those concrete examples through their own
understanding and say, that's not what I thought would happen in those
situations. Maybe my model's wrong. Maybe I understand your model. And the best
thing that can happen is then we have two way communication. They say, okay, so
in this situation, this, you mean this is what will happen in this situation.
So both this is a kind of back and forth, but both parties look at the concrete
cases and they say those concrete cases really match with the way I understood
this thing we're talking about. So that means I think the thing in my head is
the same as the thing in your head. We can't validate that directly without
telepathy, but concrete cases are giving us a way to sort of check this.

So, in my sales tax example, if the developer gives the example, I can say,
actually, no, that's not what I meant. What I meant is if something's priced at
10 pounds, we charge 10 pounds. We just remember that some of it was tax
because in the business where we're operating in the UK, all prices have to be
inclusive of sales tax, not, not exclusive of sales tax. And I gave this
example in another country and they had a different system where some stores
showed it inclusive and some stores, some stalls showed it exclusive. So I
built the system this way and invested my time and effort in the code
implementing this business rule. Or maybe it would only find out when someone
tests test the software, does some user acceptance testing because maybe my QA
team had the same understanding I did. We can validate that earlier just by
giving an example, give us an opportunity much earlier in the process to, to
catch these understanding mismatches or, I'd say requirements mismatches. But
we're not on the same wavelength.

So examples are about behavior and you can kind of, um, a really sort of
computer sciencey way of looking at behavior is there's an input and an output.
So something happens and there's an outcome, there's an action, something that
causes behavior, there's an outcome that's the result of the behavior. I buy a
pair of Levi 501's and I'm charged 32.99 in and out, right? So, then maybe
there's something missing here. It's not obvious why that action leads to that
outcome. It's not really obvious why that product leads to that price.

We need this other thing called context. Something that happened in the past is
a good way of thinking about context. You can express context as like a state,
the state the system is in. It can be really useful to think about context as
what are the things that happened in the past that are affecting this outcome,
especially if you're doing event driven stuff. So what happened in the past
that means I'm charged 32.99. What happened was that they were listed it in the
system. So someone administrator loaded that product into the system and gave
it a price, and now that means when I purchase them, I get charged this money.
Those three things are really all you need. There's some sort of triggering
event. Triggering action is what should happen and there's, you know, the
minimum necessary context to understand why that causes that. So when you're
talking about examples, it's quite easy to capture them in, in something like
this format, you know, little notes when you're talking about things.

## Examples Lifecycle

And examples go through a kind of lifecycle. Early on when we sort of starting
to think about our business rules, we can use examples as a way of discovering
potential rules as a way of being quite concrete about the system. Coming up
with different options, diverging, brainstorming. We can come up with lots of
examples of how the system could operate. At some point we're going to start
narrowing down which things are we actually going to implement, which things
are we going to implement first? Then the examples get more detailed so early
on in their lifecycle. You can think of an example like the guy goes into the
store and buys a pair of jeans and it gets charged the right price. We just
write that down or even something shorter, you know, Dan North came up with the
Friends episode naming where it's the one where he buys the jeans and you can
just sort of stick on a sticky note and keep it somewhere later on in their
lifecycle when we're closer to implementing them technically we write them out
in more detail and as we'll see, we're going to use them with Behat. We're
going to write them out in a formalized way that the machine can read. But you
know, think of that as an, as an evolution of an example as it becomes more
likely that we're going to implement that example in our, in our project.

## Context Questioning

So once you've got some examples, you can enrich them by asking a couple of
angles of questions that Liz Keogh, again named in a blog post. I come back, I
come back to this naming because it's a good reminder when you're having a
conversation. Have I asked about the context, so is that, is there another
context where this action leads to a different outcome?

So if I've got this example here, I can say, is there any situation where I
could buy these jeans and I wouldn't pay 32.99? That's a great open question.
In fact, that's not a very good open question. It should be what are the
situations where I would buy these jeans and pay a different amount? Don't give
an opportunity for no. What situations are there? So as soon as you start
having this conversation with a real person or a real group of people and
they're thinking about, you know, because it's an example, we've got a real
real product, a real amount of cash, and they think about the real situation.
They're not thinking about specification land. They're thinking about a real
store and then they can come up with loads of stuff. Yeah, maybe they're on
sale, maybe the person buying them is a member of staff and they get the
discount up to a certain amount. Maybe the jeans were damaged, and they're
refunded to stock. So in those cases, what are the different outcomes? This is
a whole discovery process of coming up with new examples, new requirements we
didn't know we had and it's great that we're discovering them now rather than
after delivery.

## Outcome Questioning

The other kind of approach Liz advocates is outcome questioning. So we've got
the context, we've got the action, we've got an outcome. Are there other
outcomes we have to think about? So in this Levi's example, apart from me being
charged some money, what else happens? Again, this is where the flood of future
user stories, future requirements sort of cracks open the can of worms opens.
I'm charged 32.99 and I get a pair of these jeans sent to me, dispatched
through the packing system and the warehouse, has to know that we've got fewer
jeans than we had last a minute ago and probably a load of other stuff, you
know, an invoice gets generated and the marketing stats get updated and blah
blah, blah, blah, blah. We discover the huge scope that we didn't know existed.
This is the discovery phase is where you're having these conversations. So
you're trying to do this early in, in a very informal way.

## Example Mapping

And one way of having these conversations that I do recommend as a starting
point is example mapping. It's a sort of workshoppy format involves sticky
notes on a wall. It's easy to get people doing it rather than sitting everyone
sitting around typing things into a Google docs. Or God forbid, a Jira ticket
on a big screen, around everyone sits around the table. It's good to get people
moving and doing stuff. So Matt invented an example mapping three or four years
ago and blogged about it. There's lots of other ways of doing this kind of
workshop, but this is a great formula. So you take the feature you're talking
about and you stick it on the wall. In this case, it's calculating the price of
a shopping basket. We've realized that customers like to know the price before
they pay. So we're going to talk about that feature and you try and figure out
what are the business rules and you lay them out in this horizontal dimension.

So we have to apply sales tax, above a certain amount you get free delivery and
then as you're going through the rules, you try and think of really concrete
examples of each rule. How many examples do we need to give for someone to
understand how the rule works? If you have VAT being applied, I can probably
come up with an example. If you buy this specific product, this is how much you
pay in tax and maybe don't need any other examples. I don't need to generate
lots of different examples because it's a simple rule, free delivery above a
threshold. If you pay 50 pounds, you don't pay any shipping. If you pay 10
pounds, you pay some amount. I don't have to put all the detail when you're
sticking these on the wall, but kind of identifying which examples we're going
to have to go into detail on later. We might have some questions. So someone in
this conversation says like, what if you're shipping to Belgium and our stores
is in the United Kingdom? What happens then? We can capture the question as
part of the example map and actually what happens, what happens with tax and
what happens with delivery. We don't know.

If someone knows the answer for delivery, they can say, okay, overseas shipping
is a fixed rate in each country and we don't have this concept of a maximum
amount where you get free shipping. But maybe we end the session with a
question, how does tax work across borders? Maybe we have to ask someone else
who isn't in this session before we can proceed as a good example mapping.

I spend about 15 to 20 minutes talking about a feature. More than that then
you're probably going into more detail than you need to. The questions at the
end are, can we start work on it without this answer? Maybe that's something we
trust we'll find out during the sprint, you know, or do we need to resolve it
first? Do we need to get an answer before we say we can work on it. More
interestingly, can we do part of this feature that doesn't involve the question
and a nice way to split is against these rules so we can potentially take this
overseas shipping and move it into a new feature.

I'm going to say, okay, well we can build the shopping basket pricing. It'll
only work in the UK and if you select the different country, we just don't show
you the total. We'll do that next week. This can be a really useful way of you
know splitting down stories. Example mapping is one technique. You can do it
one on one it's good to do it in a, in a group. I quite like doing it. If
you've got a few features to talk about I quite like putting them physically in
different places in a room and people rotate around them in a round robin. It
should be quick. It should be scrappy. You're not getting into loads of detail
on the examples. You're just kind of identifying hey we need to have an example
where someone gets free shipping.

## Feature Mapping

There are other techniques that are worth looking at. Feature mapping is very
similar to example mapping, but it adds a kind of time line concept. So you
have business rules, examples and then steps in the process. And of course if
you're doing event storming, they don't do event storming? If you're doing
event storming off of the output of that as well as being super useful. You can
zoom in on sections around the commands and say, given these events in the past
when this command happens, these new events should happen. You can sort of
extract scenarios from that.

## BDD is not about Testing

So let's talk about BDD. That's some of the most important stuff in bdd is
discovery. That's where you should start if you're implementing BDD. It's not
really about testing, it's also not about one way conversations. I'm not
suggesting that your product owner does this kind of example mapping session,
identifies a load of examples and then types them into Jira and then delivers
it to a team, who's never seen them before. It's about having multiple
stakeholders inside those conversations.

## Validating Examples

But where it comes into testing is that once you've got these examples, and we
generated them so everyone understood what we're building, they are super
useful for validating features. Once we've built the feature, we've now got a
set of concrete examples of how the system should behave. Even if we're doing
manual testing. That's really useful. I know that if I buy an item that costs
10 pounds, I should be charged two pounds sales tax. I can go into the system
and buy something that costs 10 pounds and I check, I get charged two pounds
sales tax. So once we started to build the technical solution, we're going to
write tests and these examples are going to be very useful to create test cases.

This is where Behat fits in. We've had a lot of rich conversation conversations
about what we're going to build. I'm going to write some automated tests and
there's a, there's an overlap there where a tool that can take written examples
and help you automate them into test is going to add value. First tool like
this was called cucumber in Ruby. Every language has a tool like this. We tend
to call them cucumbers, Behat is sort of retaining it's own identity because
it's a completely separate code base. It was never like a port, but we
collaborate a lot with the other cucumber tools or we make sure their syntaxes
are interoperable and stuff like that.

## Real-World Examples

So I wrote out an example. You don't need to be able to read this. I write out
an example in a format called Gherkin because it's from cucumber. There is the
same format we were capturing on the cards. You know, the three steps, the
context, the action, and the outcome. It's just written out in a way that is
machine parseable, but it's meant to be business readable. So a human can read
it as if it was a document.

Let's zoom in. The problem I'm talking about is scheduling training. I do a lot
of training and either we have a training course with not enough people, so we
need to cancel it or we have too many people. So as a trainer I should be able
to cancel courses or schedule new ones and I can identify the business rules.
This stuff at the top of the feature file is just human readable text to
explain to a person.

So the rules are, when I make a course, it has size limits, there's a sort of
threshold where the course becomes viable, and when we reach a maximum class
size, you can't sign up to that course anymore. And then I come up with some
examples and I write them out. The `Background` is context that's common to all
of the examples in the file. So we talk about context using the `Given` keyword:

> Given "BDD for Beginners" was proposed with a class size of 2 to 3 people

So that's the context. The first example is, a course doesn't get enough
enrollments so it's not viable. When only Alice enrolls in the course, then
this course won't be viable, easy to understand. Hopefully.

This is an example where the course gets enough enrollments. So Alice is
already enrolled in the course, that's the context. When Bob enrolls, that's
the action. Then the course becomes viable, the course will be viable because
it was proposed to the size of two to three. And enrollments have stopped when
the class size is reached. Given that Alice, Bob and Charlie have already
enrolled in the course, when Derek tries to enroll in the course, he can't
enroll.

It costs something to write them out in this detailed format and think about
how much detail you're gonna include. So you have a kind of scrappy
conversation about examples. When you're going to build the system soon you
write them out in this kind of `Given`, `When`, `Then` format and you show them
to the person you had the conversation with and says, this is what we talked
about. So whoever learned the most in the conversation can write out these
scenarios, so this is what we talked about in the room. Please validate that.
Let's just double check. You don't do this, you don't write them out in this
detail in advance for everything.

## How Behat Executes Examples

The job of Behat is to take these multiple feature files which have steps - a
step is: Given, When, Then - and somehow execute them as tests. And yeah,
that's hard to execute as a test. It needs some help. So the help is that we
have a bunch of classes called contexts with methods in, called step
definitions. Each one of these step definitions tells Behat when you see a line
like this in the feature file, this is how you test it.

In the Gherkin file, Given a thing happens to Ciaran and then in the PHP file
I'll write a method with a doc block that has a pattern in it. Then Behat will
know, whenever I see a line like that, I should execute that function. And
you'll see the function is parameterized. So given a thing happens to Bob,
we'll also hit the same function. And I have to write the test code inside that
function that's going to check the system.

## Testing through UI vs Domain Model

The way a lot of users interact with your system is through a UI so it can be
tempting to test through the UI. It's normally a much better idea to test
through a domain model.

So why is that? Testing through the UI is going to be slow, you have to
automate a browser, things break, web servers time out, that kind of stuff. And
if I just test through the UI, all that tells me is that the UI supports all
the business rules. That means the business rules could be living inside a
controller or the business rules could be living inside a Twig template because
we didn't put the button on that template and that means people can't do that
thing. We probably want to check that the domain model is supporting these
things. So let's see how that looks.

## Testing Domain Models

So by driving PHP objects directly from my scenario, it's going to prove my php
objects, implement the business rules correctly. It's going to give a bit of a
pressure to use the same naming from the feature file in the objects because
I'm looking at both at the same time. And I'm not going to pick a different
word when I can use the same word. And it's gonna run fast. It's going to run
almost as fast as your unit tests because it's just running PHP. So when I
execute that feature file in Behat, you'll see they all get this nice, maybe it
hasn't come across, nice yellow color and maybe you see at the bottom
everything is undefined. So Behat is telling me, "you need to tell me how to
test all of these steps.".

So I go through my feature, I start at the top, here, I look at the first one,
this one. Given BDD for beginners was proposed with a class size of two to
three people and then I write a method in my context class annotated with
something that's going to match that pattern. And I think, what do I want this
to look like? This is where I start, this is the TDD like loop I go through.
I'm going to start here. So there's a thing called a course. So I'll make an
object, I'll pretend there's a class called a course. I'm going to give it a
constructor called `propose()` because the word propose was in the example I
wrote. And it's proposed with a class size of between a minimum and a maximum.
So, I kind of create, I guess course is an entity and class size is like a
value object.

And I want to execute it. I get an error because none of these classes exist. I
kind of started here, I'm using the domain language from the example in my code
and I like to see the red.

If if the first time you run the tests, it's green, there's a lot of things you
could have done wrong, so it's always good to run the test and see them red and
then you write the code that makes them go from red to green.

So, I have to do some stuff here. It's a short talk so you have to make these
objects and you probably write some unit tests for those objects as well. And
you know if you're super advanced you write the unit tests first and all that
good stuff, but at some point I've written a course object that has a
constructor and I've written a class size object that has a constructor and I
did some extra thinking about what do I need to check inside class sizes. Maybe
I'd check that max is bigger than the min. So, there's extra stuff I have to
think about as I'm building these objects, as soon as they exist Behat starts
passing.

So, the next step only Alice enrolls in this course. In the previous step, I
sort of stored the course that I was creating as a member variable in the
context. The next step, Alice enrolls in this course, I'm going to enroll a
learner on a course. Um, there's an example here of a transformation. So Behat
does support very simple transformations from strings to value objects,
especially if we're going to be using the same sort of values in a lot of your
examples. So whenever I see a name, a learner, I'm going to turn the string
into a learner object. The course, gets an enroll method. I have to then add an
enroll method to the course that takes a learner object. I have to create a
learner value object, write unit tests, blah, blah, blah. Then this step will
start to pass.

Then the course will not be viable. So I need to write a step that checks that
and I'm using an assertion here, I'm using a PHP assertion. Any step that
doesn't throw an exception is considered a pass. So in the Then steps you have
to check something, `Then the course will not be viable`. I have to tell Behat
this is how you check the course won't be viable. You can use a different
assertion library. You can use the PHPUnit assertion library, that has a bit of
extra support than Behat because we know how to get the better error message
out of those kinds of exceptions. Or you can have an if statement that throws
your own exception. It is kind of up to you. So I'm asserting this is viable
method that returns false. I have to then write some code, some logic when I've
done all of that, my first scenario is passing. And actually that's quite a
good point to commit.

I built all the objects and implemented one of the scenarios and working per
scenario is actually very nice as a developer. It's a good way of not trying to
think of everything at once. So what's next? I do all of the other scenarios, I
work my way through in the same sort of way. So you can kind of imagine Alice
is already enrolled in the course. I call the existing enroll method. Bob
enrolls in the course I call the existing enroll method. The course will be
viable. I assert something slightly different. I assert it's not viable. And,
you know, the third scenario, Derek tries to enroll in the course. Maybe I
catch an exception, then he should not be able to enroll. I maybe check the
exception got thrown, or I check the course doesn't have Derek as a as an
attendee. So I validated all of this inside my objects.

That doesn't take long. If you look at the execution time and the screenshot,
maybe that's a lie. It's twenty it's still the same 20 milliseconds to run
those three scenarios. That's not unrealistic. They're like large unit tests.
There aren't, there's no infrastructure involved, nothing crazy is happening.
We're just checking that these objects have the business rules in them and
that's a super powerful place to start. Actually talking about what the
business rules are, is a really powerful place to start. But checking that the
object objects implement them on their, on their own and it's not, you know, in
a stored procedure or a controller or somewhere else is really going to help
when you have to write a command line, a cron job that also enrolls people, for
instance.

## Testing Services with Behat

The next layer to look at is the service layer. And I'm not specifically
talking about Symfony services. Quite often you have some layer in between your
domain model and your user interface, and in the Symfony case it is Symfony
services. So when, you probably don't want to be interacting directly with
these domain objects too much in your controllers you want to delegate it into
services.

So, we, we might want to inject our services into the Behat context. As soon as
you go into a Symfony app and you start thinking about your Symfony services
as. So indeed, you would try and have your own concept of application services
and not tie it into the Symfony service container, but a lot of apps, it is
tied into the Symfony service container. So it's really convenient to have a
way of accessing our services our defined services from inside Behat. And then
trying to use the services rather than using the domain model directly. This is
going to make sure that our service layer supports all of the use cases we want
to support.

Even though the domain model is quite complex, we can have a defined set of use
cases. These are the things that our system supports. And you can test the same
scenarios in multiple ways using Behat, using a concept called suites. So we
wrote one context that told Behat how to test every step directly against the
objects. You can define a separate suite that tells Behat how to test the same
steps against a different layer of your application, in this case services.

So it looks slightly different. It's the same process for me when I want to
implement this first step, BDD for Beginners was proposed with a class size of
two to three people. Instead of creating, instead of creating domain objects
directly, I'm making calls to services. So I'm saying course enrollments,
propose a course with a string name and the minimum and maximum attendees. So
I'm kind of assuming that course enrollments is available. There's a service
called the course enrollment service.

## Symfony2Extension

Where's that come from? You sort of see it, it's arriving magically in the
constructor. And this is then becomes framework specific. How is Behat
accessing that service, the Symfony 2 extension, which needs to be renamed but
it was a really good idea at the time to call it that, will inject your Symfony
services. So, it takes care of finding your kernel, bootstrapping the kernel,
telling the kernel that it's in the test environment. You can override these
things. And then when we, instead of just listing the the object, instead of
just giving the name of our service, our context object, we also can inject
param, inject services as arguments to that context. So in this case I'm saying
there's an argument, there's an argument called course enrollments in the
constructor and this is the service you should get from our container. I'm very
old fashioned so I'm not using the class name yet.

And this can be done in other, in other frameworks using different extensions.
And now that PSR containers are standardized, there is an extension that will
just let you directly plug any container in including the Symfony container.
But it's best to use the Symfony extension for this still because it will do
things like rebooting the kernel in between steps.

So I need to write a service and the service. This is, this is the service
object inside it, it's using the domain objects I created earlier. So it's a
way of defining a sort of course API for my domain model that the controller is
going to use. The line in the controller is going to end up looking like this.
It's just going to call a method on a service with some primitives, with some
scalers. At this point you kind of need to get infrastructure involved. You
definitely need to think about persistence. Using real infrastructure is super
slow. Using real database is slow. So it's best to use fake implementations or
infrastructure, like an in memory database that can lower your confidence
because you think I'm not testing with a real infrastructure you can use a
thing called contract tests to make sure your fake infrastructure behaves the
same way as your real infrastructure to kind of mitigate that.

So I run through my other steps when only Alice enrolls in this course. So when
enroll Alice, the course will not be viable. I'm not looking at the domain
object directly. I'm calling a method on the course enrollment service, so
they're kind of two different options. I wouldn't recommend testing the same
scenario against the domain objects and the service layer, but something I
often do is I will write my tests directly against the domain objects and then
I'll switch and start testing against the service that wraps those domain
objects, if that makes sense.

So I kind of deliberately tests at different layers because I want to be able
to completely change my domain model without the tests failing. I want to
decide, course isn't an entity anymore and actually it's all about learners
enrolling in tracks and that doesn't need to be visible at the service layer.
That doesn't need to involve any changes at the service layer.

## Testing the UI with MinkExtension

At some point you want to test the user interface, but not too much. Don't
start by testing the user interface. At the moment the best way of doing this
is the `behat/minkextension`. Mink is a API for driving lots of different
browser automation tools using the same common API.

We interact with the domain model via the UI and this checks, once we've got
service layer that works, we're basically checking that the controllers are
calling the right services. Think about it as just just validating that the UI
works. Don't do this kind of stuff. If you start with the UI layer, you'll end
up putting loads of UI crap in the, in the Behat scenarios, in the Gherkin
scenarios. It's not the end of the world, but you're missing a lot of
opportunities to, you can't have this conversation early. And it's very
brittle, when you change the UI these things just break.

So to test, using mink, I enable the Mink extension. I already had the Symfony
extension, I can tell it use the Symfony driver. This is gonna automate as if
we're doing browser automation, but it doesn't require a real web server. It's
just booting the kernel and sending a request and checking the response. You
can also use it with PSR there's a PSR 7 driver they made. So it looks kind of
the same. Again BDD for Beginners was proposed with the class size of two to
three people, I'm still going to inject a service and I'm still gonna call that
service, but maybe this is the service that's connected to the real database,
the real, real test database, the real MySQL test database.

When only Alice enrolls in this course, then I need to write some browser
automation code. So Behat checks Alice enrolling on a course in a different
way. Instead of calling PHP methods, it's going to open a page and fill in a
form and submit the form. The course not being viable, instead of checking a
boolean on a, on an object, we go to the page and we say, does this element,
this CSS element exist?

Now I don't need to do this for all of the scenarios. I'm kind of confident
that if I've got 10 scenarios about different situations with different
combinations of people sign up and then unregister then register again, and
I've checked all of that at the service layer, I probably only need to check a
few of them through the UI. Specifically the ones where the UI would do
something different. So I need to check one case where the not viable warning
appears. I need to check one case where the, this is oversubscribed thing
appears I don't need to check all of the combinations of what could happen
because I've got confidence that the system, the underlying system works. I'm
just checking that the UI is plugged into the services correctly. So this can
all be tested through Symfony directly. It runs on your laptop, it doesn't need
any extra infrastructure.

## Testing JavaScript

Some people automate real browsers, mostly when you've got loads of JavaScript
and you've written a site that doesn't work without JavaScript and you have a
frontend team who aren't doing any testing any automated testing. So it's very
slow to automate a real browser. They crash and do weird stuff.

It's often better to use Cucumber.js and do a sort of layered approach like
I've been describing in the JS application. It's better to do that then try and
drive the entire system end-to-end with a real browser because that's how you
end up with four hour test builds.

The best way I've got at the moment of doing this is using the chrome driver.
You can pass some extra stuff to tell it to run headless. This is actually an
experimental extension by this person, who I don't know, that seems to work
fast. It talks to chrome across the debug port, it's faster than selenium for
me, but it's a bit flakier.

And we used to have a lot of systems like PhantomJS that would pretend to be a
browser. But if you've got a recent Chrome, you've got a good browser
automation tool already. You can run Chrome in headless mode, you can open up a
debug port and Behat can talk to it directly. You don't need to install
selenium if you don't want to. Maybe into CI and stuff you can use selenium,
but this will work directly and it's in the background and you don't really
notice it's running or you know, the docker world now so you can just get a
docker image that has a headless Chrome in it and it boosts itself and then you
can kill it afterwards. So browser automation is much less painful than it used
to be.

## Summary

So start by, start by having conversations, try driving the domain objects
directly. Refactor them into services when you feel like the domain objects
have got to a good place and test with the service layer. Then test a few
scenarios through the UI. If you can't avoid javascript entirely, it's good
advice for life.

## Future: Autowiring, Panther

So future: currently you can't autowire services into your Behat contexts. The
API is there inside Behat for that to work. We just need someone, some brave
person to make it work in the Symfony extension. I don't think it's impossible,
there's stuff there in Behat for figuring out what to inject for specific
argument, someone needs to hook that into autowiring in the Symfony container.
So if anyone wants to volunteer afterwards talk to me.

Symfony Panther is awesome. Behat uses a thing called Mink, which is an API
that's common to all of these different browser testing tools. But in Symfony
land we already have a common API now across browserkit, Goutte and panther.
It'd be really nice to have a way of like a convenient way of using that
directly inside your Behat contexts and use the web test API that you've maybe
already know instead of having to learn Mink. And there's a lot of convergence
across cucumber tools so it's possible that it's possible, Behat will become
more like the other cucumber tools by sharing code. There's a project to maybe
switch a lot of it over to a shared go run time that we haven't signed up for.
But a lot of the other cucumbers are doing it. So we might succumb to peer
pressure.

Thanks for your attention. I went slightly over but we started late. So I think
that's okay. I'm Ciarran. If you want to talk about any more of this stuff just
grab me. I can come and help you do it at your company. I look after another
tool called PHPSpec, which is unit testing. Check that out. There's two meetups
I'm involved in look at them and if you want to see a working example of the
code, I was showing in the slides. That's the URL on GitHub. Probably don't
have time for questions, so enjoy the break and come and talk to me. Thanks.
