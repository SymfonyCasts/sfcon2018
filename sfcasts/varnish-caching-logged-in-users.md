# Going crazy with Varnish: Caching pages of logged in users

***TIP
SymfonyCon 2018 Presentation by [David Buchmann](https://github.com/dbu)

You know how HTTP caching works but need more? In this talk we look into ways to
cache personalized content. We will look at Edge Side Includes (ESI) to tailor
caching rules of fragments, and at the user context concept to differentiate caches
not by individual user but by permission groups. A big help to leverage this concept
is the FOSHttpCache in combination with either Varnish or the Symfony HttpCache
reverse proxy.
***

Aright! Welcome to my talk on Varnish. Today I want to talk about how to use
Varnish, not just for caching static information kind of things that are the
same for everybody, but caching when the content that you show depends somehow
on who is viewing that content.

## HTTP Caching For Scaling

If you've heard some caching in general, I really like this quote to explain
why do we do the kind of HTTP caching, like the caching where we cache the
whole page. In the end, like all that we do, if you do a visual website or if
you do an API, the output is some kind of text content and if somebody else,
it's exactly the same text content. The HTTP cache is the most efficient
because you'll see, okay: he's asking for, he's asking the same question, he
gets the same answer and that's really, really easy. You don't even have to
touch your actual web server

Sometimes people start looking at caching because they realized, "Oh our page
is slow, so if we have a cache, it's going to be fast." It's not exactly right
or it's a bit... like the main reason why you want to do caching is to allow
scaling because you will always have cache misses: like moments when the cache
doesn't know the response and you have to hit the application and if the
application takes five seconds to render a page, some of your users will have a
shitty experience. Cache can't fix that problem. It will have... it will lower
the number of people that have a bad experience but it won't make the bad
experience go away. So the actual like the good motivation for caching is to
say we: we want to be able to scale. We want to scale with less servers because
if there is many people asking for the same thing, it doesn't take us a lot of
resources. But the application behind should still be in a reasonable shape to
deliver pages.

## What is a Reverse Proxy?

Now varnish or any kind of caching proxy on that level, they don't know about
the application. They know about HTTP, about the transfer of the data and they
see it as a proxy between the Internet and your actual web application. So you
have the requests coming through the proxy. It checks, "oh do I have that in my
cache?" If it doesn't, it goes to the backend, gets the response. And if the
response is cacheable, if it, if it is, um, there is ways for the server to
tell: "Hey you can cache this response" or "Hey you shouldn't cache that". If
it's cacheable it will store it at that point and then return the response. And
then the next time somebody asks for the same thing, it will see: "oh I have it
in the cache" and immediately return. And with varnish that's like a single
digit millisecond time that it needs to, to handle a cache hit, that's like
really, really fast and it doesn't take much resources.

Real World Caching: User-Specific Data

So the easy case of caching is when you have some kind of page that is the same
for everybody that's really easy. But in real world for example, let's look at
this newspaper thing which has here, there is the weather, it is localized,
it's talking about Zurich. Maybe if you come from Berlin it will show you the
Berlin weather. There is something related to your session, like here you can
log in and I assume if you logged in it would show your name, stuff like that.
So this depends on, on if you're logged in, this depends on where you access
the page from and then there is some kind of things that are updated
repeatedly. Like all the time they edit and update this and then there is some
kind of headliner a thing that doesn't change that much. And there is a news
ticker which maybe is even fed from the user profile because it knows that you
like sports, so the news ticker is full of sports or you like international
politics, so that's what they put in there. And such a page will have a high
load so caching would be very beneficial, but you can't cache this with the
naive approach because then everybody would see exactly this page of whoever
happened to access it, which doesn't work.

Another example, I guess most of you have seen GitHub and use the GitHub
interface. Again, like most of this stuff is the same for everybody. You have a
issue, you have the state of the issue information. Again, you have this login
information. Then you have some something that is like, this is specific to
you, but it's going to be the same on every page, but then this is specific to
you and it depends on which repository page you're on. You can watch this
repository or unwatch it. And then the green, like all these buttons to edit is
about your permissions on the systems. It's not really personal. Like, if two
people are allowed to edit, they will see the same thing there, but if you're
not allowed to edit, you will see something different.

## Dealing with User-Specific Data

So after that introduction, let's look at some options, some alternatives we
have when when we have this kind of content and we want to cache. So first one
is kind of the, if you can do that and do it because it's much easier: avoid
sessions or if you have a kind of CMS thing with a form somewhere, maybe just
not cache that one page with the form and you have the session for handling
that form. But once that page is done, remove the session and the session,
remove the session cookies again so that the rest of the page remains cacheable.

I call that cheating with JavaScript when you have, you said some information
and then you deliver static pages and have JavaScript for changing little bits
of the page. Then the kind of tricky approach of caching, even though there is
cookies or caching per user - we're gonna come to that later - and finally the
user context idea of where you cache, not by individual user, but somehow
define groups of users that have the same shared cache.

## Avoiding Sessions

So avoiding sessions. The like the the thing maybe to say that explicitly, as
soon as there is a cookie, a normal cache will say: "I will stop caching"
because the cookie is individual. It's probably meaning that everybody gets
something different so I shouldn't cache. And with this approach you would
leave that behavior. So if there is a cookie, there is no caching, but if, if
you don't need the session, you would go and delete the cookie immediately so
that there is no session. One thing on, on Symfony, if you use the flash
messages, that triggers a session just if you do `app.flashes`, that thing in
it initializes the session. So if there is no session, the response will set
the session cookie and you stop caching. So you could check like does the
request... did have a session? So should we do that? And if there is no session
there will be no flash messages because the flash messages are stored in the
session. And like in general if you do this you will have to check your
application if it happens to somehow start sessions and then go figure out how
to not make it do that. If you don't need the session.

Another thing you will need to do, so this is the Varnish configuration
language, you would want to clean up the cookie so that it only has the php
session id in it and not any other stuff. Like typically if you have Google
Analytics on your page, there will be some random cookie things that actually
only the client side cares about, but that can be sent to the server and the...
Varnish or other caches don't try to guess what exactly the cookies mean. So it
will see, "oh there is some cookie" and it changes all the time so I'm not
going to cache that. So what you do is have this regular expression to extract
only the session id out of the cookie and keep that. And then if there is no
session id, you would unset the cookie.

## Moving Logic to the Frontend

So far so good. Cool. Then if we move the logic to the frontend, so the
cheating is not nice to say. It's not really cheating, it's just a solution
where you can still cache even though there is some things different. So
basically when you log in you would set some kind of cookie that the JavaScript
and knows oh this user is logged in now. And with the thing that I showed
before you remove the cookie in the requests, you would even remove the session
cookie for this approach. So the user is logged in, but then on most routes the
session cookie is removed and everybody gets the same page. But then, uh, in
the frontend, in JavaScript, when the page is loaded, you would change the
page. So the rendering would include all the editing buttons. For example, if
you, if we take that GitHub example, we would include all the editing buttons
and then have JavaScript show them or not show them depending on what's going
on. Or for example, if you have this, say you're logged in - "Hello David" -
you could store the information that I'm called David in a cookie and then
whenever the page is loaded, have JavaScript replace the login button with
"Hello David" and the link to the profile. And then if I click the link to the
profile, the server would not ignore the cookie and have an actual session and
not cache that because that's my profile editing - there is no point in caching
that part.

I think that's really useful when you have these like small bits of a page that
change or you have the login on a otherwise pretty static page, that's a useful
approach. You get fine-grain control over what to show. You can cache a lot of
things. You don't have network overhead with this, uh, and the backend, doesn't
have any additional load from it and you can handle complex user interfaces. On
the downside, the templating gets more complicated and you would, you would
also in like put more things into the page then needed on any specific for any
specific user. It's like everything for everybody. It can be a bit tricky to
maintain. And if you need additional data, for example, if you, if the editing
would show you some kind of list that non logged in, users are not allowed to
know of - like a list of options and you don't want to leak that information -
you will need to add additional calls to fetch that data and so on. And then
suddenly it becomes complicated.

Again, kind of similar, you could use AJAX. So say we have these parts of the
page that are the same and cached, and then we just have the AJAX include to
fetch something with the session that is depending on the session.

Yep, so that would be executed after the page is displayed. So if it's a little
information, that's not the thing the user wants to see first. It's okay. Um,
but if it's like the main thing, you have this wobbly experience of the page
loads and then information starts plopping in. And you'll potentially end up
with lots of requests to the backend and again, you complicate things. So
overall it's similar to the previous approach where you, kinda make your
application a bit more messy to optimize it for the caching.

## Caching Despite Cookies

Now you can force caching even if there is a cookie with Varnish, um, but it's
extremely easy to trade problems. Like if you cache something and the, and that
something really depended on the user, then the next user would see the same
thing even though he's a different person. So you want to really avoid that.
The backend would in this case needs to control what should be cached. So if
it's individual, it really has to, to set this `no-cache` header. You could
cache things depending on the session by saying it's cacheable, but it varies
on the cookies or the Vary header says: if this header is the same in another
request, then the cache is okay. But if the header is different than even the
same URL is not the same cache. Like it's... the cache is then calculated...
the identity of the cache is the URL plus that Vary information.

So that could work. If you have this AJAX call to show my login status, for
example, that would be the same. When I do the call with my cookie, it's going
to be the same all the time and I will do that call a lot because I do it on
every page. So that would be a place where you can use that. If you put Vary on
Cookie for everything, you better remove the cache because there will never be
any cache hits basically. Because if you... like, every user will have a
different cookie because the different session and then everybody will see,
like every request will be different and you have no almost no cache hits. And
if it's about the same user accessing the same page again, then you better make
the browser cache that then your varnish. So Vary cookie is useful in special
cases. And then if it's something that doesn't depend on the session at all,
you would just say: cache it - it can be cached, even though there was a cookie.

If you want to do that in Symfony, you will have to explicitly tell Symfony: I
know what I'm doing, since version 3.4. Before it was, if you had a session and
then you set the header "this can be cached", it would transmit that. And then
it would be Varnish that said like: "oh, that looks fishy, I don't cache it."
But now Symfony itself will, as soon as there *is* a session, it will set the
cache headers to make the, the response not be cacheable. And to disable that
behavior you have to, to set this kind of magic header to tell Symfony don't
override my headers. I actually, if I say cache then I, I wanted to say that. I
know what I'm doing.

And then on the Varnish side, so this is the default behavior. Varnish will see
is the request even a GET or a HEAD request and if not `pass` means don't try
to cache. And if the request is not a GET or a HEAD request then there is no
point in caching. And then by default the next step is it looks at the
`Authorization` header which is used for the, like these pop up basic password
thing. Or if there is cookies. And if there is any, then it also decides
`pass`. And then the last line where it says hash is saying in all other cases
I want to try do cache look up. Now if you want to cache or have the
possibility to cache, even if you have cookies, you would change this method.
You will override that in your own vcl. You keep the thing about the GET and
the HEAD because you still don't want to cache POST requests or whatever. But
then you don't check for a Cookie or Authorization header, you would just say:
now try caching.

## Edge Side Includes

Now, this thing can come in handy with edge side includes, which is a bit like
the AJAX thing we discussed before, but on server side. So, it's um, your
response that the backend would deliver, instead of rendering everything in one
page, it includes URLs to say like: "Here, please put in - copy paste in - this
bit that you get from this other URL." And then Varnish will fetch, like look
at that and fetch all separate elements and stick that together and then send
it to the client. And the very interesting thing for us here is that each bit,
like the main frame and then all embedded parts have separate caches. So each
of them can have different cache rules. So, for example, you could say this,
this login thing, we cached this, would Vary on Cookie and then the rest of the
page doesn't depend on the Cookie. And then Varnish will have two parts like
this login thing and the actual page. And the actual page is cached as soon as
anybody has looked at it. And your login thing is cached as soon as you logged
in. So you increase the chances that you have a cache hit.

Symfony has built in support for the edge side includes. So in, in HTML, like
what it will output to you is this. So it has this `<esi:include` thing and
this is a URL that in this case would be relative to the URL of the document.
And, so the cool thing about this is we have separate caching for each
fragment. It's on the server side, so search engines will see one page and not
some JavaScript thing stitched together. And in your application it doesn't add
a lot of overhead. The, like one issue with this is every single edge side
include has to be resolved before the response can be sent out. And with the
normal Varnish server it will resolve them one by one. So if you have something
with like a lot of includes that can become really slow. There is Varnish Plus,
which is a commercial version of Varnish and there they offer this additional
feature that the ESI can be resolved in parallel. You can configure like do at
most whatever, 10 requests in parallel. So it's doable.

In Symfony you will activate in the configuration: you say, I want to do ESI
and these, like, sub parts of a page should be exposed at this route that you
can specify, for example `_fragment`. The thing is this will, with a naming
convention, expose controller actions, even actions that don't have a route. So
you really want to make sure that this route is not accessible from outside.
Varnish must be allowed to access it, but the public internet should not have
access to that fragments route, otherwise they can try and call controllers
with weird stuff and you will lose control of what's going on. And then in Twig
you would included like this: say `render_esi()` and specify the controller,
you can specify parameters. These parameters, um, can't be objects because they
need to be included, it builds a URL from this which must be some sort of
string that then Varnish will send. And it's also not the same PHP process that
will handle this. So if you, I don't know, if you have some information in the
container or if you did some globals, I don't know, that's not gonna work with
ESI because it's a separate request that hits the backend. If you have a
multi-node setup, it could hit the different node even. So it's really a
separate request. So you have to be careful if your application has too much
state in it.

## Caching by User Context

Now, even with ESI, we still either cache, globally for everybody or we cache
individually with the session cookie for one single user. Now the rest of the
talk I want to spend talking on user context. So like, example from traveling
where they say... we just take this deciding criteria: if you're UK or have a
European Union passport, then go this way and everybody else goes somewhere
else. And so the distinction is not on the individual passport but it's just:
is it an EU passport or a UK passport. So we try to build groups with somehow
the same permissions or the same properties. In Symfony, for example, it could
be the groups of a user, because Symfony security has this concept of groups
already. This can be implemented quite transparently: the reverse proxy does
the job and the application doesn't really need to know about it that much. The
Hash itself is individual for every user. And, yea, we can like build our own
rules how we built that hash. And then again the hash look up will be a request
to the backend, but we can cache that. Because if you have a session, like it
makes sense if your session, if your hash stays stable as long as your session
is going on.

So how does that look? We have a request that comes with credentials. And that
request, Varnish goes to the web application and says I need a hash for this
user sending along the credentials. And it gets back a response with, with that
hash. And the Vary, like this response depends again on the Cookie or the
Authorization header. And then we do the actual content request and add the,
this context hash to the request and if the response really only depends on the
hash, on the groups of the user and not on the individual user, it would reply
with it varies on that hash. And then if like on the second time this happens,
we would have a cache hit.

So in a more flow diagram, the browser asks for `/welcome` and the reverse
proxy first gets the hash for the user... would get some hash. And then asks
for: I want `/welcome` with the hash is this and gets a response that says,
okay, the page looks like this and it depends on that hash. Now, next time we
have a request, if it's the same user, we already have the hash. If it's a page
that anybody with the same hash, so anybody with the same set of permissions
already asked for, we can have a cache hit and say, okay, so here you get this
same page that the other user got. And the thing is you're, the web server when
it handles the actual request it, doesn't need to know about this hash thing,
it just takes the credentials of the current user and builds the page as it
builds it for that single user. So you don't need to rewrite your application
to to handle it in any special way. You only, at the end, you need to decide is
what I did... did that only depend on the groups of the user or was it
individual so that you set the correct Vary information.

In Varnish side, it would look like this. So, when we receive a request, we...
so here, I use the curl Varnish module, which allows Varnish to send a web
request before forwarding the actual request. So here we create a new request
and we add a host header, I call `auth` - that's just the name of the server
that handles authentication and do a request on ourselves with all the original
headers. And then if the, if the response is not 200, we return with the
response status code, most likely that was the credentials were invalid. And
otherwise we copy the, this user context thing from the response that we got
onto the request and then continue and send the original request to the backend
with that hash on it.

And...the reason we don't go directly to the backend here is that we want to
also cache the context lookup and with setting this `auth` header we can, as a
first thing in the receive, we can say: "oh do you ask for `auth`? Then, if
you're... if I am myself, like if I was just doing a request on myself, then we
go to the backend and force the cache lookup. And if it was any other client
asking for getting a hash, we stop that because it makes no sense. It's likely
somebody trying to hack into our system.

And then finally on Symfony side, we could do something like this. We take the
roles of the current user and then, build a hash from that. So this code is
built on top of the
[FOSHttpCache](https://github.com/FriendsOfSymfony/FOSHttpCache) HTTP cache
library which provides tools for, doing this. And actually like this bit with
the user roles that's already provided in the library. So writing your own
thing would be if you need something else than the user roles.

## Wrapping it all up

Does that make sense so far? Any questions at this point or confusion? Nope.
Cool. Then I'm going to wrap up at this point. I presented a bunch of different
approaches or solutions, but often they can, they can be combined. So for
example, you could cache things even with cookies and use edge side includes
for that or use AJAX to do it on client side or mix user context with the AJAX
things or even another thing, you move like a lot more of the application to
the frontend and build more of an API on the backend. And if you have small
enough endpoints possibly you could increase the cache hits with that as well.
And of course the whole user context thing, we're using that in an API as well.
So with JSON data it works just as well. When you have different API clients
with different permissions, that that works fine too.

And depending on your scenario, you could be writing your own information
provider for that context hash. For example, the, the thing from the beginning,
the newspaper where it has the temperature where you would say the hash
actually is your geographical region. Or maybe you have a localized page for
every country. Then you could do a hash based on, on, what is your location
from the IP and whatever tools you have available to determine that. Or based
on the user agent or on some client capabilities. Or maybe even for A/B testing
where you would set some kind of A/B cookie and then make a hash based on that
and have caching of your A/B testing.

And as I mentioned before, there is a FOSHttpCacheBundle that I'm maintaining
together with others. Which besides having all this stuff for this user context
handling, it also has support for cache tagging where you mark caches as: this
page contains this or that article maybe and when you want to invalidate you
can say invalidate everything that is tagged with this information. And it
contains a system where you can, in your Symfony application, you can record,
okay, after this POST request I want to invalidate this and that cache. It
supports annotations on your controllers for, for the caching headers and it
also has a system of request matchers where you configure a on this URL and
everything that matches this pattern, I want to set some cache headers. Like
all old things under `/user` are `no-cache` or all things under `/static` are
always cached, things like that. Because with plain Symfony we have the
Response object and you would have to set on the Response object cache headers
in every controller and if you have lots of controller actions, this can be
useful.

Yep, and with that I'm at the end. I'm open for questions if there is any.

## Questions

Yes, please. Do we have a microphone? Oh, that's, that's too big to see.

By the way, we use Varnish and it handles millions of requests and it's awesome
for us. Anything for using it locally? Because when we are trying to, to go the
homepage or stuff like that, it's really painful to always try to add
parameters to invalidate the cache locally and stuff like that.

So for us, when we work locally we usually don't put the cache in front. Like
we go through, to the Symfony application directly to, to avoid the caching.

Yeah, but we use the ESI include. And for us we, we need to test it locally.
Otherwise in staging or live the behavior will be different.

So if you use ESI with Symfony, then Symfony will see, like if you use the
thing that I showed... yep, this one. So if you use this construct Symfony will
actually see if the client is capable of ESI and if it's not capable, Symfony
itself would resolve the ESI includes. So in that case it could, it should be
able to render the whole page for you - without Varnish - so that, that should
work. The other thing you could possibly do - are you on Symfony or is it
something else? No. No. Okay, then one thing you maybe could do is define a
separate Varnish configuration for that case where maybe you, in the receive,
before it does the hash, like this return hash thing to say I want a cache look
up. You could put: I actually want to pass, I never want to do a cache look up.
And then you can always go through Varnish and have all the logic the same, but
only this little bit is different. And then it wouldn't cache ever. I think
that's actually a good idea because you have a system that is very similar to
production.

Our idea is to have the same, the same system as live.

Okay. Can you pass the cube over. It's very light. Throw it.

Hi. I'm working with Varnish for a couple years right now and coming from
another framework and just edit the `esi:include` in this other framework. And
I'm, I'm using the `esi:remove` part for, for dealing with local development
because this is what I see when I'm not using varnish. So I edit the real
rendering and the remove path depending on, some, server environments, the
filling out of the environment in the local development mode, and then adding
those `esi:remove` tags with the real content, but still providing
`esi:include` tags, whether you... this is how I can just switch easily between
using varnish with the include path and having a local copy with the real
output. So maybe this is some kind of solution you want to use as well. And if
you want and have just a couple of minutes I can show you on the evening what
I, what I did in this other framework.

Thanks a lot. Any other questions?

Hello? Is it possible to use Varnish with ESI in a JSON request?

Yes. It is.

So, the only thing you have to look at, you need to set some kind of flag
because otherwise it's thinking, oh, this is stuff I don't know. So I'm not
gonna... Sorry, if you do ESI... the thing is if you would, I don't know, if
you have some pdf or some binary format and you do ESI on that, then probably
that's not what you want. So Varnish says, oh, this is like binary things I
don't know. So you have to set some, uh, some kind of flag in the startup flags
to tell it. I want you to do ESI on JSON responses. Otherwise it won't try to
do it because it doesn't know if that's what you intended. But it is possible.

Hello. Awesome presentation. Thanks. Thank you. I'm wondering about
FOSHttpCacheBundle. You said that there is a possibility to annotate,
invalidate all, or cache everything. Is there are also option for invalidate
some part of cache. Like in this (??) approach invalidate only comments under
certain posts. Like [mighty] post.

There is. It's um, so we leverage the ban feature of Varnish itself. Like
Varnish itself has that as well where you say: I want to invalidate everything
that matches a regular expression and you, you can match on the path. You can
also match on certain headers, like the domain name or random other headers.

That works fine, but if you use it too much, you run into problems because
Varnish has to keep like, Varnish might have thousands or hundred thousands of
pages in the cache. So when you send that request, it's not gonna go into the
cache and look at everything to remove that immediately. But rather it says, oh
okay. So I, I take note and whenever a request comes it compares that with this
ban or with all the bans that you sent it. And then if one of them matches it
said, okay, so this cache entry actually is gone. But that means if you have a
lot of bans and a lot of requests, you will slow down your requests and at some
point that goes... like we had that for awhile when we accidentally did too
many bans, where Varnish would repeatedly be unusably slow suddenly because it
has so many things to look at before it can return a response because it has to
check like, oh, is it banned? Is it banned with any of these things?

So, it exists and it's, it can be useful, but I would be careful when to use
it. But the cache tagging, there is a, a Varnish module where it does some sort
of binary tree on the caches, so Varnish actually understands the tags and that
can be really efficient. So there, if you can tag things, that would be much
more efficient if you need to invalidate often. And if it's like very rarely
that you say like, oh, whatever, this whole section of the page needs to be
invalidated now, but you do that like once a day, whatever, then that's not a
problem. But if you have some processes and it does that like hundreds of times
an hour, then it's a problem.

Thanks. Thank you. Any other questions?

Uh, hello. How works Varnish with the https together. Or not working?

Um, so the open source Varnish, you would put something in front of it that
handles the https because Varnish itself doesn't... like it doesn't... period.
The Varnish Plus - the commercial version - there they, they sell some, that
they sell, they added something for the https handling. But the, it can make
sense to put some Nginx or some, a small... like there is a couple of small
proxies that only are really built for that kind of thing. Like just https
termination. But Varnish itself doesn't handle it.

Alright. Yeah, I think we're closing here and I'm still around. So if you have
any more questions, come up and ask me or ask me later today or tonight or
tomorrow and have a good lunch.
