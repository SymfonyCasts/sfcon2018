# Building Apps for Immutable Servers

***TIP
SymfonyCon 2018 Presentation by [Daniel Gomes](https://github.com/dcsg)

Immutable Servers are not new and they are not that simple to achieve. However,
the advantages of having Immutable Servers versus Snowflake Servers are worth it.

In this talk, I will explain the differences between Snowflake and Immutable Servers
and also what considerations you should have while developing your App. I will
also share which tools and strategies you can use to build Immutable Servers.
***

Welcome! Good afternoon! I'm here to talk about building apps for immutable
servers. But that's not slightly true. While I was building the talk, I slightly
changed the title so it's more scaling apps with immutable servers. And as just
a disclaimer, there will be no code. We are not going to build any app. So, you
should expect to know more about servers, and how you can use immutable servers
for your app and how you should use them, and if you should use them.

## Hey Daniel!

So, a bit about me. My name is Daniel Gomes. I am Portuguese. I'm from Lisbon.
I'm a software engineer at Teamleader. I am also the organizer of `@phplx`, the
user group, the PHP user group in Lisbon. You can find me on twitter and
sometimes I blog.

So, before I start I just want to know, how many Portuguese people I have here.
Can you just raise your hand? A few? Probably 10. That's nice.

## Topics to Cover

So let's get started. Some topics that I'm going to cover. We are going to cover
different types of scaling. We are going to cover configuration drift and
management, different types of servers, and also a build process for your web or
application server.

## The Beginning of a New Project

So, at the beginning, usually when everyone at the office starts to work in a
new product or a new application, you start some wire framing, some designs and
into the site. And then you start doing the code, there. and then you need to
decide our first infrastructure.

But usually, everybody already should do something like this, did do something
like this. Is that you put everything in one server because you are just
starting, you want to just have something really quickly to the customer to get
some feedback. Right? So you end up with something like this. How many have you
ever ended up with the application, the database, cron jobs, whatever in the
same machine that your application is running. Can you raise your hand?

Yeah, 80% of the room. So, I have been there as well. So, everybody does this
and this is okay. No problem. Then you launch your app, everything went well.
Then, users start using your application, you start having some rough problems
and your server is unavailable. This is a good thing because if your server is
unavailable because of the users - it's, it's nice, it's working. You are
getting results, right?

## Scaling from a Single Server

So, the CPU, memory, whatever it is - is not handling the usage, the load, and
then it's the time that you say: well. we need to scale, right? It's easy,
right? Scale is easy. Is that easy? Scaling? Do you think? Do you think that you
can scale this thing?

You are correct in one point, but in another one - you are not correct. Yes and
no. You can vertical scale it, right? That's the first step. So, you add more
memory if you need, you add more CPU, more this or you're just fine tuning your
php.ini, your PHP-FPM, Nginx, whatever you have. These are the options that you
have with the vertical scaling. Otherwise, then you would have more hardware to
add more CPU or memory.

## Horizontal Scaling

So, and for horizontal scaling. So, if you want to add... so, the goal with the
horizontal scaling is, basically, going from one machine to more than one
machine. In this example, is 4 instances of your server, but it can be 2
instances, 3, 10, 20. It's depending on your, on your use case of course. So,
let's say that with the current server that we had, we create a cluster and we
have this kind of architecture. So the question is: Will this work? Of course,
not. Yeah? Nope! Why?

Because you have persistence and state inside our server. So, if I do a change
in this machine, in these instances, what will happen to the other ones, they
will not be in sync, right? So, what is the point of this? Does not make any
sense. So, what you need - you need new architecture, this is not the only way
to go. This is an example, of course.

So, you put the load balancer on front of your application tier or web point
tier you have a cluster of databases, a cluster for AMQP, a cluster for any kind
of infrastructure... the persistence that you need. Okay? You can also have your
assets in a CDN, whatever. So, and you end up with something like this, right?
That's okay. This should work.

## Bugs on Production

But yeah, you have it, everything is working fine, it kind of handled the load,
but then you have a bug in production. This happens. Who did add a bug in
production? Everyone, I think everyone, the ones are that don't raise their
hands are a bit shy I would say. So, and yeah, sometimes you are in a rush or
it's in the middle of the night and you just need, I just want to fix this thing
and I went to go sleep and you do SSH... Oh, I figured out what it is, I change
it. It's okay. But in a cluster, this is not a good idea. Right?

So, the problem was solved and you go to sleep. You think. But was it really
solved? If you have horizontal scaling. If you only change one instance, do you
think it's solved? No. I see some... some heads saying no. So yeah, you are
right, it's not solved. This is what happens. So you changed one instance on the
top right. That's the fixed one. But the other ones - you didn't change them.

## Configuration Drift / Snowflake Server

So, you have an inconsistent state in your... in your instance, in your cluster?
This is not good. And, at this point, you enter into the configuration drift.
And configuration drift is basically manual adopt changes to your servers that
are not recorded. If you don't record them - you cannot reapply them to the
other instances. So you have difference, different states between your servers
in your cluster. So, this is configuration drift, and this is your cluster after
a few months if you continue to do that. If you change the instance with SSH
without recording and reapplying to the other instances, you end up with
something like this. And what happens is that your servers become a snowflake
server.

I don't know how many of you know what is a snowflake server? Some hands.. so,
yeah. So probably, most of you, or a huge part, a big part of you will have
snowflake servers today that are long running servers. There are no consistency
between them in your cluster. They hard to reproduce. So, you cannot just build
a new, a new server from the image that you have because if you don't record the
changes, you cannot reapply them to a new server that you launch. So, you cannot
reproduce. And this is the most important for me - lack of confidence. Do trust
your, your server, your cluster, if you have different states in your instances?
No, right?

## Configuration Management

So, how can we solve this? So, you can use configuration management with
automation tools. And the configuration management is just the process of
systematically, systematically handling changes to a system in a way that it
maintains integrity over time.

Let's see an example. So, let's say that I want to spin up a new server or I
have a running server and I went to do some changes to my, to my server. What
you need to do is you first change your configuration files and then you run the
configuration management to apply that changes into all your servers. So, that
way you end up with all your servers in the desire state.

Let's see how this works. So I have my 4 instances. I have my... I want to apply
the state D. I run my, my configuration management and after it runs, hopefully
if nothing goes wrong, everything will be on state D. Yeah, because even if you
run configuration management, something can go wrong if you don't have internet
and stuff like that.

So what tools can we use to... for configuration management? There are a lot,
there are... These are some that I know. You have Chef, Puppet, SaltStack,
Ansible. How many of you have already worked with some of those tools? So, how
many of you work more with Ansible or Puppet? Ansible? Yeah, I think, yeah, most
of the people I think works with Ansible. I prefer Ansible too. It's more easy,
for me.

So, let's do just a quick recap. So you have seen different types of scalings,
vertical and horizontal. You have seen what configuration drift is. How you turn
your servers, all your services can become a snowflake server and the problems
that they have. And how you can use configuration management to solve those
problems. Right? So we are reaching half of the presentation. So this will be
more topics so, I think it's the perfect time for you, if you have any questions
so far to make it now because probably at the end you will not remember. Any
questions so far? Nope. So, let's continue.

## Phoenix servers

So, let me present you Phoenix servers. Anyone of you know, Phoenix servers?
Only one I think. Oh, nice! So, Phoenix Server! Quoting Martin Fowler: a server
should be like a phoenix, like regularly rising from the ashes.

Yeah, that's true. So, if you do this, you will avoid configuration drifts. You
have disposal, disposable servers and servers that can be built from scratch,
right? Because you can apply everything with the configuration management. So,
you're sure always the state when the server is built. So you can just terminate
an instance, and create a new one, and you are done. That's nice.

So, let's see an example. So I have my 4 instances. I terminate an instance.
Instance terminated. A new one is spinning up. I apply state C, run the
configuration management, and voila. It's done. Everything's working fine. Cool.

So, spinning up new servers is not a problem anymore. You are not afraid of
doing that, that everything will not be in the same state because you have the
tools in place to help you.

But, is idempotence guaranteed here? Do you think? Nope. Yeah, you're right. Do
you know why? What if the package repository are down when you are doing this?
And imagine that you are having a major outage or you have a problem in your
servers and you really need to deploy a new version of your server. And, S3 was
down for 48 hours, something like that, and it was a mess. Nobody could download
anything because other repositories were there in S3 a couple of few years ago,
I remember that, was really crazy: what the... is only S3? And I cannot download
anything. I'm not even talking about Composer, installing Apache or Nginx. It
can happen.

So, let's say that you have this process that you spin up a new server, you run
the configuration management, the configuration management will install
packages, create folders, create users, upload the app, and then when it's
finished, I have my server ready and in the desired state. So let's say that at
certain point the internet or the repositories are unavailable. This means, that
I can create the folder, I can create the users, but I cannot install the
package. So I will not have my server in my desired state. So, yeah,
configuration management is nice, but if I don't have internet that does not
serve me. Right? If I need to deploy right now without the internet, it will not
work. And that's not good for our business and for you, if you are running
critical stuff.

## Hello Immutable Servers

So how can we fix this? Anyone has an idea? What? That would work, but if that
instance has problems, you're not fixing anything, you are just ensuring that we
have a new, but yeah, that would be, could be a solution in it. But yeah, you
can have immutable servers - that will help you. Let's see why.

So, according to Kief Morris:

> An immutable server is a server, that once deployed, is never modified, merely
replaced with a new updated instance.

This is really powerful. So what does this mean? This means that when I do a
build, I have the final image with everything baked in. This means that my
application is there, all packages are installed, everything is configured. It's
just spinning up a new, a new server and application will start running. You
don't need to do anything. You don't need, well, you need the internet because
if you are using aws - you need, yeah, but that's... than you cannot do anything
right. If you don't have the internet in your local machine and if you need that
to spin up new servers - that immutable servers won't help you because that's
not the problem of the immutable servers.

You cannot perform any change after the image is built. So, if you want to, if
you did a mistake and you want to fix it, you have to build a new image. Okay?
And you need to include all scripts, everything you needed for the application
to start on boot because you cannot SSH into the machine, you cannot do anything
like that. Okay? That's the point and the goal of immutable servers. So, this is
really easy to scale out, to deploy and rollback, right? Because let's say that
you deploy something, you have a bug, and if you don't save the image of your
last version, you have to build everything again to roll back. That is not a
good thing. But usually people's save the last image so that will not be an
issue, but still.

So, therefore you have much more confidence in what you have in your servers, in
your infrastructure. You rely on them. You can... you have reliability on your
infrastructure and what's the most important - trust. Your business will trust
what you have. You are going to trust, everyone will trust in what you have.
Right?

And I think this is also very important for me. You can sleep very quietly and
you know that it's not because your server is having issues that you are not,
you are going to wake up. If you have auto-scaling then it will also help you.
But let's talk about auto scaling after.

## 12 Factor App

And also, I don't know if you know the 12 factor app, but the 12 factor app is a
methodology for software as a service that basically says these 12 points. I'm
not listing all the points here, but it says that you should have this
possibility, your configuration should be in the environment for the
application. You should have dev and production parity, and the backing services
should be attached to your application server. Backing services is your
database, AMQP, whatever, anything that is not your application running, not
your application code running. So, any service that it uses to run, etc.

And also, this one I discovered while I was doing some research for it. There is
the Reproducible-Builds.org that is, they define that a reproducible build is
something that you can verify: the path from source code to binary. So, this
means that if you, if you give the source code, the build, the environment, and
the build instructions - every time that you do that build, you will have the
same outcome always. Exactly the same. It's like the checksum. Also, Symfony is
part of this organization and there is a blog post on the resources.

## Building Immutable Servers

So, how do we do, how do we build it? You need continuous Integration and
continuous deployment pipelines. Continuous deployment is not necessary but it
would be nice because it makes sense: if you do a change, it can be very quickly
to go to production if you're trusting what you are doing, depends on your
process, of course. And one very important distinction that you need to do is: a
build is different from running your server. If you build - you create an image
that can then be launched, can be used to create a new server. Okay? A build
just creates an image, period. It's just that.

So let's say that you have this simple build process. This is just an example.
So, you spin up a new server, you run your configuration management, you
configure the application environment, you have the server in your desired
state. Then, you bake in everything that you need for your app and you have your
application final image.

Okay? That's a simple one. It's more or less easier, but it can have some
issues. Let's say that you want to apply some security updates. It can be more
tricky. I usually prefer a multistep build process. This is my preference. Okay?
So, basically, I like to have a base image where I have my OS hardening and all
common tools that I might use because you might have a server for PHP, you might
have server for Go, you can have different servers, but you want your base
server to have all the tools that you need for your infrastructure. So, we don't
have to configure different things for different types of servers. Okay?

Then I have my application base image where I install all the software needed
for the application to run, I create the folders, the users, permissions,
everything that you need before you just upload the application. And you put
everything that it needs to. Okay? It's just the basic stuff to run: Nginx,
Apache, that sort of things. Okay?

And then, the last build would be creating the application image, the final
image that you use to deploy to create new servers. So, you upload your app,
boots, you, you make any script to boot the application on start up - composer
install, to clear cache, all that stuff. That way you know that when you deploy
the application, the server is ready to receive traffic. Okay?

So system upgrades, security updates. Usually, I like to do them on the base
image, but this means that if I do them in the base image - I will have to
rebuild everything. It's a chain, right? If I do a change in the first - I have
to build again everything. It's a bit overload, it can be costly in terms of
time, but yeah, it's, I think it's a more structured process. It has advantage
and disadvantage, but yeah.

So, any application security by application is not your application, not a bug
you have in your security... issue you have in your application, it can be a
security in Nginx, so a security problem in Nginx or PHP-FPM, or whatever, that
should be done in the base image or any configuration should also be done there.
So this is basically, this is a multistep process, I like this process, it has a
bit overhead. It's not for everyone, but yeah, it's an example.

## Building for a Symfony App

So building tips for a Symfony app, you should run the composer install, cache
clear, clear any cache that you might have in your application and you build
your assets or push them to a CDN. So, there are more but these are the ones
that I remembered.

## Build Tools

So what tools can you use to help us building our servers? We have some, this is
just a subset of them because otherwise it will be a slide full of logos. But I
will just mentioned some. So we can use Vault, I don't know, if how many of you
know Vault? So, probably one third of the audience. Vault is, basically to store
your secrets and that way you don't need to have ENV vars in your, in your
application and all that stuff. Vault will store your secrets for you. And when,
in your build process you go to Vault to fetch those secrets, depending on your
environment.

Packer, how many of you know Packer? A bit less, probably 10 percent. So, Packer
is a really nice tool from HashiCorp. I really love it. With Packer you can
build your server for different platforms at the same time in parallel. It's
amazing. So you can create your production container. Let's say you want to use
Docker, you can create your Docker production container. If you use Vagrant for
development, you can also create your Vagrant image, you can deploy, you can
create stuff for Amazon, for Google in the same, in same configuration file.
It's really powerful. You have Docker, of course, Azure, AWS, Google Platform,
Chef, Puppet. So, there are a lot of tools to help us to create these immutable
servers.

So, what are the next steps? If you can have immutable servers, you are safe,
you can trust in our infrastructure, but then you can start doing more crazy
things. So, we can start thinking about high availability, auto scaling. How
many of you already do auto scaling here? Hmm, a few. Probably, five percent of
the, of the audience. Nice. So, if you have immutable servers - it's really easy
for you to just say: Okay, if my CPU is at 80 percent, launch me a new instance.
It's easy. It's everything baked in. It's just a matter of a minute, probably,
depending on where you have your infrastructure. And now it's built. It should
be really easy to auto scale and also to scale down your cluster. Or, if I'm not
having that much traffic - kill me some instances and you save money. Also you
can auto scale your web tier, application tier. It will always depend on your
architecture that you have in your infrastructure. Okay?

How many of you have heard about Chaos Monkey from Netflix? Okay. 2% more or
less? Three, five people? Yeah. So Chaos Monkey was a tool that Netflix had the
need to create for them to test their own infrastructure. So, basically, Chaos
Monkey goes into your cluster and says: hmm, I will kill this instance now. And
it shuts down the instance. To stress test their application, how it reacts to a
failure of an instance suddenly. That's a nice tool. How many of you would have
the courage to just go now to your laptop and say: I will terminate this
instance now. And it will run? I saw to it. Yeah, so that's nice. That's a good
thing, but many of you will not have that confidence to do things like that,
right?

But then Netflix said: Well, one instance in one cluster is cool, is fine. But
what if I shut down my entire region? That's Chaos Kong. So they basically say:
Okay, you are good in one region if one instance or two failed - you're okay.
Let's try shutdown the entire region. Now the fun starts. So the guys are crazy.
Of course, many of us will not need anything like this, but yeah, when you have
immutable servers and servers that you can trust, you can start thinking about
these kinds of things. Of course you are not going to need them always or in
every single job that you have or probably in your life. It all depends who you
are, because there is not many Netflix out there I would say.

## Final Notes

So, some final notes. The goal of this talk is not for you to come out of here
and tomorrow, start doing some immutable servers because as you can see it's a
lot of work. It's some overhead, and you might not need them. So, my goal with
this talk is more for you to think what you have in your application and start
thinking, if I want to go to immutable servers, what I should do now to prepare
for that. And that's a good exercise that you should do because you always say:
Oh, we don't need to scale. But if you don't need this now, let's not do it. But
when you know that you are going to need to scale, right? And if tomorrow
something happens and your application starts receiving a lot of traffic, you
will not have the time to change from vertical to horizontal. Trust me, it's not
something like this and it's done. It's not. So, you should prepare really
prepare your application, your infrastructure to start thinking about immutable
servers to be able to achieve that one day. This does not mean that you are
going to need them, so only use them if you really need them. Okay? That's my
advice for you because they are really tricky. They are really useful, really
nice, but they can be really tricky to get them. Okay?

## Questions

I have some resources here. I will probably update this slide. I will then share
the slides and update some resources, but there are some resources that you can
find out more information about this. Uh, thank you. Uh, I work at Teamleader,
we are hiring, so if you like what we are doing and you want to find out, talk
with me. Any questions? Questions, anyone? There is one, or I never throw a
thing like this - it's my first time. You? Let me see if I'm. Ah, sorry!

Yeah, a state in Symfony. Um, session. For instance, you should not store your
sessions in your application server. Otherwise, if the user, the next request
goes to another server, the server will not know if it's authenticated or not.
Right? If you use the sessions for storing the state of the user for the
application for instance.

(Question inaudible)

It depends on your use case. If the authentication user or anything that you
have in your session is critical for you, then it's a problem. There is no rule
for all the use cases - it depends on your application, what is critical for you
or not. So yeah, for, for some applications, having state in, uh, in different
states on the session in different places might not be a problem, but for others
not. If I'm dealing like invoicing or Ecommerce that you want to check out stuff -
that's an issue. Right? Can you throw it?

Oh, there is another one. Oh, that will be art. Can you pass? I will throw for
you. Well, I'm not good at this.

Hi. I have one question about logging. For the immutable servers, we have to
keep the log, of the logs of the log files of the applications.

No, you should have event streams, um, log streams. So you should log, you
should not store anything, logs, anything in your machine.

So before the instance go down, like when it is de-scaling the system, so it
should send the log to the streams, ports, or it's like it's a cron job type of
thing like that will just upload the log files in a timely fashion manner.

Yeah, yeah, true. When I say terminate is not like brute force terminate an
instance. When you terminate the instance, you should have graceful, your
application should graceful terminate everything. So, and also in a PHP-FPM and
Nginx will not allow you, you can, you can configure it, to only terminate after
it's not receiving more traffic and stuff like that. Okay? So yeah, you should
be taking in consideration also um, finishing, finishing up the requests. If
it's a cron job - you should send a signal for your application to terminate
the, the workers and stuff like that. Yeah, there is some preparation for having
these kinds of things. That's why it's not that easy.

Thank you.

Anymore questions? One more here. Go ahead. Sorry!

Thank you. Are you... in my company we are using something similar and I was
wondering about the overhead. So you have a certain build in production of that
image and then of course you push some changes to the master replication and
then you get another image, but during the night when you sleep quietly, you'll
probably have to deploy the previous image. Right? The one that is in sync with
the others. You don't start the full deployment of the new version of the server
that you build. This get can get a lot of...

You should create probably a new cluster and when everything is okay - you
switch.

Yeah. So there's a lot of overhead involved here. We have a problems this way,
we always have to expand infrastructure.

Yeah, but that, but that's the beauty, but you can create a new cluster with all
the new instance and then just say: Okay, from now on I want the traffic to go
to this site and if everything is not okay, let's move the traffic back. Or just
say: Okay, let's try the new version, let's put just 10 percent of our traffic
to the new version and if it's okay, let's move everything. So, it gives you a
lot of flexibility, but yeah, also overhead,

Yeah, there's like a one... if you don't need the immutable server and maybe you
have to consider this when you start moving to this kind of deployment.

That's why I'm saying that you should start preparing for them, but you should
not go for them just because you want. You shouldn't go for them when you really
need, but you should prepare for them yesterday because you never know.

Any more questions? One more. I don't know. How many time do I have? I still
have time.

Thank you. Um, so this works for your application, of course, the immutable
servers. But what do you do with databases? Do you have a replication cluster?

The databases? It is a different thing. So, you can have a master/slave, you can
have clusters, you can have them in multi-regions, then some sort of sync -
that's up to you. But they need to be separated from your application.

Yeah, of course. So that is something to consider separately from... alright,
thank you.

Anymore? I think no. So, thank you very much for listening to me.
