# File Storage for Modern PHP Applications

***TIP
SymfonyCon 2018 Presentation by [Frank de Jonge](https://github.com/frankdejonge)

Many PHP applications need to store files. There are many different reasons to
store file, and not all filesystems are created equally. How can we identify
our needs? Which filesystem is best for your use-case? How can we ensure our
future needs are not blocked by the code we write today? In this talk we'll
explore how filesystem abstraction and general coding guidelines can improve
our application's and make them future proof!
***

Hi everybody. Today I'll be talking about file storage for modern PHP
applications. My name is Frank. I've got a couple more people dropping in. So
thank you all for joining this talk. The other two talk slots are one improving
workflow and the other is a good number of best practices for awesome,
testability. But somehow you've managed to find the room where we're talking
about file systems.

## Who is this Frank Guy?

Uh, I'm a freelance and a free software developer. That means I work at
companies now. I can't name the company that I worked for, but it's the biggest
airport in the region of Amsterdam. So if that gives you any indication then
uh, then you know where I work. Uh, so, uh, as in terms of free software
development, I like to contribute to open source, for Symfony because this is
the Symfony conference. I thought I'd highlight some of the things that I did.
So you may be familiar with who I am. Who remembers their routing speed up in
3.2. I did, I had a lot of fun coding it up. And then Nicolas came along and
basically ripped out all my code. Uh, so thank you Nicolas for that. Um, but
then I thought, okay, I really liked this routing component. So for a 4.1 I
added the internationalized routing. Who here uses that One?

Okay. So the rest of you are just stuck with code that I wrote that nobody
uses. For other stuff that I did. Are there any people who like to do
event-sourcing here, right? So if you like event sourcing, I created EventSauce
in over the course of last year as sort of alternative way to approach event
sourcing that's a little bit smaller and possibly more easy to understand. So
if you are into that, maybe check it out.

## Hello Flysystem

But for the majority of my time I've been working on Flysystem, which is a
package of the PHP league. Who here knows the PHP league? Okay. Decent number
of people here. Who here used Flysystem itself? That's the same people. All
right. So, um, I started five years ago and I started working on this
presentation like a couple months ago. And at that time we just hit the 50
million installs mark, which was pretty great, and now we're already close to
55 so it's going quite fast as you can see, it's a, it keeps on growing,
growing, growing, and for the most part, working on Flysystem has been a pretty
good experience, uh, but also you get to basically see all the stuff that
people do with your open source work and also what they do in real life.

And sometimes it's a, it's a bit scary sometimes it's a little bit depressing
and when it's depressing, it's mostly PHP because or FTP because FTP is not so
nice. Um, but for today we'll be talking about file storage for modern PHP
applications. A note of warning this talk may not be exactly what you expect it
to be, but I hope you can keep an open mind in the things that I'm gonna
propose because they're going to be a mix of different things, both code and
not so much code things. Um, so, uh, if we're talking about file storage for
modern PHP applications, what should we do? Well, it depends and that's the
problem. Uh, so today I'll be talking about a few different subjects. One is
just to get a generic gist of what file systems are, what types we have, so on,
so forth, um, how they interact with modern applications, what is involved in
the process of choosing a file system and how you consume filesystems in code.

## The Problem: Who put these Files Here? Are they Used? What!?

So this talk is not so much how to do it right it's more about how not to do it
wrong or even if we're doing it wrong, how to do it wrong the right way. So,
okay, you're with me. Uh, so what's a filesystem? And I can already hear you
say "lol don't care". It's something that we have everywhere and when I see
this, when I see this, everybody looks at Buzz but I tend to feel a little bit
like Woody, because filesystems is one of the things that we sort of all have
problems with or something is always full disks or I don't know what to choose,
but it's also one of the things like nobody really takes it into account as a
decision process. It's like, oh, we have this, we'll use it. We don't tend to
think about it anymore, but we do have a lot of common problems, common
problems like the unstructured filesystem directory.

Who here has or has had one of those unstructured uploads directories. Alright.
Uh, let's, let's keep the hands up. Let's see if we can get all the hands up at
the end of the questions. Um, who's ever had the question like, are these files
used? Okay, can I delete these files? All right. Where did these files come
from? Like, okay, people already put their hands down. That's okay, let's go,
let's keep them down from now. It's like if we ask ourselves all these
questions, we're pretty sure we don't exactly know how to clean this stuff up
anymore. And then you can get into a situation like this. It's like that's,
that's basically what it boils down to. Right? So, and other questions like can
we change filesystems? Like sometimes there's reasons to do that but it's not
always possible. So for me it always looks like, okay, this is fine, but is it
fine?

Well, I don't think so. And I think mostly it has to do with the feedback loop
when we're consuming this stuff, we've got a timeline, and in this timeline we
start out, we're going to create a feature. We've, uh, we've got it laid out,
but all the knowledge that we really have is assumptions, then we create the
feature and now we have experience with some tools and we put it in production
and we get some initial usage. Uh, we get feature adoptions from our user and
this is where the actual usage start, but also maybe when the actual problems
start and then after awhile we detect the problems and we're going to solve
them, then we're going to get actual experience. I really believe this is
really a sort of a sort of depressing thing because it means that the
information that we needed to have in order to make the right decision at the
start is at the end.

## What is a Filesystem?

So we're sort of doomed to fail. So, how can we make the wrong decision the
right way? But before we go into that, let's first briefly go into filesystems.
So what is the filesystem actually? Well, a filesystem is a system that
controls how data is stored and retrieved. I've highlighted the word system
there because it feels like there's something in there. So what is the system?
Well, system is a collection of parts conventions and interactions forming a
complex whole. Okay, so if we expand, that's what a system is into the
definition of what a filesystem is. What did we get? Well we get this. A
filesystem is a system that provides a collection of parts, conventions and
interactions, forming a complex whole that controls how data is stored and
retrieved. So in itself, although we take it for granted, a filesystem is
already pretty complex. The definition itself already is pretty complex, so if
we dumb it down a little, we have something you use to store and retrieve data.
So it's something that we use, not something that we have. If we're talking
about a filesystem, we're not talking about necessarily the storage units, for
example.

## Filesystem Types: Nested versus Linear

Then if look at the different types of filesystems that we have. There are many
different types, but in a broad sense like how it affects us, there are two
types, that we have to deal with. One is the Nested filesystem, that's what you
have in your finder and if you're using or if you're one of the 12 people that
use windows at this conference, you have something else. Um, but there's also a
Linear filesystems. Who here has used the Linear filesystem? Okay. Who, who
here has used anything in the cloud? Okay. This is same thing normally you see
so more people then raising their hands. And most things in the cloud are
actually Linear filesystems. So if you have something like S3 or Azure blob
storage, that's basically a linear filesystem and what's, what's the
difference? So Nested filesystems are very common.

Your desktop has one, it has directories which have files and more directories.
On the other hand, Linear filesystems are like key value stores. So instead of
directories, you just have full paths, they might have separators that look the
same as a directory separators. But it's really just a sort of key value
system. They only have files and if you know that, then, if you say what's
common between Linear filesystems and Nested filesystem? So it's basically only
the files, right? So what you can say is that directories don't really exist if
you look at the common denominators. So, uh, they require a different approach
and they're very cloud.

## Filesystems in the Wild: NAS, CDN & More

So if we're looking at filesystem offering we see, or at least I saw something
interesting. There's a bunch in the group that is file system, then that's a
network attached storage. So if you're on AWS, you have a volume. If you're on
a Digital Ocean, you have like the blob storage stuff that you can also mount
into your VPSs. There's stuff like Gluster FS and Ceph and I think, I don't
know if there somebody from the SymfonyCloud in here, but I think they're using
Ceph. I see nobody from the Symfony. Okay. That's okay. I think, uh, that's the
case. There's another group and that's basically filesystem and CDN. Then you
get things like AWS S3, Digital Ocean Spaces stuff, uh, Rackspace Cloud Files
and Google Cloud Storage, all those kinds of things. That is the filesystem
plus CDN, and then you've got more and that's more filesystem plus sharing. So
Dropbox, Box, Google drive, Microsoft OneDrive, those kinds of things. And then
you have things where you can put files in but they're not really supposed to
be filesystems but they happen to support it.

So like WebDAV, which is something different entirely. It's more the protocol
of interacting with somebody er something over HTTP rather than being a
filesystem. You have MongoDB GridFS, and I think I've got a MongoDB engineer in
the room in the back... Oh, I've got two. And you can also use something like
Postgres Large Object Storage. But if we flip this around, we see all those
things and then we say, well, what's the inverse of this?

# What is not a Filesystem?

What is not a filesystem? Well, a filesystem is not one of those CDN's. So a
content distribution network is not a filesystem. It's something that uses a
filesystem, but it is not a filesystem and the same goes for all these other
things like a web server is not a filesystem, file managers like something that
operates on. So there's a lot of things in this space that sort of overlaps
with filesystems and they supply the same offering as filesystems, but they are
not filesystems.

So for me that is sort of a problem. If we look at one thing and that one thing
has two concerns, we tend to obfuscate those concerns and view it as one and
that might induce misuse of a, of a product or in the use case that we have.

## Stop Making your Filesystem do 100 Things

Ryan Singer in 2010 said this. So much complexity in software comes from trying
to make one thing do two things. And I wholeheartedly agree with this. Uh, I
don't know who knows the statement, but this is one that comes back at least
once a month for me. It's like, okay, I should dumb this down. It'll be better.

So my version of this is stop trying to make a filesystem do 100 things and
that's also sometimes what I see when you, when people use Flysystem, they want
to use it for access control. They want signed URLs for different things. They
want to expose it over HTTP. And all those things. They don't really belong
with filesystems.

## What a Filesystem Does: CRUDLI

If you look at filesystems, they're basically CRUDish, but I've let some space
open at the bottom because you've got writing files, reading files, updating
the contents of files or deleting file that's pretty CRUD. But there's also
these two other things like you've got inspect and list. So inspect is more
like getting the mime type of a file, listing is like getting what is in a
directory. So it's basically CRUDLI not yet a term, but I've coined here now...
So, if you use it now, you will have to pay me money.

## Filesystems in PHP: woh

Um, so if we use, if you look at filesystems and PHP, I'm just gonna take a
little sip water. Then if we look at the usage, it's, it started out it starts
off pretty easy so we can very easily write to a file. We can read from a file,
we can delete it, although now it's called unlink rather than delete and we can
get the modified time of a file using mtime and that's an abbreviation of half
the words. Maybe a bit confusing of them. We have things that are related to
file info and that already gets really confusing because you now have to
instantiate an object with the type of thing that you want to retrieve from it
and pass it a location to the file methods in order to get the mime type. So
pretty confusing already, but it gets more confusing because if you want to get
the directory listing for a particular directory, what do you use well you've
got glob, scandir and readdir, which takes a resource which it needs to
retrieve from opendir.

Or, you can use the spl directory iterators which are awesome, but you get all
these other types of objects back that you can't really serialize or anything.
So you need to do some normalization. And like which one of these is better? As
it turns out the last one is the most performant for the most cases in the most
general thing. But who will remember typing that out every time? So like WAT.
Oh, okay. So then, uh, if you're, uh, handling big files, you have to uh, use
functions like fopen and fread and fwrite. Who here is comfortable using those
functions? Okay. So definitely less than a quarter of you just raised their
hands. Well these are pretty essential for, for the most part. And this is a
relatively simple example where you basically, this is a copy, but then in php
rather than just saying copy, but this does not handle all the error cases.

If you look at this and handling all the error cases, it actually looks like
this. So it becomes very complex and very verbose and it's a lot different from
a file_put_contents. Right? So, so WAT, why can't you just use copy? Well,
copy, if we're looking at filesystem a copy just works locally. Those stream
things, they might work for other things as well.

## What does Flysystem do?

So as I mentioned for the last couple of years, I've been working on Flysystem.
So what does Flysystem give you in order to help you with this? Well, it gives
you a uniform API, which means this is how you interact with the filesystem
using Flysystem. You create an adapter, you used that in a filesystem and then
you use the filesystem to write. You never use the adapter directly. That's a,
yeah, it's part of the pattern, adaptive pattern. Uh, it's really applicable
here and you basically have basic operations for writing, reading, updating and
many more things. Uh, but it also means that if you want to change, for
example, to the AWS S3, all you have to do is change the configuration and all
the usage stays the same. So that's pretty powerful. And if you are one of the
people that have to use FTP, you don't need to deal with it anymore. As I said,
I've taken the pain for you. You can just use this and get done with the job.

## Streaming

The stream part in Flysystem allows you to remove a whole bunch of that
conditional code. Basically you can, get a stream from a remote filesystem, uh,
and pass that to a local one and if you'd need to, for example, change
something that's on a remote filesystem, you can first download it like this,
do some stuff, and then re upload.

This is a common pattern that you'll see when you're moving to remote file
systems because a lot of the functionality that we need in order to operate on
files require your files to be local first. For example, if you want to
manipulate images, they need to be in your local dist. They can't use a stream
from an external service. So this is something that you need to keep in mind.
But that's, this is not so much code in order for you to get this working. So
it's okay. Um, this also means that you can use two remote services to
basically pipe so many information through. So here we're reading from Azure
and we're writing to AWS and your thing to notice is because we're using
string, we're only using a small portion of our memory at a given time, so this
could be a re uploading from Azure to AWS, like a multi gigabyte file, uh, but
even if your local php installation is limited to like the default amount of
memory this won't run out of memory. So that's pretty powerful.

## Configuration Root & Relative Paths

The other part that Flysystem gives is relative paths. Everything in Flysystem
has relative paths. And what I mean with relative paths is the following. You
have a configuration root, so if you have a filesystem and it's confined to
stay in a root and every location that you provided is a relative path from
that root. And it always needs to reside there. This means that if you change
the root, the configuration changes, but you're how you're using it won't
change. So this is a pretty powerful concept because it's a very strong
encapsulation principle. So if you're using the local file system and you have
that first path, so it's in /configuration/, that's where you store all your
files and that's just basically how you set up your services. The configuration
is there, and then when you use it in your other services or inside your
services or maybe directly in your control or whatever you like, you can
basically use the relative path and Flysystem will make sure that they're
combined and actually written to the appropriate path.

This means if you for some reason need to change the location of your files.
For example, if you're running out of disk space and they attach a new network
attached storage that is bigger or automatically responding or whatever fancy
thing they've got configured for you. All you do is change the configuration.
This means all your consuming code will forever remain the same and that's
pretty powerful, but you can take this even further because it even applies the
same concept over multiple filesystems. So if you're moving from local to
something that is in AWS, all you're consuming code will never have to change.

And this, this is pretty powerful because it prevents vendor locking.
Basically, if all your code knows is that all the files are going to AWS and
you want to switch to something else, but now you have to basically refactor
half of your codebase. Uh, I don't know any manager who is like, yeah, yeah,
sure, spend the hours on that, that's amazing. They're going to prioritize
something else, but what if that expense is lower, close to zero, then you've
got a better opportunity to actually push for that change and reap more
benefits from that move. So it is better for isolation, better for
encapsulation because all the responses are exactly the same from every
adapter. It helps with predictability and that helps to stabilize software. Uh,
and in the end that makes sure that everything is super portable.

## When to use, or not use Flysystem

But don't use Flysystem for anything. If you're using local only, for example,
for templating engine looking up files or a, if you want funky things with
symlinks, then you will only ever do that on your local filesystem. So don't
use Flysystem for that. Then it's more of an overhead, you'll be limited.
Because the limitations are functional for file storage, but for at least local
specific filesystem only things, you'll just be limited. So keep that in mind.
So if we're then going into the process of choosing another filesystem, we
often use the FEYNMAN problem solving algorithm and actually this is more than
people do already. There the algorithms says write down the problem. Well
mostly people just think of the problem and then they think very hard and then
they come up with a solution. But I don't, I don't think that works very well.

If we have somebody who uses our application and they want to upload a file,
um, we the browser has to file uploads through us and then the file is there.
If you want to use the local file system for that, that's fine. Still many
people who are developing or deploying applications, deploy to one machine
only. So if you put a well configured nginx configuration for that one web
server, you basically have a CDN and there's a lot less hassle and moving
around, that you have to do so. It's always good to know if you're not
restricted by this, you can use this option, so like the local file system
stuff is the majority of the stuff and then we are in the cloud space. And when
do you want to go to the cloud space? Well that's mostly when you have that
image that you want to upload, but instead of one web server you now have three.

And if you upload the picture to one and somebody else wants to retrieve the
picture, well how do they know they're going to get it from that one server?
Probably there's a load balancer in between, but that should be not sticky and
not knowing of the implementation of the web servers below. So the generic
solution for this is pretty simple actually. You just make sure they share
storage. So you can do this in various ways. Um, obviously the file goes there.
Um, so you can do this either by using a service or using something that shares
the filesystem across many things. So you can use a clustered type of file
system, like Lustre fs or something else in order to orchestrate this. So you
need to know that the storage now becomes a separate entity that multiple,
instances of your application are using this singular entity and whether, if
it's a mounted filesystem or it's a service actually doesn't matter. So why do
you want to do it as well if you want to scale horizontally, just as a quick
poll. Who here scales horizontally their applications?

Okay. The other way around. Who deploys their application to one server?

Okay. So there's, I didn't see all the hands going up or down, but it's ok.
Still a good number of people have that. If you don't need to do that, you may
also not need to bother with the extra filesystem expenses because they do come
in expense. But if you do need to do that, we want stateless apps or in some
cases we don't want to serve the files ourselves. So we want something like a
CDN maybe for geographical reasons or other, uh, or we maybe want to share the
files across multiple applications. And I think when we then, going through the
process of choosing a filesystem, we're not asking direct questions. Mostly, if
we're asking the questions were asked like, do they need to be on premise or
can they be in the cloud aka still somebody's computer. But I think more
important things to take into account is, for example, the question of who will
do the maintenance?

If you've got a file system locally, you also need somebody who manages that.
If you're using a service that's can basically scale infinitely like S3 or
Azure blob storage, you don't have to think about it anymore, but that might
also come at a cost at a later time when you just keep growing in your expenses
go up. But that's a different problem. Hiring An, uh, an operations person to
manage a cluster of filesystems, uh, it's not an easy task. So you need to
think about that as an organization and as a developer, do you want to spend
your time doing that? Or do you want to spend your time writing code and just
consuming. But also you need to think about how we will use the files. Do we
have particular needs that the file needs to have? You can serve, um, for
example, videos from your local web server, but there are also specialized
services where you can upload your files and they are like a media service that
will do all the bit rates stuff for you.

So you've got a specialized more capable file storage solution that you can
just use. You can also ask why are we storing things in the first place? Uh,
there are a lot of cases still where people upload CSV or excel files and they
extract data from it and then they use that. They store it in the database, but
they will also keep the file separately. And do you have rules about how long
we keep that? How long are we going to keep those files? How big will the files
be? All these sort of things, questions need to trigger you into finding out
what the needs of your use cases. So it's really good to distinguish needs and
capabilities. Needs is what you require from a solution and a capability is
something that something can do. So for example, S3 can be used as a CDN, but
if you don't have those needs, maybe you don't need to use S3.

## Finding the right Filesystem

But now if we've asked all these questions, we need to somehow document this.
So I think we need to write stuff down. But if you're then looking at the agile
manifesto, we often see this, well, working software over comprehensive
documentation. And what developer then thinks is, haha I don't need to write
documentation. I would argue the difference. Um, and I had a conversation with
Rafael Dohms about this and Rafael Dohms, is basically he's a Brazilian. That
means he talks in Portuguese but then talks about barbecues a lot. Um, I think
that's correct. Correct me if I'm wrong, and he reminded me of a technique
called ADR. ADR is Architecture Decision Records. And in this format you can
store all this collective knowledge. And this consists of a couple parts. One
is context. The context is something that influences the decision, um, that is
something that is there, that's a given.

For example, the team uses AWS. If the team uses AWS, it's more likely they
will choose for S3 and if the team uses Azure, it's more likely that they will
use something like Azure blob storage. And this is a logical decision, but it's
good to keep that in mind. Um, for accountability purposes. Um, another context
might be we expect massive storage growth. This might be something to think
about, like, do I want a infinitely scalable file system? If you are expecting
a large number of files, it might be a smart thing to think about that from the
get go. Another requirement would be that we need to store things on premise or
data must be stored in the EU. Uh, do, do you know this for the files? Do you
ask these questions? Like you, we should be asking these questions and then
we'll also document the decision.

So what is the decision that we made according to the context and the needs
that we had? Well we'll use AWS S3 or Blob Storage or whatever, but then
there's another part and the other part is the consequences. So if we're moving
from local to something remote, we have consequences in our code. So we need to
change our code accordingly. This is a consequence of the choice that we made
and by having such a document, you also have something that you can communicate
with your team leads, but also to management in order to say, well, we see
these conditions and we made this recommendation. Now we as a company can make
this decision. So you make people accountable with you for the decisions that
you've made. So if later in time your needs have changed and now you're not the
developer who made the bad decision, no, you and your manager both made an
agreement that this was the right choice at the time. So you actually have,
something to motivate your manager in order to make the new right decision with
you without it coming at your expense. So I think that's a really, it's more
political than anything, but it's really a good thing to think about.

## Use-Case Driven Storage Models

So now we've chosen something and now we can actually create an implementation.
So I would argue that the best thing that you can do for any application is to
create Use-Case Driven Storage Models. And what do they look like? Well, they
can look like an interface, so the interface could be ProfilePictureStorage and
we'll just have a store function that will accept the user and their contents
or in this case a resource containing the content. So profile picture or for
example, a financial report. Same thing and how does this look internally? Well
internally, it looks something like this. We've got an Filesystem Financial
Report Storage which implements the Financial Report Storage interface and it
uses the file system internally to dispatch. And this is a really powerful
thing because this makes sure that the path created, which is a relative path,
so easy to move around, will always be created here. We don't have multiple
areas where we're operating on the same part of the filesystem. So we've got an
encapsulation of our Use-Case in this thing. Just to reiterate, it's really
good to isolate your use case. Um, so we now have consumed this and now what
happens when stuff goes bad?

## What Happens when Stuff goes Bad

So who here knows of the concept of, or the thing called (Blameless)
Post-Mortem? Okay. Who has a meeting after something like a production incident
went down? Okay. That's way too few hands. So like if we look at this, if we
know that we need the information from the end in order to make the decision,
uh, back, uh, or at the front. We need some sort of feedback mechanism. So what
I propose is to document that as well. At least there's a sort of reference for
the, for the future. So the next time you'll see, well we have these problems
and instead of relying on gut feeling, you actually have a record of what
happened. And what this needs to contain is what actually happened. For
example, the storage was unavailable or what caused it. For example, a full
disk, what did we do to fix it? Well, sometimes this can be the dirty thing you
needed to do in order to go like ssh on the production machine and manually
deleting files that you no longer need. Or even deleting files from a different
part of the application in order to free up more time for more space for the,
for, for the problematic area.

This is something that can happen. And, but more importantly, you can also
document how you could have prevented it. If you're looking at your way of
working, some of you might be working agile. If you're bothered by this a lot
and you can give priority to these kinds of solutions, you automatically have a
very good justification to tackle technical debt in your, workflow as part of
your workflow and with probably with consent of management because they saw
what kind of pain they and the business endured and what kind of solution you
can provide.

## Summarizing!

So to round things up, use Flysystem. Keep things simple. Use a filesystem for
what it's for and use the other capabilities if you want but always keep that
file storage concern, isolated. Document your requirements, document your
decisions using ADR. Create specific storage classes per use case and create
use generic solutions for generic problems and use special solutions for
special problems. And just to finish up and see how weird things can be, um,
for example, you can use ceph and ceph also has a way to talk over HTTP and it
actually implements the S3 protocol so you can use your AWS adapter to use to
talk to ceph and ceph can be installed locally so you can be your own S3 if you
need to have stuff on premise.

This might be an interesting solution for you if you're heavily invested in S3,
but need to move stuff back on prem. You can also use mongoDB as a file
storage, but for very, very specific use case. For example, you can have a, geo
distributed cluster of mongoDB nodes and then you can use the file storage
component of that. So the files will be near where they are used. So for sort
of file sharing, file transferring, geo where use cases, you could use
something like that. For more cluster stuff, you can use Gluster FS and you can
use or have a Gluster FS cluster running on Kubernetes and if you have Gluster
FS running there, you can also create a persistent volume on top of that
Gluster FS cluster and Kubernetes. And then you can have a service on
Kubernetes that consumes a persistent volume on Gluster FS on Kubernetes. So
that becomes really complex and as you can see, a, it's now a arrow up, which
coincidentally also is the direction of where the complexity is heading or
maybe not do that.

So last question, what do I do? Well, it still depends, but I think with all
these questions and all these things that you now can think about, I think we
are able to make the wrong decision the right way. I hope I have convinced you
and if not, I hope you enjoyed this talk. Thank you for listening.
