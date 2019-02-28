# Microservices Gone Wrong

***TIP
SymfonyCon 2018 Presentation by [Anthony Ferrara](https://github.com/ircmaxell)

Microservices are the latest architectural trend to take the PHP community by
storm. Is it a good pattern? How can you use it effectively? In this talk, we'll
explore real world experience building out a large scale application based
around microservices: what worked really well, what didn't work at all, and what
we learned along the way. Spoiler alert: we got a lot wrong.
***

Good morning, bom día. So today I want to talk to you about a little bit of a
story. I want to do a very different talk than most other conference talks that
you have seen or that you would have here today. Rather than telling you what to
do or telling you what technologies to use or how to do something well, I'm
going to tell you a story of something that I've royally screwed up. We all make
mistakes. We've all gone out and built systems. And a team that I had the
opportunity to lead a number of years ago, we wound up building a platform. And
I'm gonna tell you a little about the background. But we made a bunch of
mistakes and rather than let those mistakes die out and let me and the team be
the ones that learned from it. I really want to take all of you on a little bit
of a journey and tell you about some of the things that we did right some of the
things we did wrong.

So today is really going to be a journey about what we built. Why we built it.
How we built it. Where we ran into significant trouble, and where everything
worked well. And this is my puppy Adda who's going to help me on some of these
transitionary slides.

## Background

So let's jump into the background. I walked in to a growing team. They had
recently received a major round of venture investment, grew the engineering team
from five to 18 within a year, it would later be about 30 people. And it was a
very good mix between php developers, a couple people with Go experience, a
couple of really, really good frontend engineers, data engineers, et cetera.
They really cared about software quality. The people that they hired in were
very, very good engineers.

But they were dealing with a legacy system, and we've all dealt with legacy
systems in the past. This was quite an interesting one. The one I really want to
highlight here is this bottom one. There were over a thousand cron jobs that ran
as frequently as every two minutes, which corrected data in the database. That
should tell you a little bit about, of an idea about the state of the system.

It was so hard to find and fix bugs that rather than fix them, they just patched
over them and band-aided it. And that's not a knock on the developers. Like this
system was around for five years, was built very, very quickly, was scaled with
different varying amounts of talent. You know, a classical legacy problem. But
the business needed more. The business needed stability. They needed an
application that worked for them, that scaled with them. When I walked in, they
had just gotten off of a nine month product freeze, which means that engineering
said, for nine months we're going to do nothing but solve technical debt. We're
going to do nothing but try to clean up the system. It was another three months
before the first meaningful product release happened from there. And it was
incredibly, incredibly painful to work with.

## To Refactor? Or Rebuild?

So we were left with a decision, do we refactor or do we rebuild? The team had
spent about nine months, like I just mentioned, trying to refactor and kind of
not really having very much success with it. And so what we decided to do was
not to rebuild. We went into a room - head of product, head of UX and about five
or six other individual contributors and myself - and we went into a room with
four whiteboard walls. And we started to ask ourselves, what does this system
do? What are the core things in this application that it assumes. And I'll give
you an example right now. Your system - any application that you work on -
probably has a unique column on the user table for email address. Emails are
unique within the system. That is an assumption. However, in most well-designed
systems, that assumption is isolated to that user system. If you wanted to
change that, if you wanted it to allow multiple emails to register for multiple
user accounts, sorry, a single email to register for multiple user accounts, you
just remove that unique, change your login, change your registration and you're
done. This application, it was all over the place. Things inside of systems that
had no business knowing about an email address relied on the fact that emails
were unique. And so in order to change that assumption, we would have needed to
touch about 60 percent of the code.

So what we did is we went on the four walls of this whiteboard room and we
filled every single wall with a core assumption of what our platform was. And
then we asked product: which out of these do you want to change significantly in
the next 6 to 12 months? We wound up with, out of four entire walls, exactly
three that were not going to be changed significantly.

And so what we realized is that it's not a refactor. It's not a rebuild, it's
actually a V2 . It's actually a completely separate product that we want to
build. At the highest of high level, it solves the same business problem, but
how it does it is drastically different. We could have taken the time and
refactored that in, but it would have taken three or four years to actually get
to the end goal of where the product wanted to be. And so instead, we set us,
set ourselves a four month goal to get an MVP of the V2 up and running. And this
is the story of that MVP process.

## Architecture

So stepping into the technical architecture here, when we started building this
MVP, we had to have some kind of a guiding framework. And what we settled on was
this, and I'll walk you through each one of the pieces. But the basic concept
here is everything that runs on a server only does API calls. The frontend is
the only thing - this frontend server here - is the only thing that actually
knows about HTML, that actually knows anything about a browser.

Everything else talks through this gateway to a service over REST. So API-first
development. The API gateway is the first thing I want to focus on because it's
one of the things I think we got really, really right. This was in the
neighborhood of 2015 and Amazon had just released their API gateway project two
days before we decided to settle on Tyk. Tyk is an open source project, it's
written in GO, uses MongoDB on the backend, it integrates well with console,
with a lot of modern Dev ops tools. But basically what Tyk allows you to do is
configure REST API endpoints with JSON. So I can hit an API and say, create this
new endpoint, here's the backend server that it's going to deal with, I want you
to handle OAuth for me, so terminate OAuth and just give me a user ID. I don't
want to know any of that stuff. Handle rate limiting, handle quotas, do all of
this stuff. And what it actually, one of the really cool parts is it had the
support for middleware where we could actually have a very slowly-evolving
frontend REST API, that our frontend servers built on, that our clients build
integrations against it, etc. While our internal servers were more RPC based, a
lot faster moving and didn't have to worry about backwards compatibility nearly
as much. There was still some challenges there. But the API gateway pattern
definitely is something that I'm actually really proud of from this that we
actually built out, worked really, really fascinatingly well.

### RabbitMQ & Event Sourcing

So stepping back out, the next really key part is RabbitMQ. Again, this is 2015
timeframe, the stack that we're building on is mostly php. We had some GO
services, et cetera, and at the time Kafka did not really support PHP well. Go
did not really support PHP well, sorry, Go did not support Kafka well. And so we
decided to go with RabbitMQ. If I was doing this decision today, I would 100
percent pick Kafka for a reason that's going to be apparent. We were using this
as a pseudo event sourcing database, meaning every time we made a change in the
application, we would emit an event describing that change. Theoretically you
could replay those events in order and get the system back into the state. I
said theoretically because in practice it didn't really work that well. The, one
of the big mistakes that we made was none of the services relied on Rabbit to
set the state. So whenever they emitted the state changes it was kind of just
advisory. So we would run into problems where the JSON wasn't fully populated or
messages were just completely blank by accident and they weren't caught for a
while.

if I were to do this again, I would absolutely use a similar-style system, but I
would make it event sourcing first. Meaning that event list *is* the single
source of truth and the services mainly use that, have a query against it, have
a API where they can look at those events. But, so this was something that
really was difficult to get running, caused us a lot of frustration, but in the
end actually gave us a lot of insights. So it was kind of a mixed bag because
having everything talking to a single archive meant that we had one single
picture over everything that was happening in the application from the event
stream. So while it was a pain in the neck, it also did help us a lot.

### Service Layer

The other thing I want to talk about at this layer, is this service layer. So we
divided our services into roughly three categories, domain services which are
meant to sound like domain objects because that's kind of how we were thinking
about them. So our business entities, all of our business logic, all lived in
these types of services. They communicated over HTTP and they all had their own
persistence. So they all have their own databases and caching systems. We also
had asynchronous services that purely listened over RabbitMQ to do things like
long running jobs, batch processing, we did a lot of video transcoding, et
cetera. So all of that stuff was handled via asynchronous services. And then we
finally had these things that we called meta services.

It's kinda hard to explain what a meta service is. So I'll give you an example
in just a second,.but one of the challenges, we approached this design as normal
object oriented design: your domain entities, you split them apart, you find
your boundaries, you create your services just like you would do it in PHP,
whether it's Symfony or Laravel or Zend or whatever framework you want to do.
And those services talk to each other. We modeled our microservice architecture
off of very, very similar principles. There is something that we didn't consider
though.

## HTTP Calls will Fail

How unreliable a service call is in relation to a method call. So ask a
question, ask yourself a question. How often do you expect a method to fail
randomly? And I don't mean the method to not return because that would be
somewhere around one in a billion. If you look at Symfony in a normal default
configuration running in production, your front page may take 10,000 method
calls to render. And how often does one of them fail randomly? Maybe one every
100,000 requests? But we're actually not talking about what happens inside that
method because that stays the same when you go to services. We're actually
talking about the method call itself. It's so infrequent that you've probably
never even thought about it. You've never thought that: hey, I have this object,
I know it's a valid object, I'm going to call this method on it, maybe that's
not going to work. Whereas in a service architecture, you absolutely, positively
have to. HTTP calls, if you're very, very good at operating an HTTP service,
you're maybe going to get five nines, five nines of uptime, 99 point nine, nine,
nine percent uptime, translates to one every 100,000 requests will fail
randomly. And there's nothing that you can do about it. So compare those two
numbers, one in infinity versus one in 100,000. This is what got us into a lot
of trouble.

So the question is, how small should you build your services? We started with
the idea of one week. One week should be about the amount of time that if you
have a solid specification for a service, you should be able to go from zero to
a working service. That way, if we made a mistake, if we had a significant
issue, we could literally throw a service away. I've heard some people talk
about that number as a rough benchmark. And at this point in my career and after
this experience, I will tell you that is an absolute mistake.

Here's why, this was a rough model of our domain. We did an e-learning platform
that would deliver lessons via a web platform. And so we had users, we had
assignments which assigned lessons to users, we had a history which showed what
lessons a user interacted with. We had content associated with lessons and we
had assets associated with content. And the way we modeled this as a system
architecture was roughly every one of those entities became its own service.

Now it's actually a bit more complicated than this. This is a little bit
simplified to make the point: each one of these services did have more than one
database table. It did have more logic built into it, but this is the rough
concept. And so this raises a question. If you are building for the frontend,
how would you get everything that you need to service a request or to render a
page that shows an assignment? Today and in fact, back then, one answer could be
GraphQL, right? GraphQL is phenomenal at stitching it things like this together.
The problem was this need isn't just had on the frontend. Backend services
needed to look at things and aggregate as well.

So that's where these meta services came in. They had domain knowledge. So it
knew what an assignment was. It wasn't just stitching things randomly together.
It knew what it was creating so we could actually add some business logic in
there around that. But most importantly, it acted as a pain function for
developers. If we got our backend model wrong, we would feel it when we built
that meta service. And so by forcing us to maintain a service to fill in the
gap, it forced developers to realize that: hey, this other model was bad. Or
this other model had problems.

So let's ask another question. How would you get a list of lessons ordered by
the author name of the content within those lessons? Think about where that data
lives. You want to order by this user service over here, but you want to join
against content and then finally return lessons. That would be an absolute
nightmare to do over REST. I mean that's, you basically would have to query
every single row from every single service and try to stitch that back together.
It took about six months to solve that problem once we sat down and tried to do
it, which is where I think the real failing of this architecture and going this
small on services. We ignored what the business domain was. We ignored the
bounded context and went way smaller than was necessary. And to be fair, we
didn't know that this type of requirement was going to exist when we built it.
So keep in mind, keep things as wide as you can and only really cut those
services when those boundaries are clear and easy. By the way, the way we solved
this was with a service that used ElasticSearch and basically kept its own model
of everything in here and became a generic search service, which yeah, was
challenging.

## Infrastructure

Let's talk about infrastructure. I just told you something that we got horribly
wrong. Now I'm going to tell you about something that we got ridiculously right.
I think at least. We had been running Mesos, Apache Mesos for a while at that
point because we were using Spark. And Apache Mesos is basically very similar to
Kubernetes but for arbitrary jobs. You can run a farm of servers. You give Mesos
jobs and it figures out how to run them and it runs them across the cluster. So
we had a lot of experience running Mesos and at the time Kubernetes was really
not a thing. I think they had just announced it. It may have even been alpha,
but none of the cloud services supported it. And so we decided to go this
direction, running Marathon, which was the Docker scheduler on top of Mesos.

Took a little bit to get running. But once it was running, it worked
phenomenally well. And so I want to take you through the life of an actual
request just to show how powerful this was. The very first thing that happened
when you called an API was it would hit an external elastic load balancer, an
Amazon ELB, which are very, very, very reliable, but kind of slow to
reconfigure. Whereas Marathon would want to reconfigure things every couple of
milliseconds at times. And so the ELB would talk to a ha proxy instance, which,
was a little bit less reliable but was very, very, very fast to update. Those
requests would then go into Tyk into our gateway and then Tyk would call an
internal service inside the firewall to an internal ELB, which would then go
through the same process and hit our service. This looks heavy, but including
Tyk, including OAuth termination, including rate limiting, including the REST
deserialization in any middleware that we had in here, this entire process took
about 10 to 15 milliseconds. So really, really, really fast and gave us near
infinite, well not near infinite vertical scalability let's not go that far, but
gave us a good bit of vertical scalability with this, or horizontal scalability
actually.

And then if that internal service wanted to talk to another API, it basically
just bypassed that external step and just talk straight to that other service
through that ELB. So what we wound up having was a system where if we wanted to
add nodes, we could drag a slider and within 30 to 50 milliseconds have every
single machine running a new service and the system would reconfigure itself. We
were deploying on average, I think it was about 500 machines per day where we
would spin down old machines and spin up new machines and we had almost 100
percent uptime during the time we actually ran this, that I was there. It turned
out to be very, very reliable once we got it up and running.

Touching on logging really quickly. Logging is insanely important when you're
building a distributed system. This was one of the core things that we did
initially using LogSpout to collect, to stick things out into DataDog. So StatsD
as well for application metrics. And we also used something called Zipkin.
Zipkin was simultaneously one of the biggest pain in the rear ends that we
worked with as well as one of the most powerful tools that we had. Getting it
running at least at the time was an utter nightmare. Once it was running, the
data that we got was incredible.

Basically, when you have a service that calls other services, you pass along a
request id on that other call. And Zipkin, looks at that data and is able to
correlate requests. So you can see for one service, it may have taken 100
milliseconds to serve that request. You can look at every single sub request,
every single part, no matter how far it fans out into your system, all from one
graph. You can see the network effects, the configuration that happens when you
have one service fail, how that affects other services. Zipkin took us a very
long time to get up and running and we paid a significant price. A lot of things
would have been a lot easier to debug had we have gotten that up and running
sooner.

The final piece on the infrastructure side, we implemented something called,
that we called the service.json file. This basically lived inside of every
single services repo. It described the name of the service, which would turn
into a DNS name. It would describe what other services this one required to run,
what database it needed, and so we could actually spin up databases, run
migrations against them, configure the credentials 100 percent automatically.
Same thing with health checks and the APIs that that service exposed, as well as
all the things that Mesos Marathon needed to run it properly. And so with one
file to get a new service into production, all we needed to do is go into
CircleCI and say, build this project. And it would automatically deploy
everything into production when you merge into master. Worked actually really
quite well.

One thing I'd point out does anything that I just talked about look familiar?
This is basically Kubernetes. That's what we wound up reinventing and looking
back on it, it feels a lot like we reinvented a lot of the wheel. But the
reality was we were just a little bit too far ahead of where things were coming
and I don't mean that in a good way. But we wound up reinventing a lot that was
ultimately coming down the pike for major open source projects. And today just
use Kubernetes. You don't need to build the rest of it and you can get the same
exact benefits.

## Local Dev Experience

So moving along, that's how it worked in production, at least in theory. Local
dev was a little bit of a different story. So the initial intention and what we
built at first glance was a command line tool. So you would check out a
repository for service and you would run this command line tool in it. And it
would read that `service.json` and figure out what you needed to run your
service, configure a docker compose file to spin up all of the other services to
run all the migration files to get your databases all set up and everything like
that, and get everything up and running so that you had a local dev that
basically exactly mirrored production, in theory.

The problem was nobody actually used it. Every engineer ran their own service
natively on their machine. And when they needed to run a dependency run another
service that they had to talk to, they would either mock it themselves by
creating a little, you know, Node.js script or a little PHP script to simulate
that endpoint, or they would talk to another developer to get the other service
running on their machine. And this wound up having a big problems because the
amount of times that they would update that destination service was not really
that frequent. And when they did, they weren't really in the habit of running
migrations,.and so when we went to integrate this in production services weren't
used to talking to each other. It became an absolute nightmare.

And so we stopped and we asked why. Why weren't the developers using this tool
that built and worked? That was built and actually worked? And the answer was
actually pretty simple. There's two layers to it. The first is that tool was
slow. It took on average, because of the number of dependencies that had to spin
up, and it basically started from a docker, from a fresh slate every single time
you booted, took about 20 minutes to get up and running. And you figure you do
that once a day, once every couple of days. But that also means anytime you want
to reset the state of the system, you have to wait 20 minutes. That's kind of a
ridiculous amount of time to wait. It was also really unreliable about half the
time it wouldn't start,

But there is another key point. The way we had built the team in the way we had
built this tool was such that it was somebody else's problem. There was somebody
who is designated on the team to build and maintain this tool. Developers had
agency over their `service.json` file, but they had no agency on whether or not
their system ran in this tool or not because ultimately the tool didn't matter.
What mattered was prod and as long as prod worked, nobody really cared. And so
that was a really, really huge fault and failure on my side. And one of the big
things I learned as a takeaway is you've got to, especially in a services
architecture, you have to get the local environment right and consistent because
that will be the difference between a system that works in a system that doesn't
work.

## Getting it Running

And speaking of that, I want to talk about actually getting this thing off the
ground. So we spent about three weeks building the first couple of services,
authentication, user service, and a basic content service. And we had all three
of them running in local dev and we went to put it into production. And it took
a month from that point before the first successful login on the frontend
happened. A month, from working code to serving traffic. That's just absolutely
ludicrous.

And so we started to ask ourselves, we had retrospectives after retrospectives
and there is a whole bunch of reasons for it. One of the key ones being the
local development environment was not stable. So therefore getting into prod was
actually the first time these services ever talked to each other. And so the
services were built in isolation. It really caused a lot of pain. Some of this
was just normal growing pains, you know, you're building your infrastructure to
kind of an ideal, you expect health checks to behave exactly a certain way and
in practice there's a little bit of fudge room for that. And so there's always
little kinks to iron out, but it was really, really challenging.

A little more detail on a couple of these. I'm not going to read these each
independently. I'll get to the coordination in a second. But getting the local
state and staging environments into a known state was insanely challenging. So
what I mean by known state is, let's say you have a bug in production. How would
you replicate that in a local environment? If you're dealing with a monolith,
maybe depending upon your security requirements, you clone the database? Or if
you're actually doing things well and GDPR compliant, you actually have a system
in place to anonymize that data? Or even better yet, you're able to recreate it
directly on your local without having to copy any data.

That was literally impossible here. Every service had its own database. There
was data in all sorts of places and Rabbit MQ queues, in S3 that the system
used. And so to get the system into a known state was literally impossible. The
only way people could actually do it was by going into the interface and
clicking things in the interface to create content and to create assignments and
all this stuff. So both from a debugging standpoint, but also from a build
standpoint, it was insanely, insanely challenging. We had an idea for a tool to
solve this problem, which was basically to detail the state in a yaml file and
give it to a tool which would then coordinate with all the services to get
everything into a known state. It seemed like it would have worked to solve the
problem, but again, just a matter of time, we didn't actually get a chance to
finish it.

## Dealing with Change

The high-level coordination is a really interesting challenge. How do you deal
with change? So change is always a natural part of software development cycles
and of getting things into production. You know, we're never going to get it
right the first time, whether it's feedback from clients tell you that you built
the wrong solution, whether it's an executive stakeholder that walks in and
goes, I demand that you add this feature because I'm just executive and I demand
things. Or maybe it's because we actually literally screwed up the first time
that we did it and we misunderstood the problem from an engineering standpoint.
We built the wrong thing. So change is a natural part and, you know it's going
to happen.

Here's an example of a real change that we faced. This is a rough abstraction of
one of the hierarchies in our system where a program has many topics, a topic
has many lessons, a lesson has many cards, and a card has many assets. Don't
worry too much about what these things are. Just think of them as entities. What
happens when the business comes in and says, I need a new layer?

In a normal monolithic application, this would be trivial. You would add a new
database table, right? Maybe a little bit of a migration and 90 percent of the
time things will just work and maybe you spend a couple more days cleaning stuff
up and adding the UI elements and stuff like that. It took a month to do
something like this because, remember, everything is a separate service. How
would you make a change to this hierarchy?

Well, it turns out, you first have to make the change in anything that depends
upon that service. And you have to make the change such that it can accept
either the old or the new way. And then you need to add that new service. You
need to run all the migrations to get all the data into that new service and
then you have to change all of the other services again to get rid of that
duplication, to get rid of the acceptance of the old way of doing things or the
new way of doing things. And so basically what would have otherwise been a very
simple refactor turned into massive coordinated surgery. Typically these type of
refactorings required half to three quarters of the team working for weeks at a
time. And these are things that any application goes through, like these are not
complex things.

And so what we should have done, in retrospect, what I would have rather done is
this: take our domain model and group it by bounded context, group it by the
things that are similar that need to know about each other and put these other
services out in other areas. Asset service is its own thing here, it could be
part of lessons, but considering video transcoding and stuff like that, there's
a lot of things that are unique just to assets. And this is ultimately what we
wound up doing. We did wind up throwing away three quarters of those other
services and building what some people would call microliths.

## Lessons Learned

So I want to wrap up a little bit here with some lessons that we learned and
some key takeaways here. First thing that I would recommend is don't do
microservices. Now this is a little bit of a joke because you can very clearly
see what we did at least initially was not really microservices. It was a
distributed model. But I would strongly, strongly say, do not build a non
monolith unless you can invest in operations, unless you can invest in
automation and tooling and have every single developer own and be responsible
for that tooling, it's not gonna work out well. Or at least, it didn't work out
well for us.

Start with big services, especially when you don't understand the problem. You
may think that you understand the problem, but unless you have solved that
problem in production, start with a big service because it's far, far, far
easier to take something that's big and complete and split it into, break it
apart because you know where your challenges are. You understand your domain,
you understand the problems. But to try to stitch two services together,
especially when you have dependencies in other parts of the application, becomes
insanely challenging.

Automate absolutely everything. And I don't just mean deploys. I'm talking about
your infrastructure changes. If you're not using Terraform, use Terraform, etc.
Like make sure the backups are 100 percent automated. Make sure that how you get
the application into a known state is automated. Automation and autonomous
systems are absolutely critical, especially as complexity increases.

This is, I think, the biggest lesson that I learned. Anytime that we're really
dealing with distributed systems, but a lot more so applications in general.
Normally the way we write code is we write the happy path first and then we test
the sad paths and we write the sad paths. And even TDD with the red green cycle
is designed to kind of do this. You don't write the failing test for your
failure case first, you write your failing test for what the business problem
is. Then that fails and then you write the code to solve that. Think failure
first. Start off with: what happens if this thing breaks. And write the code to
solve that first, because things *will* break. Your database will go down, you
will have corruption, you will have a network failure. No matter whether you're
building a monolith or a distributed system, things will go wrong. And changing
your thinking to start thinking about what's going wrong, what's going to
happen, what's failing and how will I gracefully handle that, becomes absolutely
critical at maintaining a scalable and highly available system.

And then finally, this is I think one of the things that we thought we
understood in the beginning, but we didn't. SLO's are a concept that came out of
Google's SRE book, which is their service-level objectives. Basically, what does
the business care about for this service? We measured everything in our
services. We measured requests per second. we measured memory usage. We looked
at number of database queries per call. Like any technical metric you can think
of we probably were tracking it. But we weren't really tracking with the
business actually cared about. Yeah, we thought we were. We thought were, you
know, number of pieces of content accessed per second, but that's really not
what a business cares about. The business cares: was a lesson consumed? Did the
learning happen? It was an e-learning platform. Did a service respond in a
timely fashion and not a timely fashion from a technical definition, a timely
fashion from what the actual end user cares about. And so by defining these
SLO's when you, before you build your service, you actually have a metric that
you can test your service against.

The biggest takeaway that I have from this entire experience, and I've talked
about this a little bit on twitter and I know there's different opinions on
this, but I've learned that complexity is the number one enemy in software
development. Everything that we do: when you look at refactoring, when you look
at object oriented design, are ways of managing complexity. And quite often what
we do is we take complexity from one part of the system and we spread it out
into the rest. This file is easy for me to read. This code in this method is
easy for me to read. Therefore, it's simple. Therefore my application is simple.
When in reality you didn't simplify anything. You just moved the complexity from
one place to another. And so managing complexity becomes the absolutely most
important thing to building a reliable system.

The way I would phrase it is this, it is insanely simple and insanely easy to
create a system that is so complicated that you cannot understand it. And if you
can't understand it, how can you run it? This is basically the story of the
system that we built and where it failed, where it broke down. And so I want to
say thank you and I think I have a minute or two for questions if anyone has
any.

Come up to the podium. Nope. I'll throw the cubes. Ok, to anybody? Or to
questions? Okay. Here you go.

Thank you for awesome presentation.

Oh, okay. Who wants the cube? Who wants a mic? Up here.

Can you hear me? So, uh, I wanted to ask you a question like: normally when
you're trying to split the system into smaller services, you often end up
splitting the complex logic into complex.... You can't hear me? Can you hear me
now?

Yeah, it's, I can't really hear you over the background noise.

Uh, so I'm going to try my best.

Yeah, that's better.

Okay. So when you're trying to split the large system into smaller components or
microservices, you often end up making the service simpler but the communication
harder and more complex as you said. So how would tackling such a problem be? Be
like, identify, be like making a service more responsible communication
adaptation for data and then forwarding data and formatting it? Or how, how
would you advise people to tackle that?

So I think that's where I get back earlier where I said failure is the key and
thinking about those failure modes when you're splitting that apart. Think of
the happy paths when, if that service is offline or if those data communication
channels fail, how would that behave? And try to work out the system such that
they actually do behave correctly, when those things happen. Or they can
gracefully. Unless I misunderstood the question, it's kind of hard. Come up
after, we'll chat afterwards about it. Alright, thank you.
