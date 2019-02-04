# Building Global Web Apps with Multi-region Hosting

***TIP
SymfonyCon 2018 Presentation by [Jordi Boggiano](https://github.com/Seldaek)

This session will explore various setups and case studies from my attempts at hosting
sites used by global audiences.

There are many ways this can be achieved with different levels of success, budgets
and global-ness.

The talk will touch on Terraform, AWS, global DNS resolution and CDNs amongst other things.
***

Hello. So I hope you're having a good time so far, we're getting through the
first day slowly. So, I'm Jordi Boggiano and I want to talk today about hosting
applications across multiple regions or meaning geographically, across the whole
planet ideally.

## Hi Jordi!

Um, so just a quick word about myself. I've been doing internet things for quite
a while now. I've also been leading Composer and Packagist development, a bunch
of other open source projects. And for, let's say like work... kind of getting
money, you know, because one has to at some point, I work part time at
teamup.com and part time for private Packagist, which is kind of helping to pay
the composer development as well.

## Why Host Across Multiple Regions?

So the first question is why would you want to host something across multiple
regions? Because it, you know, it is definitely going to cause you some pain,
like it's not the easy way. I mean, obviously I think the main reason for me is
that you have used those across multiple places and latency tends to be like
painful. So, there was this video that came by the other day, I don't know if
you've seen it but it's like the effects of one and a half second of latency in
the real world, you know. I mean it kinda shows it's a funny, like a silly
example, but it's true that like a little bit of latency can really mess with
people and it just makes for a really terrible experience.

Other point is like if you have hosting across multiple regions, one single
region can go down and in theory, you know, you should have a resilient system
in it. You should be able to stay up even in case of kinda critical host
failures, which you know, these days it's like cloud infrastructure, it's like
all magical and nothing ever breaks in theory. But you know, sometimes things go
bad. Like it has happened that AWS has a complete region down even including
like, you know, they have this concept of availability zones which in theory
should be completely separated infrastructures. So in one single region you can
host like different availability zones. And then, they guarantee you somehow
that, you know, if one zone goes down, the others should stay up. This, in the
past hasn't always been true. So it's not quite enough if you really want to be
safe.

The third point is really, like most of us, I think are using CDN's for at least
delivering web assets. This is a fairly common practice and it's very easy.
Usually there is lots of tools and websites that help with that. But why don't,
why don't we do it for the rest, like if we saw that it is valuable to do it for
that. Why not the rest? So, why shouldn't you do it? I mean, I don't know: is
anyone here like hosting things like on a global scale or like more than one
region? It's kind of hard to see but I see a few hands. But like maybe three to
five percent I guess. So this is not so common. So why don't you do it? I don't
know. I mean, I came up with a few reasons why I didn't do it until recently.

## Reasons Why You Typically Avoid Multi-Region Hosting

Maybe they don't match exactly yours, but, it's just a kind of a lead of where
to go and where we could improve things. I think the first reason: we use the
database. Like having the database in multiple regions is a major pain. Like
this is really, like that's usually already a killer by itself. Like, if you
don't have like a multi-master setup, then it gets really complicated to have
synchronization across regions and if the master region goes down, then what
happens to the replicas? It's tricky, what can you do there? I think like using
one of those, you know, cloud databases from the get-go is probably a good idea.
I think the issue is like usually people start with MySQL on their computer, and
then everything is fine, but then five years down the line when you actually
need the global scale, it's like it's too late and it can't rewrite the entire
application. So this is a bit of an issue. Um, but like all the providers have
some solutions and then you have like MongoDB for example is, you know, it's
available no matter what platform you're using. Ah, there's another talk at the
moment in the other track if you are interested. Um, anyway, you know, there are
some solutions, I don't have a ton of experience with these so I don't wanna
dive in too much, but you know there are things that can help you, but for that,
usually you have to write the application really with this mindset from the very
beginning.

Um, yeah. What I found is that really like the cloud providers, they sell this
magical cloud thing but it's usually, actually doesn't help you that much. Like
for at least for this problematic, I found that there are some limitations which
are really annoying. Um, like one is Redis. It's just a simple example, but you
can replicate Redis for aws within one region, can have as many replicas as you
want, no problem. We cannot go beyond the regions. If you want to replicate in
another region, there is no way. Um, so, sure you can host your own Redis and,
you know, you can do things, but they just don't solve the problem for you.

Um, another issue we had, which thankfully fixed about a year ago was that so
this concept of VPC. I don't know how familiar you are with us, but just to
explain it really quick, it's like a, it's kind of like your private network
for, for all your infrastructure within aws within one region. And, so ideally
you want to keep things within the private network and only have one entry point
for like the web, you know, like the http port on one entry and that's it. Like
the rest shouldn't be reachable from the outside because that's just more secure
that way. Um, if you have things in multiple regions, usually they need to talk
to each other somehow. Like if you can't connect two VPC's to each other, that
means you need to open everything up on the internet and that's like, ehm, like
I don't really feel like trusting MySql or Postgres authentication to the
Internet. Like, it's just bad things have happened elsewhere in the past. I'd
rather not take the chance. So this thankfully has been fixed now so it's fine,
but it's actually like guided some of my decisions in the past, which, so that's
why I'm mentioning it.

And then finally I think awareness is also an issue in that most of the
developers they work with like fast internet connections, they're usually close
to their servers and so they just don't feel this pain of latency. Like it's
usually, we're not the ones experiencing the problem, it's more like users in,
on some random satellite connection or some far away country from your hosting
location. Ah, so I think the demand internally is not there usually.

## Case Studies

Um, yep, So that's that for the intro. Now I want to look at a couple of case
studies. And kind of really the idea is just to share, like, a few approaches I
took. I'm not saying this is like the ultimate solution and not trying to sell
you the silver bullets. It's just ideas that might help if you are attempting
this yourself because what I found is that there's not a lot of information on
how to do this stuff. Like I've looked at this for years and it's just, I
haven't found a lot of info. Usually those that do it, like mega corporations
with insane budgets and they can afford to have, you know, hundreds of Dev ops
engineers doing this stuff. Like for most small companies, that's just not an
option. So anyway, let's dive in.

## Case Study: Packagist.org

So first of all, I wanna, I wanna look at packagist.org, which I guess is
something you're all familiar with, so that kind of makes it a bit more
interesting, maybe. Um so there we have like just, just to look at what the
goals are with what we're trying to achieve, first of all, really high
reliability. Because if this goes down, the repository, like with all the
metadata composer just fails and things go bad very quickly. People tend to use
Twitter very quickly. So, it's like, it has to be up. That's, that's just a
reality. It also has to be simple, um, for different reasons, but I just like to
keep things simple if I can, because it's, you know, we have a very small team,
it's mostly me doing this. Um, so like, you know, this, again, like there's not
a hundred Dev ops people that can do like crazy infrastructure. It has to be a
simple solution that works well and that just also doesn't break because I can't
be like on the watch 24/7, so. It has to be global because we have really users
throughout the globe and ideally low cost because this being open source, well
there's just not a big budget.

So what did we end up with? Um, I kind of have a self-built CDN. So we have like
these primary servers that kind of generate files and all the metadata and then
we have a set of mirrors that are spread throughout the world. And those just
like synchronize from the primary. So the benefits from doing it ourselves
versus using some of the existing providers, uh, is mainly invalidation: because
we need really fast responses because when update, like the, they push
something, you know, they want to run composer update like 10 seconds later and
if it doesn't work, like if it doesn't find the new tag, they just pushed or
something, then they come and complain this needs to happen fast. Um, these
days, I think there are a few cdn providers which are actually, um, which offers
like invalidation to the extent that we need. So I'm looking at maybe switching
to, to one of them, but we'll see.

Um, then to kind of route people between the servers, like the replicas we have
Route53, which is the DNS solution of AWS, um, which basically does like a
latency-based routing. So it's like if you are close to this region you get
routed to one of these servers in that region. Then we have health checks on all
the servers. So if one of them goes down, or is not responsive anymore from AWS,
it just gets killed off, and people don't get routed there anymore. We have
like, you know, a few minutes of, DNS TTL. So it kind of, re-routes people
fairly quickly.

And like, again, like in terms of simplicity, it's really easy to set up a new
one. Like a few weeks ago we had this issue where, um, some of the mirrors were
just unreachable for some people. But it was like, the problem is it wasn't a
global failure. The servers themselves were fine, it was a routing issue on the
Internet somehow. So the health checks didn't pick that up and it was just, some
people were affected. I don't know, probably some of you in this room had some
problems. Um, but like, I mean people were saying, you know, I got, I got, I
tried at home and it was fine and in the office it's broken, like it was really
strange. So, at some point I just couldn't do anything really about the Internet
routing sadly, so I just a completely swapped these, these instances and like
created new ones in some other, some other regions and like it takes me like 20
minutes or so. It's quickly up to speed. So, um, so that's good.

Uh, so the setup kind of looks like this where we have like multiple replica
regions. And you see like for the... so the metadata is being pulled from the,
from the primary to the replicas, but the website is not. So the website we only
have it in one region still. That's just for simplicity, because yeah, I mean
it's a bit more latency for the website users that are far away, but that's just
a cost I was okay with. I mean that's a tradeoff you have to make. So when users
go to the website, this is just a proxy from the replicas to the main one.

So, what are the problems? Well, as I just mentioned it, this is only the
repository, not the website. So it's not a complete solution for sure, but it
solves the, let's say the high availability needs we have, that are mostly at
the repository level. Um, so that kind of solves the critical part. Another
thing that was kinda weird with this is like I had reports of files sometimes
like people will get a 404 when running composer. And it took me quite awhile to
figure out what was going on, but it's, the problem was the, like as we route
people like in a kind of round-robin fashion, like, whenever you resolve the DNS
you just go to this server or that server within one region, there was a race
condition actually, where one server could be up to speed with the latest
metadata but the other one not yet in one, one single region. And so one request
would go and hit one server and, like, it would get the filename to get next,
and will try and get it and will hit the second server that wasn't up to speed
and then you get a 404. and then, you know, if you retry it's fixed, like within
seconds, usually it was fixing itself. So it was kinda hard to debug why,
because every time someone reported it, I'm like: "I don't know, the file is
there, I don't see the problem". Um, so there's a simple proxy hack there where
it's just if the file is not there locally, it proxies it to the main region
again instead of returning a 404 and that kind of solves it. Um, just, you know,
little things that you don't necessarily think of when you, when you get
started.

## Case Study: teamup.com

So second use case, um, second case study is team up. So that's a, it's a
calendar application. So it's also used like pretty well, pretty much all around
the world.

Um, so what are the goals here again? Global audience: we kind of need to be
everywhere. Um, low latency just because it's not a good experience whenever
you're like clicking something if you need to wait half a second, it's just not
nice. The high reliability, I say full data access, obviously it's good if you
can like, you know, work fully with the application. But the critical really
critical part here is accessing the data because we've had times where we were
down in the past and people send us emails like completely freaking out because
their entire business was down. Like they just, somehow they use the calendar,
this one source of information for everything they have to do. Which is good, I
mean, it's quite fascinating to see all the use-cases there are out there, but
it's just, that means this is extremely critical infrastructure for a lot of
small businesses. And like yeah, we just felt really bad about like any downtime
because we're like: "oh my God, this is just someone out there is like sitting
in the office, like completely lost". Um, I mean it's the same when, you know,
when Github is down or something and don't laugh too much. Like we have the same
issues with some tools, right? Like industries, like different industries have
different bottlenecks. But, and again, one of the goals there was low
maintenance because we're a very small team, like only like four or five devs
and so it's, yeah, there's just not a lot of manpower to keep this going, so it
has to be somewhat self-sustainable and stable.

So what did we end up with? Um, so what we used wasTerraform. I don't know if
you're familiar with it, it's kind of like puppet or Ansible, but for like
setting up the, the infrastructure. So it's kind of a high level: just you
configure all the servers, all the VPCs, all the routing, all the, lots of
things. Um, and that allows you to automate things and that means it's pretty
good because if you need to add a new region, you can just like copy a few lines
and say ok, this, this one is now, like just load this all this config, but for
this region and serve that region and you're done. You just run it and creates
all the servers and everything.

Again, here we had to make some compromises with what is run where. So we have
the primary region that has everything like websites, databases, background
workers and all that. And then the replicas they have website, a database copy
but no workers.

Um, other kind of trade-off is we're storing the sessions in Redis. And as I
mentioned, you can't easily replicate across regions. We thought, well, I mean
actually people usually don't go from one region to the next, like, you know,
unless you are in a rocket or something, you don't transition from one region to
the next that quickly, that losing a session would be a problem. So we just
decided to have like local session buckets in every region and that's it. Like
there's no, there's no concept of global session. So the reads, like if you were
just looking at the data, this is handled locally in every single region. And
then, when you write something, um, we're talking to the primary database in the
primary region, uh, through this VPC peering because we built this early this
year. So this was available thankfully.

Um, so it looks something like this, with a primary region, some replica region.
As you see they're really the same apart from the workers: user comes, does a
GET request is handled locally, no problem. Um, if the user does a POST doing
some changes, we have the database writes going across. So we started with the
region, the primary being in the US west coast and a replica in Europe.

So I don't know if you can spot the problem there, but the result was something
like this. It was just not super fast. The reads were going fine of course,
because they were handled locally, but like the writes were just horribly slow.
Uh, so what happened? Like I felt really stupid when I realized this. I don't
know why I didn't think of that before, but obviously every Redis call or SQL
query that you do has to go from Europe to the US west coast, which means, yes,
somewhere around 100 milliseconds of latency. So, you know, typical pages maybe
running like 5, 10 queries. Well you multiply that by 100 milliseconds and you
quickly end up witha response time that's actually way worse than hitting the US
server directly.

So, yeah, it wasn't a very proud moment. But I thought okay, like I can see
there's some issues I can fix here. I was like, one of the problems is already
like establishing the connection. So a single, just doing a single query was
pretty bad because establishing the MySql connection, including TLS, means
usually like two round trips. So already just opening the connection, you're
like 200 milliseconds, then you send the query and then it's just, it was really
bad. So I thought okay, this proxy SQL I can use this to build connection pools,
then we will reuse the connections, so we don't have to have these round trips
every single time. So that helped, sure. I mean, it shaved off like these two
round trips. Um, but yeah, it was still really unworkable. Like some, some of
the pages were doing way too many requests and it was just, so way too many SQL
queries and so it was just not really working.

So we changed the approach and I'm like, I'm a very stubborn person, so I didn't
want to give up. So what we ended up with, I don't know if anyone else is doing
this, maybe it's a crazy solution. Like I haven't had a lot of feedback on this,
but feel free to let me know later. Uh, so we proxied the write. So if we see
the POST is coming in or like a DELETE request, we say okay, this is gonna do
some modifications on the primary database. So we don't want to handle this
locally on the replicas, we just proxy the entire request.

The problem is you can't really do this like at the Nginx level easily because
you don't have the sessions in the main region. So if you just proxy the request
in Nginx that's the easy way to do it, but then you're missing the session data
and so yeah, you just find yourself logged out when you're trying to do some
modifications that doesn't really work. Uh, so I implemented this in php
instead, and so in the application when we see requests coming in and it's a
POST or something that's going to modify the content, we say, okay, we'll just
take the local session, pack it up in a header forward everything including that
header. We also for the client IP, obviously with the usual, like "forwarded"
headers and so on.

To make sure that this is, you know, not possible to abuse because the problem
is, we're now like, unserializing session data from headers, which, you know,
it's not the best idea in terms of security or taking user content and like just
dumping it into the session and serializing it, like you want to be really
careful when you do things like that. Um, so definitely we do use HTTPS over the
proxy link. Uh, it's also going through the VPC peering for additional security,
and on top of that, we also sign the requests just to make sure that nobody can
inject a request that would kind of deserialize session data.

So with all of this, not too bad. Then if like if this, somehow the proxying
fails, then we will go back to actually dealing with the request locally as we
would otherwise, and we do the slow SQL request over the ocean and it takes a
while then to run, but at least it completes successfully. So, now it looks
something like this. Um, so if you do a POST, you come through and then we send
the same exact POST, but with some additional headers for the session, the
client IP and the signature.

I hope this makes sense. So results kinda like, you know, faster turtle for
sure, but still a turtle. So what happened here was like when I tried this, I
was like trying to disable the replica and just hit the US servers directly, and
I got and average of something like 120 milliseconds, like roundtrip time to,
for any kind of request response. It Was around this ballpark. So it's not
terrible because like, US is fairly a good link. Um, but then I noticed when I
used the replica on the reads, I got about 20 milliseconds. That was super fast,
hitting the local server, great. But when I did a POST, well I had this 20
milliseconds to reach the replica and then the replica internally would, for
some reason add 200 milliseconds. And I was like, what the hell? I don't
understand how it's much slower to execute from the replica within the AWS
network and everything, I would think this runs faster than me hitting the US.

Ao yeah, it's just turned out to be, actually the proxy every time had to open a
connection. Again you have this round trip time of like opening the SSL
connection. So what can we do here? Well, we can again do some connection
pooling. So, uh, so this time I added, a local Nginx proxy because we anyway
have Nginx running on the local machine. So we just, instead of proxying to the
US directly, we proxy to the local proxy, proxy in a way. Ok, it's getting
complicated. But, so that way will you, I mean, this adds really nothing in
terms of complexities like these ten lines of config in Nginx and like this
never fails, it's quite reliable. You just have to make sure that you do a few
things like this... oh, that's interesting. This laser pointer doesn't work at
all on the screen. Anyway, I shall use the mouse.

Um, so you want to make sure here that you set the HTTP version to 1.1 to make
sure that you have keep alive enabled, or you could use two workers, but I don't
think Nginx supports it for proxying. Um, and the other one you really need, is
this connection, overriding the connection header just in case the client has a
connection close in the request just to make sure it's gone. Um, and then you
set this keepalive on top.

Um, this kinda, it went well, but at first like I had very mixed results. Like I
was trying it and it would sometimes be fast so I had like sometimes the request
like a response within like 80 milliseconds. And then sometimes it was still
doing this 200 millisecond overhead and I was like, I just don't get it, like it
was really random. And then eventually I figured out that this keepalive 8, like
the way Nginx works, it actually just allocates these eight buckets and then you
have these worker processes in Nginx that says, you know, typically you set this
to like the amount of CPU cores you have or double that or something. So, on a
big server you maybe have like 16 or 32 of these Nginx processes and each of
them actually has eight connections. And so depending on which one you hit, you
hit one that has a connection open or not. And so like sometimes it was fast,
sometimes it wasn't. And so like, it's just the kind of things that make you
lose a lot of time for really small details. But I'm glad I understood where it
was coming from because it was kind of not making me feel good to have this
somewhat random result.

Um, okay. So what are the other problems that we're having with the solution
now? Uh, because with this, now it's consistently faster to hit like the
European server, this works really well. Um, the issue is while you're going
across the ocean and more, so there is always something that's going to fail.
Like there's always one request, that's gonna fail. Like if you do enough
requests, some of them are going to fail sometimes. So that's why I kept this
fallback step, so that in the worse case we can kind of handle it slower but at
least it's handled.

Other issue I had, and this one I didn't get to the bottom of, is the load
balancer on AWS sometimes times out these requests. And I just, I don't get it,
like looking at all the logs, it seems to be hitting the servers, gets a
response like within 20 milliseconds or so - it's not a timeout problem. But for
some reason it just gets stuck at some point. I can never get back. So
eventually we just had to decide to abandon the load balancer for the proxy
request and we just hit the EC2 machines directly, which is not ideal, but
that's just. like it, it actually works better in the end than with it.

Other challenges, as we are sending the session via the headers, session size is
then limited because the header size has a limit. Um, so this may or may not be
an issue for you, like we actually really hardly use the session, so it's really
just for marking the user as logged in or not. So there's very little data in
it. So this doesn't really hurt us, but, ya know, if you're storing like tons of
stuff in the session, it definitely might be preventing this whole thing from
flying.

So in the end we got to this point where it's like super fast. Um, so what are
the downsides though? Just to quickly recap. In case the primary is down, we're
still like in read only state in all the replicas. Like that's just something
you can't really fix unless you have multi-master setup. I don't think we're
getting there, like with our current team size. Like, AWS announced I think last
year that they will at some point release a multi master Aurora, which is like
this kind of AWS implemented the version of MySQL and Postgres and whatnot. I
don't think this is out there yet, but I'm not sure. I haven't checked in a
while. I also don't understand how they can possibly guarantee this will work. I
can't just conceptually, it doesn't make any sense to me, but they have very
smart people so maybe they figured out a way. It would be really amazing if we
could just like switch from using MySQL or Postgres to using like a multi-master
MySQL or Postgres, like by just pressing a button, that would be great. But
we'll see.

Um, the workers, I mentioned they're only in the primary region, it just makes
sense for latency reasons: those are writing lots of stuff. We just don't want
to have them all over the place. It's a danger, yes, like if the whole region is
down, that means the workers are down, but it's just something we have to live
with. Other downside, definitely higher complexity than, you know, the other
solution. But it also does a lot more. So I think it pays off for us.

So the end result of this kind of infrastructure: that's the history of all
uptime, like from February 14, 2014 to now. We had lots of really bad months
where it was like down to 99 percent uptime time. Like this is really bad. I
don't remember the numbers but this is like really, really bad. And now in the
last 9 months or so since we migrated, we have something that's more up to like
9, like four 9's almost. There was a glitch there in July, not sure what
anymore, but otherwise it's really been super stable and we're really happy with
this.

## Summary

Ok, so just to sum up quickly, I think really this you have to take on a case by
case approach. The first point is to look at the audience location obviously. I
mean if you're doing some website that's only for like German users, you know,
sure you want to probably host it in Germany or somewhere nearby, and you don't
need to have like some, some region in Sydney because yeah, it just doesn't make
any sense. So that's a per-project thing. I don't know, depends on what you're
working on.

All the other issue is really: what are the requirements in terms of latency,
like what's okay latency for you. Like, if you know, if you think taking
something and you get a response like 300-400 milliseconds later is fine, then
that's your number, right? Like you gotta look at what you want to achieve. Like
we wanted to really try and bring this to kind of snappy, like instant feels, so
you kind of, it's like, it's like a desktop app kind of. Um, but yeah, that's,
that's again something you need to evaluate for yourself.

Then I think one of the big factor in like deciding how complex you go is the
team size. Because I think anything is possible but like some solutions require
really big teams and that's not always available. Then finally like the tech
stack again, like go for a cloud database or wait for Amazon to solve physics
and just come up with this magical database, that will do everything great. And
that's that. Thank you very much.

## Questions

I think we have like one and a half minutes for questions unless I got the time
wrong, so I'm not sure if there are any questions. Yes. I don't know if we do
microphones here or not. Okay: just shout, I'll repeat it.

> So why are we only running the workers on the primary region?

Uh, because otherwise you would have this latency, like let's say, as they run
in the background anyway, like having them near the user has no benefits. Ok, I
can't hear you anymore sorry. Let's discuss this later. Anyway, yeah. So that's,
that. Enjoy the rest of the conference. Thank you very much. If you have
questions, please come by.
