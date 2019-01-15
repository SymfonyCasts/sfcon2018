# Knowing your State Machines

***TIP
SymfonyCon 2018 Presentation by [Tobias Nyholm](https://github.com/nyholm)

Web development is not just about delivering a response. It is also about writing
good code. The state pattern will help you move complexity from being all over your
code to one or more state machines. This talk will introduce state machines, show
how to identify uses of them and implement them in your Symfony application in an
object oriented manner using the Symfony Workflow component. 
***

So excellent, excellent. Thank you all for being here. Um, do you know this is
a large conference and you know, when you're organizing a large conference like
this, you do want to have the best speakers, right? So, and what do you usually
do when you're having a conference? You put the best speakers upfront, like the
keynote is obviously the most impressive one and then you have the best
speakers and then they slowly decline. Um, as you know, Michelle is not here,
Michelle was about to do this talk so and her kid got sick and you can't really
leave a two year old home in Switzerland by itself. So she had to stay home. So
the conference organizers thought, Hey, what should we do? Michelle is sick. So
obviously they did the best thing they could. They call the second best
speaker, right? Unfortunately that person couldn't make it. So they called the
most knowledgeable speaker who knows the Workflow component the best, and that
person didn't make it either. Um, however, like five or six calls down the
line, they find my name and here I am and I hope I can give you. Yeah, thank
you. At least one guy or girl is very polite. I hope I could give you like a, a
talk that could match Michelle's talk.

So with uh, with that said, my talk is titled Knowing Your State Machines and I
would like to do this talk because I believe web developers only care about
printing out HTML as quick as possible. They don't care about fancy algorithms,
they don't care about design patterns. They just want to print stuff really,
really quick. And so I want to tell you about something that makes your code a
little bit more elegant. So I didn't, I didn't bring any state machines. It's
nothing I can have in my pocket. It's a state machine is just something that
hold states, like any states. It could be the state of a pull request or this
small washer on your - small lights on your washer. It could be, yeah, it could
be anything. So it's basically a way of thinking, a way to relate to problems.
And why state machines? Blah Blah. There's a lot of texts in this. If you work
at a company where your manager needs convincing, this is a slide - just to
take a photo or show him later. I'm sorry, show him or her later.

## Tobias does a lot of Open Source

Quick about me. I do plenty of work on Happyr. We are a job advert platform -
it matches people with confidence, blah blah. I do plenty of Symfony things. I
run PHP meetups in Stockholm and I do plenty of open source. I started my real
open source career in 2015 on a SymfonyCon, just like this one. The PSR-6 cache
had just came out. So me and my friends were like, hey, why don't we implement
this. Why don't we make an implementation of PSR-6. So we did. We created the
PHP-cache organization and CacheBundle and I thought open source is great. So I
started to get involved with HTTPPlug, which is kind of an abstraction over
HTTP clients and when you do HTTP clients, you do API clients and then you do
mail clients. And I even wrote like, Oh, this is how you should do API clients,
this is the boilerplate for FIG. And currently I'm maintaining both Guzzle and
Buzz. I've written my PSR-7, I've written PSR standards and stuff. I, I do
plenty of things. The most interesting library here, which I like the most is
this one, the NSA one. With NSA, you can totally violate class privacy:
accessing private properties or methods is super easy with NSA, I do also
contribute to a bunch of other libraries: these are only the most fun ones.

So how should we do this about state machine? How should we do this? I would
like to start with a lot of theory and then l will put on some examples,
examples, and then more examples. How does that sound? Excellent. Because I've
got slides for this. How I can barely see you, but how many of you, how many of
you feel like you know, state machines by heart? Excellent. It's like 12 of
you, which, which is excellent. Then I guess like the rest of you will learn
plenty. You are.. anyhow, let's get started.

## Graph Theory

Let's get: first rule of state machines. We've got to talk about the graph
theory and the first rule of graph theory is like these, these are not graphs.
These are charts, these are diagrams, these are call them whatever. They are
not graphs. A graph. It looks like this. A graph has a node. A more interesting
graph got two nodes, and nodes can be connected with edges and edges can either
be undirected like this one or it can be directed. An edge can also have a
weight. Does this seem familiar for most of you? Excellent. Excellent.

This is a full-fledged graph. This is like the cool graphs can be. And it can
represent pretty much whatever thing you like it to represent. If you slap on
some numbers on here and maybe you can have a problem, maybe this represents
cities and roads between them. Maybe it represents computers in the network,
maybe, maybe they represent people and how well they know each other. And say
we've got a graph like this and you wonder how much.... say they are computers
in the network. How much data can you transfer from a to b? This might be a
problem you're having, so this is called the maximum flow problem and there is
a solution to this. And say that you want to find the shorter distance between
a and b. You can solve this this by Birpartite breadth-first search. And say
that you want to... Say that you want to find out what's the cheapest way of
visiting all these cities, or these nodes, visiting only every node once.
That's a traveling salesman problem and there is a solution to that as well.

Or say that these are cities and you should put out some electricity grid
between them. This is the minimum spanning tree problem and there's a solution
for that as well. What I want to tell you is if you've got a problem, if you
can express this problem as a graph problem, there's already a solution to it.
So you don't, you don't really have to solve these problems yourself, you can
just solve the problem getting your problem to a graph problem. Does it make
sense? Thank you. This, what I just did, I gave you like six months of
university math of graphs. So this is basically the big graph theory I just
gave you a quick introduction of.

A cool thing about graphs is that you can reduce them. Say, consider this
graph. I want to reduce them by giving them a new rule. And say, the rule might
be: let's remove all the loops. If I remove all the loops in this graph, I get
a tree. And there's plenty of algorithms you can do in trees as well. One of
the most difficult to implement is the red then black search tree, which
basically is good way to store things if you want to search quickly in it.
Anyhow, I want to show you this. You have the big graph theory and inside of it
you have the theory of a tree. And you can continue reducing graphs with
different set of rules and you will get a workflow.

## What is a Workflow and State Machine?

A workflow also has nodes, but sorry, but we decided, oh also has nodes and it
also has edges and we decided that every edge should be directed. We also
decided that we call the nodes places and transitions. And a rule in this
workflow is them saying: a place can never be connected to another place. All
of this has to be a transition between them. I know I'm shooting a lot of rules
at you now, but do you kind of feel comfortable? Excellent. And we decided, I
mean just so it's easy to see, we draw these transitions like a square and a
good example of a workflow is this: if I'm in place A and execute the
transition t0, I'll get to place C. If I'm in place B and they execute the
transition t1, I get to place C and D. Make sense? Good. If we continue to
reduce this workflow, say we want to add rules saying that a transition could
only have one input and one output and also the names has to be unique. Then we
actually get a state machine that looks something like this. So workflow
multiple one input, multiple outputs. A state machine only can have one input
and one output, something like this. So if execute t1, if you are in B and
execute t1 you get to D. And because I'm drawing these images by hand, I will
choose to draw with this annotation. Fair?

## Real-world Workflow Examples

So right now I have been talking for eight minutes and we discussed the big
graph theory and we discussed the tree and we discussed what a workflow is and
that a state machine is just a reduced workflow.

So let's start with some examples. I mean I know this part was the boring part,
this part was the theory part. Let's start with some examples of recognize that
we recognize. This is a simple state machine. We defined three states and we
defined how we can move between the states, right? So it's easy to see that if
you are in green we can easily move to yellow and then we can easily move to
read. However though if we are in green, we cannot move to red. There's no
arrow to red. It's impossible to go direct to red, which is which is a good
thing. I've got to keep asking you simple questions until you give me some
continuous feedback. Do you understand this?

I work at a company called Happyr and we deal with a lot of job adverts.
Basically we put up adverts and we match people and people in common with soft
skills, etc, etc. Anyhow, we have a lot of job adverts. So in a business
meeting we decided like this, this is the state machine for a job advert. So we
started draft and then we can send it to review. While in review, we can trash
it, untrash it, and we can send it to review again if you'd like. And from
review you can publish and archive it. It is impossible to archive something
that is in review or that is something that's in trash. It's impossible.
Excellent.

Since you are very confident now, what is this a workflow of? Someone said a
pull request issue. Uh, I will say yes. Correct. So basically you start coding
your pull request, you submit it to Travis, Travis runs for awhile. You may
want to decide when you're waiting for review and then you may want to decide
to update it. So we'll go back to Travis running. And maybe someone says, hey,
we need some changes. So you go back to coding, then you update it, then send
it to review and hopefully it gets merged or else it could be rejected and
re-opened again. Seem straightforward?

So how about this? Oh yeah, pull request. So how about this? What is it the
workflow of? What is the state machine of? People usually yell out like, like
you, sorry, like you did. You said doors. I'm like no, no, not doors, it's
elevator doors. But you're equally correct. It's doors. So basically this is
the four states that a set of doors can be in. Either the doors can be opened
and then you press the close button and then be closing. While the closing we
can press open button so they go to opening. It's impossible for doors to be
open and then in one, the next instance be closed. They have to be closing,
right?

## Moore & Mealy State Machine Implementations

I think. I think we kind of know, know state machines now. Maybe business a
dictates that... Maybe business a business dictates that you should have a form
like this, like maybe a sign up form or whatever. So you can have a state
machine to know where the user is in the form. So if you leave this process,
you can, you can get back to where where he or she left off. And if your extra
nice you can have back buttons so we can travel back and forth in this process.
If you want to implement this, there's basically two implementations. I'm gonna
can talk to them with them bot. No, I'm not going to talk *with* them. I'm
gonna talk with you *about* the both of them - the implementations. The first
implementation is the state pattern and that's using the Moore state machine.
And a Moore state machine only cares about the current state. The next state
only cares about, the next state is defined by the current state. The other
state machine is the Mealy state machine and the next state depends on the
current state and some input.

## Moore State Machine Implementation

I just want to say this out loud now and then I'm gonna repeat it later. Let's
start with the state pattern and the Moore state machine. This is basically
what you easily read on wikipedia. You... a state. I'll click this. Say you
want to implement, something like this. We want to implement a... something
that reminds you to complete your profile, say a user signs up and you want to
say like "Hey, you should add your name" and we remind them twice about adding
their name. If in order to add your name, like we should, hey you should update
your image or you should add your resume. So implementing this with a state
pattern means that each of these five states have a class and that class itself
only have one f unction. It has a Send function. And, and that calls itself
decides what to do and where to go next. Makes sense? So the idea here is that
we have, have, we have a worker like a cron job saying like: Hey, give me all
the state machine from database somehow and start yourself. I don't really care
what you're doing - you just send the email, send the correct emails.

So in the implementing this, I'm basically looping over all the states and if
the, if the state machine says "I'm done", then we're done. So any given state
just looks something like this. I, this is, I mean, you send an email and then
I say go to next state. Makes sense? We should obviously do a little, we should
modify this a little bit. We can maybe say like, if the user doesn't have a
name, if the user *do* have a name, go to add your image state. So the state
itself says where to go next. The sates itself decides everything what to do.
And the state is a single class. We should also be a little bit more friendly.
This would actually flood the user's email address, so uh, email inbox. So we
should: you should be in this state for at least seven days until we send
another email.

I know this might feel weird. I know this might feel like that's not really
PHP. It feels more like Java or something else. It feels weird and I will agree
with you. This feels a bit weird. Using this and, for the simple state machine
I showed you the five states, you have a bunch of classes like this. And they
all pretty much look the same. Each of this state just look, oh, they look like
this.

So I... with this I would like you to get an idea how the state pattern works
and some of you guys are nodding politely. Excellent. You obviously have issues
with this. I mean you have issues: how do you store the state machines in this
database? How do you keep track of the state? How do you store the createdAt
entity... or data. Also it feels kind of weird that you have one class with
state.

## Mealy State Machine Implementation

So let's, let's do something different. Like now we know the Moore state
machine, we know the state pattern, so let's look at something different. Let's
look at something that feels more like PHP. And I'm going to stress this
multiple times: the Mealy state machine, the next state depends on the current
state and some input. So it's obviously a little bit more flexible. Let's try.
Let's consider this state machine again, you all know this by heart now. If you
would write an implementation of this, if we like PHP developers would write an
implementation of this, how would we do? Well, what's the first? We obviously
need a class, right? We need a state machine - traffic light state machine
class. And we kind of want to have a function that applies new state. And I
imagine it to be good to have like a function that "can I apply this state?"
Good. So let's. So I create my state machine looking something like this. I
define all my states as constants and I also choose to have a private property
called state. And then I just, via a switch statement say: if someone say: "Can
I do transition to yellow?" I check if I'm currently in state green and someone
says "can (I go) to yellow?" Then sure, that's allowed. If I'm in state red and
someone says "to green", then it's obviously no, I return false. And I do a
similar, very similar thing for apply. I just do something like this.

So would this state machine. I can easily, I can easily implement my... This,
this class is easily implement my traffic light state machine, right? There's
an issue though: this state machine has a private state. So I need to store the
state machine in the database somehow. This state machine is only for one
traffic light. If I have multiple traffic lights in my system, I have to have
multiple state machines which each needs to be stored in the database, because
of the state property. What if I made my state machines more stateless? Like I
removed the private state and put that on the, on the traffic light entity? So
I did something like this. Can this *specific* traffic light do this
transition? So now I don't need to store the state machine in the database
anymore. I can use this state machine for all the traffic lights.

It feels quite, feels quite good, right? Yes. It feels quite good. I diddothe
same thing with apply function. I just say `$trafficLight->getCurrentState()`
and I have a `$trafficLight->setState()`. Very similar, very small changes. And
I feel pretty good about the state machine at this point and whenever I'm using
this, it might be an idea of mine to have a function that allows more traffic.
I mean if, if, if the state is currently in the red state and I want to allow
more traffic, I just, whatever state they're currently in, I just move it
towards green. So I might attempt to do something like this. So I know if I
call allow more traffic twice. I will definitely be in green.

## Making the State Machine Generic

You look confused and you should. Because this was like my trickery, but I
didn't trick you. If I want to make this state machine more generic, more
reusable, obviously this is a horrible idea. Right? So if I'm going to make it
more reusable, I may want to consider instead of having the states all over the
place, I may want to put them in a, uh, in a property like this. So with this
weird array looking thing, it's actually me trying to be a bit clever again. So
I can just say: can I do the transition? I was checking if this is, if this
property is set, if this array key exists. I do the same kind of wizardry in
apply. Do you feel confident with this? You kind of understand it at least
that's, that's the most important thing. However, say you want to make this
even more generic. I mean, I shouldn't, I mean this is only for traffic lights.
I said, traffic light here. What if I do something different? What if I... Oh,
sorry. Oh, look, the states, it should be states here. What if I inject the
states instead in a constructor like this?

Then I have those states - I can do this for any kind of state machine and any
kind of traffic light. And now, oh no, I have traffic lights over here. What if
I did something like this to have the `StateAwareInterface` instead? And this
`StateAwareInterface` just have a `getCurrentState()` and `setState()` method.
With this class of 15 lines. This is actually a pretty good, pretty generic
state machine, right? And it all feels (like) PHP, all in one class and I can
configure it however like and use an object that I like. I hope we feel like we
understand this. Does anyone have... oh no, I can't ask you for issues. Does
everyone feel like you understand this class? Excellent. Excellent.

## Hello Workflow Component

I'll tell you a little bit of a secret. This is actually how the Workflow
component works. This is the workflow of the Workflow component. And it was
very true like two or three years ago, this was exactly the Workflow component.
And nowadays there's a lot of edge cases, but in principle it works exactly
like this. The Workflow component all have some more functions we showed. I
showed an implementation of "can we do the transition?" And "apply transition".
The workflow component has two more methods saying "like what state are we
currently in?" And what are my valid transitions?

## Workflow Component Example

So let's show some examples. Same old job advert workflow again. In the
Workflow component, this might look like something like this. The red circle is
the current state, so I can say: give me the state machine for job adverts. And
I say give me the current marking for this entity and give me the places. And
marking here sounds weird. It's because it's basically the abstraction of
getting properties from the, from the Advert. If you ignore that, it's, this is
very simple, right? So I get an array back and I see I'm in Draft. I can
continue showing examples saying: state machine, can I do publish? No, I can't,
but can I do "to review"? Yes, that's true I can. So I continue. I go to review
and when I execute this line, it actually moves the circle away and from here I
can say go to trash, go to draft and go to review again. And in this state I
can say which are my enabled translations, and I get an array back to say I can
go to publish and trash.

I just, you just understand everything in the Workflow component. The Workflow
component itself got like 15 classes and this is the most complex one. And you
understand it fully now, which I think is quite... not impressive that you
understand it, but it's impressive how simple Symfony components are. Uh, this
is also, since this is Symfony, this is also available in Twig. Imagine,
imagine when you're editing an advert on Happyr, imagine it looks exactly like,
like Wordpress. You have this big text box in the middle and then you have a
sidebar with like trash, publish and stuff button like this. This is the code
for the sidebar. So I'm checking if the workflow "can" go to review, then show
the link for review. If the, the, the, this advert can be published, then show
the link for published. So the same sidebar will look differently on different
adverts. And if you are, if you're smart with the naming of your work for
transitions and you're naming of the routes, you can do something like this to
print the sidebar. I want to show this because it's, it's possible, but you
probably shouldn't do this. I do want to say that... I mean, I think this is
pretty cool, right? You, sorry.

I need to reverse, blah blah, this is a sidebar, decisions... This is pretty
cool, right? Your sidebar really, it changes depending on the states. And
what's cool here though is on a business meeting, we sat down and someone
asked, how come you can't trash something that's in draft? It's obviously no
arrow here, but what's the reason? And I'm like, yeah, there is no real reason.
So let's, add this arrow. And here's, here's the kicker. I only changed the
whiteboard. I want to change how the, how the state machine looks like. I did
not change any of my models. I did not change any of my controller. I did not
change any of my services, I did not change Twig layer, I didn't change
anything but my configuration. And still with this change, I will see the trash
in the sidebar when the advert is in draft. And that's pretty cool. A year ago,
I did this presentation and when I said that's pretty cool. People like, yeah,
that's cool. Good job, and they were kind of cheering. But you become the
words, not only to be excited with me.

So I want to change the configuration And the configuration looked like this.
It's super simple, This is my, this is my core of my business. This is my
business logic. If you're really, really paying attention, you might spot a
weird thing here. I told you that a workflow can have, no sorry, I told you the
the state machine only can have one input and one output. And clearly I have
two inputs here. This Symfony configuration is smart enough to know like, oh, I
can make two transition of this. So it, it says, it looks like only one, but
it's actually gonna to become two transitions.

## Multi-Step Signup Form Example

And also quickly want to show you a multistep signup form. Business may dictate
that this is the way we sign up to our service. So have an intro, you add your
name, you add your email, you add your twitter, and then you're done.

And doing this workflow in the configuration is simple, you just do it like
this. You just say that this is a state machine, this is my places, and this is
how you move between places. And business might say this is good and all, but
we want to have back buttons. We want to have a workflow like this where you go
back. And we said, sure, no worries. We just do something like this. However,
this is not valid YAML, right? You cannot have multiple keys on the same level
which is called `back`. So what we need to do, we need to do something like
this instead. We add arbitrary numbers or arbitrary names and then we have the
name. So this how we get around the issue with YAML. So with this you can
actually please the business to have back buttons.

## Workflows: Modeling a Process

And I know I've been talking a lot about state machines now and I haven't
really... This is called the Workflow component. And it says workflow like all
over the place here, right? So what is the difference between a state machine
and workflow? In the beginning I mentioned the state machine is a reduced
workflow, but what is the actual, what is the actual difference? As an example,
I want to show you this: this an example of a workflow. Previously we've only
seen state machines. This is the example of a workflow. And the main difference
in a workflow and a state machine is that in a workflow, you may be in two
places at once. It might feel weird, but it's actually valid. A workflow more
or less describes a process. A state machine describes traffic lights, when you
actually go back to the beginning and stuff - how you move. So this is a
workflow for a blog post: it's first a draft, then you request reviews and then
you get in, you get in the places for waiting for spellchecker and waiting for
the journalist at the same time. So there's an example where a transition that
makes you to be in two places at once. And spellchecker is probably automatic,
right? So that's pretty quick, but you have to wait for the journalist. And it
might be tempting to try to publish this blog post right now. But you can't
execute the publishe transition because both its inputs are not fulfilled: we
don't have one of the inputs, right? So we have to wait for the journalists to
catch up and then you can eventually publish it.

I will also say a workflow rarely contains loops. It's rare that you go back in
your workflow. However, state machines, loops are encouraged... kind of. People
used to ask me: when should I use workflow when I use state machine? And the
answer is boring. It's obviously: it depends. I tend to use workflow when
there's a clear process. I use state machine for everything else. It's just how
I like it. The most important thing is that you know which one you're using.
You need to know how to relate to them. It's like, for 99 percent of the
problems, it doesn't really matter which one you're using. But you should know
which one you're using. Does it make sense?

## Workflow Events

Excellent. Excellent. So back to the Workflow component. If you have been
around in the Symfony community for awhile, you might notice Symfony have like
30 components and the components are very decoupled from each other, but they
play really, really well with each other. And the secret there is events.
Events is the thing that makes them play well with each other. And the workflow
component is no exception. We have events or, the Workflow component has events
for everything. We, all the events looks like this: you have a generic event,
say `workflow.leave`. Then you have an event, say workflow - the job advert -
when they leave. And you have events for, workflow, job_advert, leave, draft
node.

And all these, you have all these three types of events for everything. So we
have workflow leave, when you're leaving a node. You have transition that would
actually transition from one node to another. And when you enter the node, when
you have been entered the node, and when a transition is completed. And then
you have announce saying: Hey, I'm all done. It might feel weird that we have
so many events. I mean these add up to like 21 events for each transition,
which is a lot of events. However, there, there's a reason for all of them.
Like, I know I've reviewed all the pull requests adding more events and they
all makes sense in this specific weird edge case. You should only care about
the one you care about. I mean, you shouldn't... I only use like leave and
enter... and that's fine.

## Guard Events

So there's plenty of events. There's also this even more events. There's one
event called the Guard event and the Guard event is something that you use for,
adding extra logic, like external dependencies. So: can I publish this blog
post? Like say we want to, we want to have a rule saying: you can only publish
this blog post if it's between3 and 4 PM. Or if the database is online or if
the manager is in the office or if you have the role `ROLE_PUBLISHER`. So a
Guard, a Guard event is how you add this extra knowledge to your state machine.
And I think you all are comfortable with events subscribers. I basically say at
the bottom, I say: this is the event I'm listening to -
`workflow.job_advert.guard.publish` - like the "got published" transition.
Whenever that event is dispatch called the `guardPublish()` function and run
this three lines in the middle. I would, I will assume that you are comfortable
with this.

## Normal Event Listeners - e.g. Logging

Except for the Guard event, we may, we can register normal event listeners. Say
you want to log every time you move from one node to another. So I say, the
workflow, when the job advert leaves any node - `workflow.job_advert.leave` -
then I just to do a logging. I say advert with ID performed transition from one
node to another. It might be useful to have this kind of log message.

## More Workflow Examples

I'm checking - you're still with me? It looks like most of you are with me.
I... no surprise: I love the Workflow Component. I use the Workflow component
all over the place. If I have... the Symfony Security components have the
concept of voters, which are basically: if someone wants to watch, view an
advert, I got to make sure that advert is published. And I'm using the Workflow
component to make sure I am in state `published`. So here's a simple security
voter and I say: if someone tries to view then I check this: if the state
machine got the marking for the advert and marking is published, like if I'm in
published state, then you're allowed to view it. You should not be allowed to
view something that's in draft. So I used my state machine all over the place.

I also use the state machine for domain events. I know for sure that no advert
in my application will ever be published unless the
`workflow.job_advert.enter.published` event is dispatched. I can trust that for
sure. That's why I use this event for my domain knowledge. Like say business
say: you should email all your users whenever an advert is published. And this
is a good, is a good place to hook into this. So I never dispatch any events on
my own, I just use the workflow events. Obviously we don't email all our users
when adverts are published, I just wanted to say, but it's, an example how you
use your domain logic with these events?

## Rendering the Workflow as an Image

Um, whenever, whenever I'm in business meetings and we, the business people say
we need this and we need that and we gotta figure out a new way to do stuff. We
write the workflow on the whiteboard. And we change our minds, we go back and
we add new states. We, because business people, they tend to like graphs and
images, they tend to like, big circles and squares and arrows between them. So
it's easy to work with, work with a whiteboard. However, when the whiteboard
meeting is done, I want to make sure, I go back to coding and I got to make
sure that I have configured this properly. So I add my configuration and then I
run this command, the dump command and I'd get an image. I mean the dump
command just prints out some text and I use this program called dots or this
script called dot which converts, takes to an image, and I can get this image.
So then I compare the whiteboard with my image and more often than not I made a
mistake in my configuration.

So this is a good way to make sure you configure this properly. And so I
usually dump two or three times and then I, then I know I got the right
conflagration.

## Hints on Finding State Machines in your App

And I know I talked a lot, and state machines they sure are great. I gives us,
I've told you so many benefits, but how do... I mean where... so I want to give
you some hints. I want to give you some: if you ever have, if you ever have
code like this, this is code, I've written, I wrote a cCRMm or whatever it's
called, basically for salespeople and they have a quotation and I defined the
Quotation can be in these three states, four states. I even have a property
called status. And whatever you see code like this in your application, you
should know like, hey, maybe I should use a state machine for this.

Also, when you're looking at your database and you see something like this.
Like have boolean for active, a boolean for ended, a boolean for closed. This
is also a good sign, like "hey maybe a state machine will be better in this
scenario" And people asking me: "Oh, Tobias, should you really have state
machines if you only have three or four states?" And the answer is obviously it
depends. Are you sure you never ever would add more states? How are the rules
between those states? How do you move? And I would say, if you want to make
sure you don't have any bugs. Yes, a state machine might be a good idea.
Obviously if you're only have two states, I mean it all depends and I
understand, I'm sure you're getting the gist.

## Identifying Processes in your App

So how about processes? If you don't have a quotation, but a process. What if
you're doing e-commerce, it's obviously a process here. You obviously create an
order somehow and items are being added to that order and then you need the
user to fill in the delivery address and then you need them to add the payment
method and then you need to call or somehow notify the delivery team to make
sure that that order is delivered. And this is also like if you have this
strict workflow, you should think of the Workflow component. You should think:
"Hey, maybe, maybe I should use a workflow here." And then you start drawing on
the whiteboard: you draw the order is created, and then somehow we add an item
- we have transition add item, so the item is added. And then you add the
shipping and then you go to the checkout and after the order is confirmed. And
you look at this. Yeah, sure, this makes sense. This is a clear workflow.
However, this means a user can only can add one item, right?

## Predictability and Less Bugs with the Workflow Component

So we need to add an extra arrow here. The user can add more than one item so
it can buy more stuff from you, right? And this, it's pretty simple: you just
go through the workflow and the item added, sure, add another item, add more
items, add shipping, and then order confirmed. And the cool thing here is: this
will, each of these steps will dispatch an event. And on this event I have an
event listener because this is my domain logic. I can trust that this event
listener is executed each time an order is confirmed. And I just asked the
delivery service to take care of this order somehow. Makes Sense? So using the
state machine, I also know that no order will ever be completed or confirmed
unless I added the shipping address. It's literally impossible to complete an
order without adding a shipping address. So what I basically want to say is, it
all comes down to this: Using the Symfony Workflow component will make your
write fewer bugs. And I hope this short presentation of mine have given you a
new way of thinking, a new way to relate to your problems.

Thank very much. Thank you.

I will share my slides on joind.in. And so if you, if you're interested in
anything, you please feel free. If you have any questions, I think we have a
few minutes and I will throw this cube at you. Anyone want to have a cube?
There's a question over there, please help me. Two rows back. Three rows back.
Did you activate the cube? It was a mistake. Oh, okay, mistake question.
Someone else - just throw it to someone waving over there. Whoever has the
laser pointer in their eye.

Sorry, can you hear me? Okay, great. So a question is really simple: where is
e-commerce here?

Where's the camera? I don't know where the camera is - oh there's the camera.

I'm saying this was callex using the workflow component for E-commerce.

Ah, where is the e-commerce? Yeah. I assume that you missed my five minutes
beginning. Michelle sadly got sick. So I did a stand-in talk about Workflow
component. So yes, I missed a lot of e-commerce stuff and we can only blame
Michelle's three year old kid.

There's one more question and then I'm gonna let you all go.

You hear me?

Yes I do.

Uh, so my question is how to properly connect workflows with the security. So
you have two ways: you can write a guard listener, uh, to prevent some
transitions based on security. Or you can, as you showed, you can do a voter
and there decide based on workflow whether you can do something or not. So
what's the correct way?

It depends what you're trying to do. I show the voters, which, I will try to
explain, I used for handling my app security. I used the state machine as a
substitute, as a help. If you're going to add security, saying like you want to
make sure... if you want to make sure that you have the correct role for doing
such as this action, use the Guard event. Okay. So you do something like this.

So both ways are correct?

Both are correct depending on what you're actually trying to solve.

Okay. Thank you.

I will be around for the rest of the conference and I also told you the best
speakers in the front. I'm doing a second talk at the very, very end of the
conference for some reason. So if you're interested in Symfony 4 internals,
please come and visit me then. Please review, rate all the speakers on joind.in
- it's very helpful for all of us. Thank you very much.
