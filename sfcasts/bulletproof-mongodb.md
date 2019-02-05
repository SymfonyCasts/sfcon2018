# Bulletproof MongoDB

***TIP
SymfonyCon 2018 Presentation by [Jeremy Mikola](https://github.com/jmikola)

An all-too-common approach for database error handling is to log the exception,
return a 500 response, and move on to the next request. MongoDB and its PHP driver
have an array of features that can greatly improve an application's resiliency in
the face of unexpected errors. This talk will examine how the driver monitors connections
to a cluster and look at how we can tune its behavior to meet an application's unique
needs. We'll also demonstrate how PHP applications can take advantage of newer features
such as retryable writes and multi-document transactions to guarantee ACID data
integrity without having to fall back to PDO and an SQL database.
***

Hello, my name is Jeremy Mikola. We have 40 minutes and 58 slides, so I will try
to be brief and there probably will not be time for questions. I will be outside
and catch me before tomorrow, etc.

## Hey Jeremy

Um, welcome. A little bit about myself. I'll make this very brief. I've been in
the Symfony community for a while. I missed Cluj last year, but I've been to
most of the other conferences. Later join me on the main stage for Jeopardy. I
work for MongoDB, it's been about six and a half years now, which is longer than
I expected it to be. It's still taking good care of me and they keep giving me
more clothes, so I'm happy there. We might have a WurstCon tomorrow, but these
days, I work less with Symfony. I work on the MongoDB driver and Doctrine.
Andreas is here and there's some other folks from Doctrine here I think this
weekend. So that's mainly what I work on.

## Topics to Cover

Um, I did not get fired for this. This is MongoDB snapchat for databases. Uh,
this was a few years ago. I'm still employed, happily. And so just an overview
of what I want to talk about today. The subject was a little strange with
bulletproof MongoDB. But I want, kind of a best practices talk and talk about
some of the newer features that take us away from, whereas a few years ago this
was maybe more relevant that the criticisms to MongoDB, but there's more new
features that give it more maturity and stability. So I want to cover.

So I'm going to start by just a show of hands. How many people are familiar with
using MongoDB in production or playing around with it first. Okay, great. That's
about half. So we'll talk a little bit about deployment models. I think for
development environments, a lot of us use just a single `mongod` process, we
don't use clusters or anything. But talk a little bit about that because of the
better concepts definitely work with clusters. Talk a little bit about how the
driver works specifically the php driver, which makes a lot of, have to do
special behaviors because it's, we don't have threads, we don't have a lot of
the other features that things like Java and Python have. And then talk about
designing applications and the three subjects there are some concepts that you
can apply to your application, how, what are the best practices for error
handling and how do we take advantage of transactions, which is a newer feature
within the last year of having actual ACID transactions that you'd find in a
relational database.

## The Presentation Abruptly Ends

That's the talk. Please leave feedback on joind.in. Is this going to happen
regularly or is there a presentation mode?

Yeah, that's what happens if he doesn't go through the slides quickly. That's a
good password. Yep. There we go. We're good? Yeah. Well, I think we're good.
Yeah, disable the screensaver. So if you want to know what to do (inaudible).
Sorry about that.

## And Starts Again: Setups

Thanks Andreas. So the basic is a standalone, this is just running one `mongod`.
This is a good development environment, my production websites, I'll just use
this because I have everything on one linode. This is: everything stands alone.
You don't want to do this in production.

The two main things that you would want to deploy your database is one with
replication, this should at least have three MongoD processes on different
machines and your client and driver is talking to the, um, in a cluster, you
have one that is the one that you can write to, and usually you also read from,
and then the secondaries are constantly replicating the same data. And they're
also communicating with each other to keep aware if one of them disappears. So
if the primary server was to crash or fail over, you take it down, someone
unplugs the network cable, you still have two out of three and they can take
over. And one of them will become the new primary. And your application goes
from talking to this primary to talking to the next one.

And so those are, when we talk about failures and recovering from failures, this
is a very common thing where a machine goes down or you're doing maintenance or
things like that. Uh, and then more advanced, which I expect maybe less people
are using is with a shard cluster. And this is where you have your app servers
and you would have another cluster of replica sets, another three that store the
configuration for where data is stored here. And then similar to sharding in a
SQL database, you'll have your data split across, sorry, you'll have your data
split across multiple shards.

And typically in this case, you might have a very large collection say... where
is it... your user accounts maybe a smaller collection, but your big collection
may be all the photos they're posting all the tweets that they're posting. So
you would split that across multiple shards. And this would be horizontal
scaling, it was discussed in some other talks earlier today. But that's, you're
spreading out instead of making a bigger Amazon instance or scaling vertically.
And because you can have hundreds or thousands of shards, we have another
process that kind of acts like a router so that the driver does not have to, the
php driver does not have to open up, in this case, this is a comically large php
app where you have fpm workers and they all put in sockets. So you want to avoid
the case where we have thousands of connections being opened by every php
process. So there we would go through a routing process.

## Driver Internals

So those are the two basic deployment models: sharded cluster and then a replica
set. And they leverage each other. So sharding, uses replication internally. And
then if you're a smaller application, you don't need to scale out, you would
just use replication. But the driver different than, you say that the PDO MySQL
driver, it will typically talk to one server at a time, whereas most of the
MongoDB drivers are responsible for doing, talking to the entire cluster of
database servers. And this is also different from, if a driver, if anyone uses
Cassandra from the last presentation, I remember going to Cassandra, there the
driver talks to one node and that talks to everyone else. So in most other
databases, the drivers talk to a single database node at a time.

But taking just a dive through what the PHP driver does when you use it. So we
start with a connection string, which is similar to if use Doctrine with a
relational database there's a DSN. This is very similar. We have specifications
that tell us how to parse this. So it kind of looks like an http url. We have
host identifiers, so maybe this case it's one server, but if it's a cluster, it
could be comma with multiple servers that you might talk to. And there is
additional and DNS if anyone's familiar with SRV records. And this is something
that was added recently, instead of having to maintain constantly changing
connection strings, if you add more nodes to your cluster, we can use DNS to
keep track of that for us, which makes maintenance easier for the very large
customers. So this is something that our cloud provider uses. We have a cloud
service in MongoDB and it uses SRV records to give you shorter connection
strings.

And now once we parse a connection string, we know that's maybe one or two
servers that we have to talk to. We need to discover everyone. So we're going to
start by doing a handshake. We know a few servers that we can talk to, we reach
out to them, we send them a command, and that is basically an exchange of
knowing what protocols does the server speak because there's many different
versions, and what protocols does the driver speak. And this carries over with,
if we're doing authentication, if we're doing compression, a zlib or snappy
compression for the protocol messages, we just want to negotiate what features
are available. And I'm rushing through a lot of this but the slides are very
verbose, but everything references, if you'd like to look this up later, I'll
share the slides.

So once the driver talks to the server, it know, well we support this
compression. We support this authentication mechanism. The next step is to now
we would go through authentication and we'd say I want to start discovering who
else is in the cluster. So if it was a replica set and maybe we just connected
to that one primary, we do want to reach out and know who the secondary servers
are. With a shard cluster it might just be talking to multiple router processes.
And so this we call server discovery and monitoring, we abbreviate it as SDAM.
It's one of the larger specifications. It's one of the more complicated ones
that all the drivers have to do. But this defines how we have a bunch of sockets
talking to servers and how we monitor them to keep track of new servers showing
up old servers, going away, getting network errors. And before we even talk to
the servers, we know from the connection string, maybe we can infer that there's
a certain, we think we're talking to a shard cluster, we think we're talking to
a replica set, but once we actually start getting responses then we know for
sure what kind of servers that we're talking to.

And so this is where things start deviating. And we have a multithreaded
drivers, think of like a Java app where they have large app servers, with
request handlers and threads, and there's one instance of the driver running on
the app server. and then the other example where we had all the boxes on top. In
a php app, with typically fpm, you have hundreds, thousands of workers across a
large app, a fleet of app servers. And so collectively a single threaded app
like php is going to open a lot more connections on MongoDB side. As well as,
just in the individual app server, if you have each php worker, we can't share
things between PHP workers because every, in a php application, everything
shares nothing with the other. It's kind of a design point of PHP is that we
share nothing with the other php app servers. So it definitely complicates
things in how we have to deal with talking to a distributed database cluster.

So we need different approaches to monitoring. And in a Java driver, Node JS, if
we have a multithreaded asynchronous or we could have a background thread that's
in a connection pool, which is more common. And then if you look at php, a lot
of database drivers in php, there's no reason to use connection pools because we
can't... most sane people are not using threads in php. And so typically a php
database driver, we persist the socket. So the PHP request comes in, we'll use
the socket, there's only one thing running at a time and when we're done with
that php request, we leave the socket open for the next request that gets
served. So for an FPM worker, that makes sense. The first worker will open the
connection and then every other php request that got served can use the same
socket.

And that saves us the time of setting up SSL and going through the whole
handshake. We can kind of inherit some of the state that we're connected to. So
in a single-threaded, separate sockets would be redundant. We don't do
connection pooling, it doesn't really make sense for php's design. And then
there's different improvements that we can make for monitoring. So there's a
concept of when, just in php itself, when you do, when you work with streams or
in other database drivers, you have a socket timeout in php, which is the
default socket timeout INI setting. For some reason php defaults that to 60
seconds, even though the PHP execution time is 30 seconds. So it's quite
possible that you would timeout talking to doing a `fopen` or doing a http
request in guzzle or something and your php scripts would die giving up. So
those defaults are a bit odd. But in the Mongo driver, we have a socket timeout
and that's us talking to the database, and then a connect timeout is for us
establishing the connection. So logically we'd like this to be a bit smaller
than socket timeout. Socket timeout is going to be you running queries and doing
other things and those are going to take longer. Connection timeout we want that
to be quick: try and connect to the server, if it's not there, we'd like to know
as soon as possible so we can return an error.

And there's other improvements that we can make here to kind of do this for a
single threaded driver. We don't have a background thread, we have to kind of do
everything on demand monitoring. So what we can do is use an event loop
internally to just kind of try to monitor all the servers at once, and waste as
little bit of your time so that your php script can continue executing.

And so with monitoring implemented, I won't dwell too much on that, the next
part is for the drivers to select servers. So the driver connects to a bunch of
servers and now we have to, when we do an operation, we need to talk to one. So
if we're doing a write, we need to know maybe three or four servers we're
connected to, which one is the primary. If we're doing a read, we might be
comfortable sending the read to any one of the servers. We can read from a
secondary if we want to. So this is a separate spec, just for knowing how to
select a server. And for it's very simple for writes, obviously there's only one
we can talk to. But again, for a single-threaded driver, we don't have a
background thread. So the server selection is kind of integrated with the
monitoring. They have to happen in the same logic.

And so how this fits in with PHP, if this is a basic script, we construct a
client, we get a collection which is basically a table, drop and insert and do a
find. In the very old driver, and even the new one, people expected that the
first communication with the server would happen when we construct a client.
We're actually, in the PHP driver, we're deferring that until we actually need
to talk to a server, so you have no IO happening when we construct a client,
it's the first time we know that we have to talk to a server is when we're going
to start monitoring, try to get a server for the operation. So the first
operation here is going to be trying to run a drop command, which is a DDL, a
database definition language command to drop the collection. That needs a
writeable server so that at that point we say we need a primary, I'm going to go
discover what the topology is, get a primary, monitoring starts. So the first
exception you'd have if there was no network connection, it would be from the
drop command.

And if you're curious, this is all the output. So it's just inserting and
dumping out the data. And so this is internally what happens, we construct, we
have a higher level composer package that wraps our php extension. And then
server selection, if we look inside what the drop command does, you see the
first thing it does is, I need a primary server and then I'll run the
operation.

## Driver Configuration

And so that's basically trusting the php driver to do monitoring and knowing
that it's a bit less efficient than if we had background threads, it would
certainly be better, but we have to basically do it on demand when we need to do
certain, when we need to access a server.

So, as far as configuring how all this works, there's different use cases, some
people's applications want to fail quickly, other people's applications you want
the most, you want to avoid errors at all costs and you are comfortable waiting
for that. So there's a lot of different options and flags available and these
are things that you would pass when you construct your client.

So the first one that I mentioned earlier is your connect timeout, and this is
when we do initial socket timeout. I'm sorry, the initial timeout in
milliseconds for opening the connection. And you'll find this, I think every
other stream or database driver has something similar to this. And there, our
default, which is, probably you would want to lower this considerably, is 10
seconds, which is a safe legacy value. Generally for this value, I think you'd
want to customize it to a little bit above your latency to your server. So if
your servers are within one second ping time, and usually maybe there are like
50 or a hundred milliseconds. So I might use 250 milliseconds for this. And
maybe double it, but certainly keep it below 10 seconds because what you want to
optimize for is if I start the php driver and I try and do something and there's
no server at all, I don't want to wait 10 seconds, I'd like to fail sooner.

And the second thing is `socketTimeoutMS`. And this defaults to a comically high
300 seconds. And this is more because of, we use the C driver internally. That's
the default. This is the MongoDB driver's equivalent to - in php - the
`default_socket_timeout`. Since our driver no longer uses, originally we did use
php streams internally, and this did apply, and now we do the socket IO
ourselves. So now we have our own value for this. So it is separate from php's
default socket timeout. But the two things you want to consider are what is your
socket timeout for other things that you're doing in php. And then secondly,
what is your script's max execution time. So on a command line at zero, it will
run forever, which is helpful with a large, when you're installing things on
composer, it tends to take a while. And for a web environment, I believe max
execution time is usually 30 seconds, which is still a long time to service the
request, but you generally don't, you want this to not exceed your
`max_execution_time`. So I would also recommend probably lowering this to 30
seconds.

And as far as monitoring, and this is the stuff that's happening in the
background, but you have some control over this. We can control how frequently
you want monitoring to happen. So a single-threaded driver, we do this less
frequently, so once a minute, and we save state between requests. So if you're
serving a quick php request and you just constructed the driver, you're not
going to hit 60 seconds. This is more for controlling if your PHP workers serves
hundreds of requests, maybe 40 or 50 requests down the line, at some point we're
going to realize that it's been 60 seconds and we just want to go talk to the
servers and see if we have a good sense, if they're the same topology that we
knew about a minute ago.

And this helps insulate us form errors, it detects changes if you were to, uh,
one of the benefits of MongoDB is that you can scale up and down while your
application is running. So if you were to add more members to your replica set
or the shard cluster, at the very least, the driver would find out in 60
seconds, 60 seconds intervals.

And the second is a socket check. And this is a special thing for
single-threaded drivers as well. In PHP's case we might have a worker that
serves a php request and then doesn't get used again for another 30 or maybe
five minutes. Who knows, because you have a pool of workers. So in this case,
before we use a socket, if it's been longer than five seconds, we'll just see if
it's still valid and that's, that way we can recover internally before you get
an exception in your application. These can both be configured if you need to,
but generally these are good as they are.

And the most relevant things to your application is going to be server
selection. So when the driver tries to select a server and we see that happens
every time you do an operation, if you're doing a write, we're trying to select
the primary, the two main things that you want to focus on here are the server
selection timeout, and the try once behavior. And so by default, a threaded
driver will wait up to 30 seconds when it tries to do an operation because it
has a background thread for monitoring. If your Java application is trying to
talk to the database, it will spend up to 30 seconds trying to find a server to
talk to, and that's not as bad as it sounds because they have their monitoring
constantly in the background so they very quickly pick up changes and they have
connection pools so they constantly have available sockets.

And again, php has neither, we have one connection to a server and we use that
one socket to do both monitoring and for your application needs. So therefore,
we have a behavior that allows us to fail fast. So if we try and talk to a
server and it doesn't exist, we immediately throw an exception. And the reason
we do this is because, consider the case where we didn't, and we allowed PHP to
block up the 30 seconds looking for a server. Your php workers would start
piling up, opening more connections and trying to talk to servers that they
can't reach, maybe there was a fail-over and so there is no primary, it might
take a five or 10 seconds for a new election to happen for the database cluster
to heal itself. And meanwhile, you're constantly opening new php workers. So
your app servers are getting backed up and you're not serving requests any
faster either. And so the most consistent behavior, even though it's maybe not
the best thing for MongoDB's case because it can heal itself, the cluster can
heal itself, but because of, to cater to php's unique needs, it's better to just
throw an exception immediately as the default behavior. And so that's the
behavior that we've had historically. And so we kept it with the newer driver
that we rewrote about three years ago.

Okay, I'm gonna talk a little bit about later when it turns into deciding how to
retry from errors, we can change that behavior and how to go about changing that
without. It's not as simple as just deciding to, say try once false and then
I'll start waiting 30 seconds. We definitely want to do that intelligently.

## Application Concepts

Okay. Just do a time check. Twenty minutes. Halfway point. Okay. So some
application concepts, before we get into error handling and transactions. These
are things that... concepts that the drivers provide and I don't think they have
analogies in a relational databases, but possibly other, um, other non-relational
databases.

### Write Concern

The first is a write concern, and in this case, ideally a production app is
talking to a replication cluster, so your data is being written to the primary
and then replicated to the secondary nodes. And again, to give us insurance so
that if this guy dies, one of these folks can step up and be the new primary.
And in some cases they might sync directly from the primary and in this, yeah,
in this case, they're both reading from the primary, but they're not all getting
the data at the same time because they basically tail the activity on the
primary. When you use a write concern and you issue a write to the database, by
default, we'll send it to the primary and if it succeeds, the driver gets back
control, you go on and do your business. And you're not really waiting for it to
replicate. But ideally, consider the case where you wrote to the primary and the
data you inserted didn't get a chance to replicate. And then the primary died.
From your application's perspective you think everything's succeeded. Uh, when
in reality it's possible that that data permanently got destroyed with the
primary. And when one of these secondaries steps up, the data is basically not
there anymore because it was never replicated.

So the way to control this in an application, and this is always at the expense
of, if you want more durability in a distributed system, you're going to wait
longer for it. And so you're gonna decide to use this. Some things are more
important than other things. If someone is doing a financial transaction, you
would like to make sure that the data is persisted and can survive any kind of
disaster scenario. And so the write concern you would want to use is just saying
I'm concerned with how this write gets acknowledged in the system - would be a
`majority` write concern. And this is a, you can either use a number, but
`majority` is a nice string that if you grow your replica set to seven nodes or
10 nodes, it's always going to be the majority number of them, which, and as
long as the majority have acknowledged the write, it can survive any disaster
scenario. So if we used majority here, the driver would not get a response until
at least one secondary, at least two nodes have acknowledged the write. So this
is generally what you want to use. And again, it is a tradeoff: if you did this
for every write in your application, you're probably wasting time. But there's
probably some things in your application that aren't as important to wait for.
If you're doing a log messages or small tweets, it's not as important as someone
updating their password or storing a billing address or something like that.

### Read Concern

On the flip side, we have a read concern, and this is when we're querying the
data: do I just want to go to the primary and read whatever's available? Or do I
want to make sure that I also read data that is acknowledged by the majority of
the system? And so as one applies the writes, the read concern is obviously
going to make your query slower because maybe you're waiting for the data to be
replicated or there's, you're going to purposely read a bit older data that is
safe to read because it exists throughout the cluster instead of the immediate
write that was just written only to the primary. So this is a lot to take in,
and it's something to research on your own and find what works best for your use
case. But I will call out the `linearizable`. This was something that was
introduced about two years ago and if anyone is familiar with the Jepsen tests,
which is the call me maybe distributed database tests, a guy, Kyle Kingsbury
runs a kind of a test framework for basically experimenting with databases and
finding out all the ways, how to break them basically. And so he had tested an
older version of MongoDB years ago. He's tested many databases over the years
and predictably the older version of MongoDB did not get a glowing review. And
he found all sorts of ways to break it and like write to the database and shut
down some nodes and realize that your data disappeared. And so this was
something that around the time of Mongo 3.4, which is about two years ago, they
put a lot of effort into addressing and making sure that we can pass that test
suite.

And now that, his test framework is now part of our CI system. So versions since
3.4 have been able to satisfy this. This is important maybe to people on hacker
news. As for everyone's, for your individual app, does every individual app need
these kinds of guarantees? Of linearizability basically being able to,
everything you write and being able to read it back immediately, and having
those very strong guarantees? Not for every use case. But the functionality is
there if you need it, and it goes without saying that the more guarantees you
get, these are going to be slower than just reading, `local` is basically just
reading whatever the primary has for you. But know that those exist and when you
need those extra guarantees, there is functionality available to obtain them.

And so because these things tend to take longer, I advise not to use socket
timeouts. We don't want to leave... using socket timeouts as a way to abandon
operations on a remote database is probably a bad idea, whether it's MongoDB or
SQL. If you start a long query and the driver just goes away and ignores it.
You're letting something continue to run on the database and now it's basically
kind of a Zombie process until the database server realizes that no one's ever
going to get the result for that. So we really don't want to use socket timeouts
to limit our operations. MongoDB provides... all the operations that talk to a
database, you can specify a time. And this basically corresponds to CPU time
that the server will kill the operation if it goes too far.

And now this is, socket timeouts are exact. If you tell your drivers, say I want
to give up after 10 seconds, it's exactly 10 seconds for the, on the client
side, whereas `maxTimeMS` is a bit softer. It's saying that I'm allowing this to
run for 10 seconds of processing time on the remote side. So this may end around
11 seconds or 12 seconds. So you don't want to set this exactly to whatever your
socket timeout is. But if you have long running operations, this will ensure
that you at least also don't leave them, even if the client does give up, you
don't leave them running on the remote side. And so this you can apply to reads
and writes, but for write concerns I mentioned when you're waiting for
replication, in that case, just to take us back to this, this "Apply" step is
the operation succeeding. So this would be your `maxTimeMS`. But then waiting
for the replication to happen is a whole separate thing. That's the operation
succeeded, but you're still, just going to wait before it returns to the driver.
So that's known as a `wTimeout`. And that's another thing you can say: if you're
using the `majority` write concern, you might want to... you know, that your
writes are going to be quick, but you also don't want to wait forever for
replication to happen. And so in this case, you will be able to distinguish: did
my write actually succeed? So I inserted successfully, there was no duplicate
key error. But at the same token, we timed out waiting for replication. Does
that mean that my write didn't succeed at all? No, it succeeded, but you're not
guaranteed at that moment that it had replicated. It's probably still going to
replicate if nothing, no servers died. So this is, we might, you might call this
a soft error instead of a harder error of the data actually not entering. You
have a message?

## Causal Consistency

A causal consistency, a causal consistency is basically being able to read your
own writes if you write something, there's like a linear progression of data. So
if you write to the database, being able to read your stuff back of the previous
operation. So there's an ordering component to that. So whenever an operation
logically depends on the thing before, there's a causal relationship for it. And
so the different guarantees there, one is being able to read your own writes,
which is, and when you have a standalone system where it's just one database
server, you implicitly have this. But when we add distributed nodes and there's
replication and things like happening, it gets a lot more complicated to make
these guarantees and that's when things get slower. So in a replica set, the
idea of causal consistency is not only reading own writes, but they're
monotonic. So that means if they get applied in the order that they are executed
by the driver, writes follow reads. So if you were to read some data and then
write, do write after that, that write is being applied to the same data on the
database side that you had previously read in the driver. So just, logically,
this is how we expect things to work. So it doesn't take, it's not hard to grasp
what this is, but it's something I think of, we take this for granted basically
when we're working with standalone databases where there's only one node.

And so this, there's `majority` read and write concerns. This basically can be
satisfied by using those. And `durability` is the idea that when I write
something it will survive: it's durable and that it will survive if we pull the
plug on the primary, it's been replicated. So it's durable: can survive an
error. And in your application, we can make use of this. The driver provides a
session object. We can pass this around to operations and this provides this
functionality if you need it. There's plenty of examples for that.

## Logical Sessions

So I mentioned sessions. This is a relatively new feature within the last year
or so. Sessions are a way... so previously operations in MongoDB were tied to
the connection. And so once we lose the connection, the server has no, doesn't
remember us at all, that's our, the queries that were running or any writes that
we did, they were tied to our connection. Assuming they didn't, if they made it,
if they were applied, that's different. But if any state that we had about who
was connecting to us was only based on the connection. So sessions allow us -
much like php sessions - give us some state beyond just connecting to the
database. And so sessions we can be, we can create them explicitly and we can
pass them around to operations. Internally, the driver also, anything we talk to
the server now, if it's a new enough MongoDB 3.6 or later, we're going to send a
session with it and make sure every operation that we execute is tied to some
session. That gives us an extra way to, uh, as a system administrator, to
monitor operations that are long running. We can, instead of just having to,
instead of getting orphan things, we can, the server keeps track of sessions and
they can be cleaned up, clean up like Zombie queries and things like that.

## Abort, Retry, Fail

And then this plays into our ability to retry things. And so, another quick time
check 10 minutes remaining. So this was something that I pulled out of an old
version of Doctrine MongoDB. Please don't do this. This is just the retry loop.
It takes the closure, how many times you want to retry it and will continually
try to call the closure until it doesn't throw an exception. So I assure you
Doctrine MongoDB has never, the ODM has never used this for writing. It's used
this for opening connections and doing queries. But that's still bad because if
you're doing queries, we really don't know if the query is still running on the
server.

So this doesn't exist anymore. I think it was deleted. But what's the problem
with retrying anything? We get an exception, why don't we just retry it? And
this is something you can apply to any database. So we have to be aware that
read and write operations, they can change the state of the system. So read can
leave the query... I mentioned if you have a socket timeout of like five seconds
you start query, that's going to take five minutes to run, right? The driver
gives up, it goes on and does other things in php land, but the server, it says:
"oh, I'm gonna work on this hard for you for next five minutes because you
really need this response". And then by the time it tries to give it to you, it
realizes, oh, that bastard went away.

And that's just a query case. In a write case it's kind of more dangerous:
instead of just wasting resources on the server, a write operation, if it's not
idempotent, which means we can't safely run it multiple times, we have the risk
of, think of a write operation that does an increment. It maybe, it's an
innocuous, it's not really a problem, but if you were to just retry the
increment operation, you don't know if you're accidentally over counting or
under counting. So at best we're wasting time and resources, at worst we're
making the data inaccurate. So we want to really think about how we approach
this.

So the different kind of errors, first you want to ask before we decide to
retry: what are the kinds of errors that we're trying to address? So I think
there's three types of errors and one is a transient error. I think of that as:
we dropped the network connection or the primary stepped down because it was
under maintenance and a secondary stepped up. This is a kind of a self-healing
scenario, it will resolve itself if we try it again, probably. Then there's a
persistent outage, which is like the whole data center shut down, but you really
can't do anything about that. But we really can't, we don't know the difference
between these until we at least try maybe once or twice. And if we retry and
there's no changes, we can assume that what we thought was a transient error is
probably a persistent outage. And lastly, there's a command error. This is: we
did something, the database came back to us with a result and said, you're an
idiot, you did this wrong. So in that case, if you're trying to insert something
and it says your command is malformed, you don't want to retry the command,
you're probably gonna get the same response. In fact, I think the definition of
insanity would be retrying in that case and expecting a different result.

So those are your three types of errors and what we really want to optimize for -
we really can't do anything about this - this is either change the code or
actually this is an error for the user. Persistent outage, we probably, maybe
someone will get a page and they'll fix it, but our application probably can't
do anything about that either. But we can optimize for this: this is really the
case, the transient errors, what we want to handle.

So retryable errors is either gonna be a network error or maybe a response from
the server that indicates that it's a transient error. So, if the primary was
coming down for maintenance, it might be totally offline - we got a network
error - or it might be alive for a bit and realizing that, Hey, I'm shutting
down, here's a response and I'm telling you that I'm no longer the primary. And
so those are two cases. This is usually caused by, there's maintenance or
there's a failover happening and we managed to talk to the server before they,
before they totally disappeared. But this is probably the most common one: it's
just going to be the network error. This is more of a timing coincidence.

And so from a retrying reads, I'd say we want to avoid leaving long-running
things running on the server. So queries that return a single doc if you're just
querying by id or a single response, you can retry those, right? Those aren't
very expensive queries. If you know, it's a short run query, and this is your
personal use case and maybe you're just returning one batch of documents or you
know, it's gonna just run for maybe a second or two. Those may be safe to retry.
And as of MongoDB 4.2, which will be coming out later in the year, the driver is
going to try to automatically retry operations, any kind of read operation, even
if it's a long running query. And this is because these server will have now
functionalities that, you know, the connection was abandoned so I can abort the
operation. So this kind of addresses the, we're going to leave some long running
operation accidentally running - the queries are going to churn for five minutes
while we gave up. So that'll avoid that. What we still can't do is if you're
iterating a large batch on an existing query, what we do `getMore()` is when you
iterate on your results. We can't retry that because those are forward-only
iteration, so that doesn't work. But the initial find command or the aggregate
command, we can retry that.

And now writes are a bit more complicated because not all writes are idempotent.
But given a few things, so sessions are cluster-wide, so they, I mentioned that
before, uh, they keep track of our state long after our connection dies. Every
operation is associated with a session and we can also give it an id, which I'm
just inventing now, just trust me, this is, we're just every session we'll just
have, the driver will increment some id value just so we can uniquely identify
everything. And then we also already have monitoring and server selection I
talked about. So we can rely on monitoring. If the primary disappears, we can go
back to monitoring and say I need a new primary. So we can rely on safely
retrying things, so if you're doing updating a single document, inserting a
single document, or doing a batch of those operations, they're all, each one is
uniquely identified and we can continually send them to the server and trust it
to do the right thing, which means since they're all uniquely identified, if the
server gets the write and realizes: "Hey, I never applied this", it's going to
do it and return the result. And if it already applied it, it knows that it did
so and it's going to return the result that we didn't get the first time, maybe
because there was a network error.

So this basically gives us a safe way to do stuff and we like to call this at
most once a. So any of the write that we retry, we want them to happen at most
once and because at most once that might be zero times if there is a persistent
failure and we just can't talk to the server.

And so this really wants, again in php, the try once, doesn't work with try once
because we want to actively monitor the server for a loop. So because we want to
talk to the server, the default behavior would just give up if we can't find a
primary because php wants to fail fast, it's not going to wait a few seconds for
an election to happen and for a new primary to step up. So to really make good
use of retryable writes, we want to enable, we want to disable the try once
behavior in your write. And there's a link here, of a gist where I demonstrate
how to do this. And I also tuned the timeout value down from 30 seconds a bit
lower. And this is basically our user Atlas cloud service. I keep spamming
update statements to it and I initiate a failover and take the primary down and
if you could watch the logs of the script, it doesn't generate any errors. When
the primary comes down it certainly waits about eight or nine seconds for the
election to happen, but if you want your app totally insulated from errors that,
in that case we can avoid exceptions entirely, which is really nice. In
Doctrine, in ODM's case, most of all the updates it does, it doesn't do
`updateMany`, so this is a great use case for Doctrine to take advantage of
retryable writes.

Okay. 60% of the time... it works every time. Generally it's going to improve
things, but again, it's, we can't resolve the other... it's about transient
errors, it's not about the other important things that we can't change.

## Transactions

So lastly, I'm a little bit over time, I'll try and do this in three minutes.
Transactions, this is our actually ACID compliant transactions. And so getting
to this point, goes back a number of years and adding features slowly to the
database and ultimately sessions, and then 4.0 is the most recent release with
transactions on replica sets. And then in the summer it will be transactions on
shard clusters. I cannot emphasize the complexity involved with transactions on
shard clusters. Replica sets is a bit easier, but I don't, this is above my pay
grade. So transactions at a glance, when we do a transaction, you're probably
doing a transaction because of writes. So the entire transaction has to go to
the same node and because it's a transaction and you're probably doing writes,
that means the primary, it doesn't really make sense to do a transaction with
just reads. Your read and write concern instead of per-operation, you do that
for the entire transaction. So everything has to have the same level of
guarantees around it. And that's important to the concept of a transaction
because it's either all going to work or it's, none of it's going to work.

And while you can do many operations there are some things you can't do, you
can't create collections or drop tables. But all your basic crud operations,
your aggregation frameworks and stuff, you can certainly, those are all
supported intentionally because that's the common use case.

So in php it looks pretty straightforward and I'm not using custom write
concerns or read concerns here to keep it simple. But from our client, we can
make a session object and that's our gateway to start and commit things. And you
want to make sure you pass your session to all the operations. If I forgot the
pass this here, then this `insertOne()` is not associated with the transaction
it's actually just going to run immediately. So that's kind of <inaudible>,
there's not really, we don't have like SQL grammar here. So the API relies on
you passing around the session to every operation. And similar, if you're doing
causal consistency, you want to associate your operations with sessions. So for
transactions you're definitely gonna use explicit sessions and pass that option
around.

And now the important thing is transactions can be retried. And so, the driver
is able, if a commit or an abort command fails because of a socket error, we can
retry that, and we'll do that for you once, just to try and insulate you. And if
there's a quick error that we can recover from, great. Uh, if you want, you can
retry committing more times if you want. If you want to do that in a loop until
it succeeds or retry any number of times. We don't bother retrying other
operations that's left to you to decide if you want to do that. And the other
important thing is transactions and retryable writes are two completely separate
things. So retryable writes was implemented first and that's meant to be used -
you can throw that into any application and kind of, it just works for a lot of
your write activity. Transactions are something you're opting into and the
transactions were really the main idea that the main goal that we had to get to,
and retryable writes was something quickly we were able to do along the way. Uh,
so keep in mind that, they are separate concepts that are mutually exclusive
even though they kind of use the same internal machinery.

But the important thing is if the entire transaction fails or if an operation
inside the transaction fails, you can restart. You can restart the transaction
from the beginning. So if we were doing a transaction and say this second insert
failed, I can catch that exception and then try and restart the whole thing. And
so the important thing you want to do here is probably have your stuff in a
closure or a function call that you can call and wrap with a start and commit.
So you can catch an exception and you can retry quite easily, or do a while loop
if you really like the procedural style.

So knowing when to retry. Our exceptions in the driver, there's two different
labels. Why not? Gruber seems to agree with me. So transaction errors, we need a
way to highlight errors from the driver and know with more than just the
exception message or code. So having a label, um, there's two labels out of the
gate and one of them, this is, if an exception is thrown during a commit, the
exception, having this label tells you that you can retry the commit.

And this is a label that you might get on any runtime exception in your code. So
those insert ones might throw an exception with this label and that means things
have failed and if you want to retry it, start from the top again and start
from, start transaction and try and go through the whole process again. There's
examples of how to do this. It's very hairy. So something that we're trying to
do in the next few months is have a convenience methods so that you just give us
a closure and we'll run it and do all the try catching and retrying for you. But
if you want to wrap your head around the annoying try-catch nonsense that you
have to do, for handling this, you can take a look at the documentation
currently has that and we're going to adapt it to a better API.

Alright. So these are some resources. I will share the slides. This was an older
presentation I did, just on if you want to know about retryable writes. There's
a lot of good diagrams and stuff in here. This is a lot of relevant
documentation you'll want to read up on the subjects we talked about. And this
is my coworker's presentation from many years ago before we had retryable
writes. So if you're using MongoDB 2.4 for some reason and you want to retry
write operations, he has an approach for what you can retry and how to do so
responsibly. Thank you. Please leave me some feedback later. I'll see you all
tonight for jeopardy and I'll stand outside for questions after.

Thank you for your laptop.
