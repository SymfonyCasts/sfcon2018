# Changing PHP

***TIP
SymfonyCon 2018 Presentation by [Pedro Magalhães](https://github.com/pmmaga)

PHP releases a new minor version every year. Major versions happen when there
are enough changes that justify to do so. Who is making those changes and how
does that process work? What is the process to get an RFC to vote and the
subsequent merge of the code? How can I make my first contribution? Is there
anything I can do even if I don't know C? If you are intrigued by PHP
internals, this talk is for you.
***

Okay. So, Hi everyone. Thanks for joining this talk about PHP.

## Hi Pedro!

First, a little bit about myself. I'm Portuguese, I'm Pedro Magalhães, by the
way. I'm Portuguese, I'm 33 years old. Uh, I'm a PHP developer at the Dutch
company called Emesa during the day and I'm a PHP contributor by night. Some
nights, at least.

Um, I authored one rejected RFC, which was to get rid of that small "b", uh,
before the strings because in PHP you can always put a "b" before any string.
Uh, and this is a relic from the past when this was introduced for forward
compatibility with PHP 6 and the binary strings and the unicode strings. And
that never happened as we all know, there is no PHP 6, but the "b" is still
there 10 years now and it still doesn't do absolutely anything. But I tried to
get rid of it, and people didn't want to get rid of it. So, it stays. Uh, and
well, also one approved RFC - it's the first one that was approved for 8.0.

And uh, yeah, I also tried to contribute a little bit to other open source
projects, like I have my very own badge of Symfony code contributor. And, yeah,
I like games, and beer, and technology. Yeah, okay.

So in this talk it's called changing PHP and changing PHP can have two
meanings, uh, and I'm going to talk about both of them. So one of them is as an
objective, um, changing is something in a state of becoming different. And so
we will talk a little bit about how PHP is changing lately. Um, and then as a
verb. So I really like this definition to make the form, nature, content,
future course, etc of something different from what it is or from what it would
be if left alone. So this is the two parts of the talk.

## Changing PHP as an objective

So, let's start with the first one - with changing as an objective and, um,
let's start with a reality check. So this is where we stand today. Um, 5.6 has
24 more days of security updates and then 5 is done for, forever. 7.0 is
already out of security updates. Even though there will still be one less
release, uh, that was not planned, but there will still be 7.0.33. Um, but
yeah, that's it for 7... for 7.0. Then 7.1 is already security only since five
days ago. So, currently the only actively supported version of PHP is 7.2 - is
the only one that is getting actually bug fixes. But, the good news is 7.3.0 is
released today.

### PHP 7.3

It's not announced yet, uh, on the website. Um, but, but it will be later
today. I promise, by night you can have 7.3 in production during the social
event we can do it. Um, and yeah, it will be, it will be nice. But yeah, it's
really, really great. It's a great timing that 7.3 is released today. So
because of that, let's talk a little bit about PHP... about 7.3.

Um, and let's talk about some of the things that it will bring. So, one of the
first that I want to mention is JSON_THROW_ON_ERROR as it kind of advertises,
it will be possible to pass chase and throw an error to json_encode() and
json_decode(). And it will start throwing exceptions instead of you having to
go out and check json_last_error() if something wrong happened. And, well,
yeah, sometimes it did, sometimes it didn't. And a lot of the libraries for
handling JSON are pretty much just wrapping this up, just checking the
json_last_error() and then throwing it for you. But now, PHP can do it by
itself.

Then for the OCD ones among us. Now you can use a trailing comments in a
function calls. Um, so your, your diffs are going to be much nicer now because
you don't have to add a comment to the previous line and all of that. I, yeah.
For some people it may be, it may be useful or nicer. Then, array_key_first()
and array_key_last() also as the name advertises, it gives you the first and
the last keys of an array. Um, and um, yeah, without changing the internal
pointer of the array. So, the array stays in the same place where it was.

And this way you can check it. It has, didn't fortunate decision in my opinion,
of returning NULL, in case you pass an empty array, I would prefer to throw
something, but mostly it's, um, it's, um, it's a good addition.

And also, flexible heredoc. Uh, so for heredocs and nowdocs, um, you can now
use this, uh, this syntax where the difference is that the closing marker. So,
in this case, the end no longer has to be followed by a new line, so you can
actually now close the function call that you are doing a after the closing
marker. And the other thing is that the closing marker, I'm basically
specifies, um, the indentation of the rest of the text. So, in this case,
because END is not really at the, at the, um, at the first column, um, and it
is at the fourth column, it means that C will only have one space before it
when it's printed.

So, basically, the closing marker now dictates, um, where the, um, where the
intention starts for all the text. Then, uh, SameSite cookies - it already
existed in Symfony since 3.2. So, this is not news for a lot of you, um, and
since 4.2, there was 3.2 is already from two years ago and 4.2, it also added
support for that for session and "remember me" cookies. And the SameSite
cookies for those of you who don't know, basically it's a way to prevent a
cross site request forgery. And, uh, basically the way that it works is that if
you put it too strict, uh, the cookie won't be sent if the request is cross
site. So, if you are on a different website and you'll make, um, yeah, you
click a link to your website, the cookies want to be sent with that request.

Um, but because that will be really unfriendly in some cases, uh, you also have
the Lax which allows the cookie to be sent if this is considered a safe request
and it's safe request in this case is mostly a GET request, or any "read"
request that is a top level navigation so that when you are on Gmail, for
instance, and you click on the Twitter link, you stay logged in because, uh, if
even if they use the SameSite, if it is Lax - than your cookies will also be
sent.

But, in PHP, this is the current signature of the setcookie() function. And so
we wanted to add one more - SameSite. Can I join you guys? And the guy said:
No! So we had to come up with the, with the alternative syntax for this because
this was, this went to vote and it was rejected. And so, the solution that
ended up coming forward was this one of setting the cookie and you get the
name, the value, and then an array of options. And the array of options is of
course, all the other options, so `$expires`, `$path`, `$domain`, `$secure`,
`$httponly` and `$samesite`.

More on 7.3. In this case, so, we have a `while($foo)`, `switch($bar)`, `case
"baz"`, and then we have a `continue`. Who here thinks that the `continue` will
continues the `while()`. Who thinks it will continue the `switch()`?

Okay. Well, that was not a lot of confusion but in case it is confusing because
it can be confusing to some people. Now it will throw also a warning letting
you know of that, that the continuous the `switch()` is just a break and it
asks you if you wanted to, if you meant continue to because then you, as with
continue and with break, you can add the number to say the number of loops that
you want to break out of or continue from.

Another thing also in reflection, in errors, uh, you would see before you would
see integer and boolean. But that's not really what you typed down. So now
it's, it matches what you use as well. So, now they are always called int and
bool, which is nice.

Then we have a new password hashing algorithm if you build it with the, with
the password Argon, with the lib... Argon library. Then you can also use now
the PASSWORD_ARGON2ID. Um, in FPM, there are also some changes. The entire
logging of FPM was refactored and right now you can set the log limit variable
to define what will be the maximum length of a line on the log, um, which
prevents some before you could lose some information because of, because of the
line that was too big. And right now this solves that problem.

And also it allows you now to not decorate the workers outputs because PHP
would also always add some information to the, to the output of the workers and
now you can actually specify that you don't want that. Um, besides that, also
FPM finally got the `getallheaders()` supports, which is basically to give you
all the requests heathers and well, it didn't work in FPM.

Then, another interesting one is the `hrtime()`, which is the highest
resolution monotonic clock and the monotonic clock, basically, it's a clock
that's always goes in the same direction. That is a guarantee so that it's not
affected by a daylight saving and stuff like that. It will always, always,
always goes forwards. Um, and it starts at the unspecified point in time, in
the past. It's not important, it's just that you can measure it twice and we
will get the most precise difference between those two times possible. So this
is really useful also to measure stuff, of course.

And of course there are a lot of bug fixes and performance improvements and
stuff like that. But who cares? I'm, this is the truth. And sometimes it's also
the truth for contributors, like, ah, yeah, I want to do something new and
shiny, but yeah.

### PHP 7.4

So, but this is mostly what I have to say about a 7.3. About 7.4 which is
coming one year from now. So holds, you have to sit down a little bit to wait
for that. But, uh, there it is already showing a lot of very interesting stuff.
The first one that I would like to talk to talk about is about the preload,
which is already accepted and it's already merged in the master. Um, and it was
developed by Dmitry Stogov and to quote what he said because I think it's a
good definition.

So on service startup, before any application code is run, we may load the
certain set of PHP files into memory and make their contents permanently
available to all subsequent requests that will be served by that server or the
functions in classes defined on these files will be available to requests out
of the box exactly like internal entities. By example, the string length or, or
the exception class.

And what does this mean exactly? So, currently, on 7.3, um, so PHP starts and
the first request comes. And I imagine that on your index.php where your, your
requests are arriving, you do a new A. These will trigger the autoloader
because it doesn't know about it yet. So it will trigger autoloader: Hey, what,
what is A? And your autoloader imagined that it is composer's autoloader a will
then find that file and include it. The file is compiled into OP codes and
those OP codes are stored in the OPcache. And after that it will run the
constructor of A on the subsequent requests you do new A again. It will again
triggered the autoloader, it will, um, the autoloader will include A.php but
this time we don't need to compile it anymore because it's already on OPcache.

So now we load the OP codes from OPcache and we run the constructor. On 7.4
with preloads. So we, when we start PHP, we can pass these, uh, these setting,
or we can have this ini setting saying `opcache.preload` and the file in this
case, so, for this example, I'm just preloading A.php directly. Typically, it
would preload the file, that would include all the other files that you want to
preload.

You can only specify one file and then that file is responsible for preloading
the rest of what you want. But in this case, for the sake of simplicity, let's
say that we just preload A.php, and at this point A.php is compiled and it's in
memory. So for the first request, when you do new A, it runs A, um, A
constructor because it already knows what it is. It already, it's already
loaded in memory. It doesn't need to hit OPcache at all.

So, yeah, all the requests are now immediately... you don't have these
intermediate steps. This, of course, there're the downsides, uh, all the files
that are preloaded, if you want to change something in them, uh, you need to
restart PHP. So, there is no way to invalidate these files. The only way to
change them is by restarting PHP.

## Typed Properties

Then, typed properties, finally. Um, so yeah, it, it's happening. I'm sorry if
you're having a recruiter speak flashbacks. Um, but yeah. So, it's happening.
It's going to be on 7.4. Um, and some, some interesting things, so, object
types, uh, cannot have a default value unless they are nullable and indicates
that they are nullable then you can use null as a default, but you cannot set a
default values for, for thing, for properties that are of a specific class and
they will not be null. They will be undefined if you don't define them in your
constructor. So we have a new states for, for properties which will be
undefined, which you shouldn't really encounter if, if you ever encountered
undefined is because someone forgot to set the property on your constructor. It
shouldn't happen, but, it, uh, it is there.

And, currently this is still being discussed. Uh, but this one was already
voted, was already accepted. It's not merged yet because they are still
struggling with some implementation details. And the other one, it's currently
on discussion. So this one I cannot tell you that it will happen for sure, but
I'm pretty confident it will, which is about variance. So in this case, so we
imagined that we have a class A, grandparents and then we have a class B which
is the parents. And then we have a class C which is the child. And then we
define an interface and that interface receives a B, and it returns a B.

We can now, well with variance, we can extend these interface and we can use a
contravariance on the, on the, on the, on the parameter types, which means to
go up one or multiple levels, so... and, uh, for the return type you can go
down. So that's covariance. And, basically, these does not break the, it does
not violate the Liskov substitution principle because currently you could
already, even just when you have the type hint and on the child, you could just
remove the type hint. But this is now more complete, so it will actually, like
if your parents can deal with the B, then your child can deal with something
that is greater than a B and then can deal with it as the same way that if your
parents returns a B, it's safe for the child to return to the C because the C
is also a B. So it never breaks, it never violates the Liskov substitution
principle.

### PHP 8.0

Okay. So for 8.0, um, there is already a change. So I said that, uh, the RFC,
um, I have an RFC that was accepted for 8.0 and it's very simple, but it's
basically about this. So if we have an array with the, with this initial
structure, and then we add one more element to this array, uh, what will be the
index of these new element.

You wish, but it's not. Currently, it's not. So, as the documentation says: If
no key is specified the maximum of the existing integer indexes, uh, is taken
and the new key will be that maximum value plus one, but at least zero. And I
really, really, really disliked this and so I got rid of that part. So, on 8.0,
uh, it will be minus 41 while currently it's still zero. And because this may
bring some, uh, yeah, BC breaks, um, this was, uh, this was a slate that for,
for a new major version.

But for the rest of 8.0, um... OK, I have to switch modes. Well, I want you to
contribute to, to PHP and to help shape up 8.0. So now it's really the right
time to do that. 7.4 still being developed so you can still introduce new
deprecations for 7.4 for things that you really wanted to change on 8.0. So now
it's a good time to do that.

## Changing PHP as a verb

And so, let's talk a little bit about how you change PHP, about the verb part
of it. Um, but first I'm going to tell you a little bit my first, about my
first contribution and tell you a little bit the story about my first
contribution. Um, and it was like these. So, one afternoon, in an office
somewhere, uh, we were having a security scan, um, and certain requests were
hanging on certain conditions. Um, and basically it boiled down to the fact
that the text protocol of Memcached cannot really handle new lines in the keys
properly.

Um, and yeah, because it would hang or it would even allow injection. So this
is actually, this was an, uh, an injection for Memcached was possible to, to
abuse these, to get that. And this was easy to fix on our side, right? So just,
okay, don't accept new lines in the keys. Uh, but yeah, I started wondering
like why, why does PHP let these through? What, what, yeah, why would they do
that?

Um, and so, luckily in the PHP world, we are in the world of free software and
two of these freedoms specify that you are free to study how the program works
and change it so it does your computing as you wish. And I wish Memcached to
not be vulnerable to injection and then the freedom to distribute copies of
your modified versions to others. By doing this you can give the whole
community a chance to benefit from your changes.

Well, so let's, let's try to get that for everyone. And so I started looking at
the code of the PHP Memcached extension and I was positively surprised to
figure out that, well it's not rocket surgery. It's not that hard. And the
change in the end, in terms of code, ended up being just these and maybe it's
not very well visible, but I think that this is reasonable for anyone. This is
not very complicated. It's C, it's a little bit, syntax is a little bit
different from a PHP, but in this case, not even that much and this was all
that was needed to, to cover that, to cover that hole.

So then, um, well now what do I do with this? Uh, I emailed the
security@php.net because this is a security concern and they told me, yeah,
just open a pull request. Don't mention that it is about security, just open a
pull request and uh, and we will merge it eventually our. Okay, sounds good. So
I did, I opened the pull request, and it got merged, and since PHP Memcached
3.0.0, that's the problem no longer occurs. So, that's nice. So it's no longer
vulnerable to at least this type of injection.

### How you can Contribute

Then about your first contribution. First, I wanted to know, did anyone of you
already contribute to PHP anything in any way?

Maybe you did, but. Okay. So, let's, uh, first things first. Subscribing to the
mailing list is pretty much essential. So this is where all the discussions
about PHP happen. Uh, this is where the RFCs are presented. This is, uh, yeah,
where a lot of these discussions happen. This is also a nice place to get help
if you're trying to fix a bug and you cannot figure a step out, you can always
mail internals and there is these very nice website, externals.io that
basically just exposes the internals to the outside. Is a nice way to read the
mailing lists. It does the whole treating thing and yeah, it's, it's very nice
to, to read there. But you can always subscribe to the mailing list on
php.net/mailing-lists.php. So this is, for me, it's a good first step.

### Help with documentation

Then, helping with the documentation. Sometimes there is ambiguous language
that should be fixed or updated documentation or you can help with translations
to your own language. If it's not english and it's missing something and you
can visit edit.php.net, and it gives you this fancy interface with the facebook
like button where you can actually edit the documentation there and you can
submit your patch to review and then anyone with a php.net account, if they
find that this makes sense, they can merge it. And then the website is a
rebuilt every 24 hours. So it might take a while, but eventually your change
will be there.

There is talks of possibly migrating these to GitHub, which would make sense
and make things a little bit easier. But on the other hand here you can also do
it anonymously. So we'll see. We'll see what will happen with that.

### Bugs reports

Then - reporting bugs. So, we do have bugs.php.net. Um, and yeah, as with any
bug reporting, you should try to always provide the minimum reproducing code.
So, okay, this is all you have to do to have this problem. Any other useful
information and of course what you expect versus what you actually got. And in
this case it's a, it was a bug reported by Nicolas, caused by me and fixed by
me as well. So that was nice.

Then, um, you can also answer bug reports provided that you have an email
address or not. Maybe you don't really that and that you have some basic
mathematic skills - you can solve that problem you can contribute. Um, and
yeah, I mean sometimes you just know how to reproduce it with less codes.

Sometimes you just, you see that the person that posted the question is not
giving you enough information, is not giving anyone enough information so you
can tell: Hey, can you, please, be more specific or anyone can, can help with
this. So replying to bugs is all, so really, really nice. Um, and then who's
doing it here, also doing it on StackOverflow, answering people or on a, on the
chat of StackOverflow. Um, those are good. Those are good places to help out as
well.

And then about fixing the bugs themselves. So, PHP is on GitHub. It's a, it,
it's true. it's a mirror of git.php.net. But it has pull requests, you can just
do your change, open a pull request, like with any other project and the change
can be merged. So, yeah, it's pretty simple to start off.

Um, to help you with the, with your diving into the PHP source. Um, there are
some helpful tools. One of them is LXR, which is a way to look at the code a
little bit like an advanced idea because you can click on anything and it will
take you to that function. And yeah, you can see that one on the room11.org,
lxr.room11.org. Um, Room11 is the PHP room on the StackOverflow chat and a lot
of PHP maintainers or contributors are also their daily. Uh, so it's also a
nice place if you're trying to dig in and you don't, um, yeah, you need some
help. It's a good place also to find that help.

Then, you also have 3v4l.org, uh, which is really, really, really great. It's
a, yeah, it's a website where you can run a snippet of code and it will run it
on 200 plus PHP and HHVM versions so you can see exactly when a behavior was
changed in which versions did that happen. Um, and the shout out to Sjon for,
for making this and making it available.

Then, you also have wiki.php.net, which has a lot of useful resources if you
want to dig in. Uh, so yeah, so is a recommended reading. It has some things
that are a little bit outdated, but yeah, you will always be able to find the
interesting documentation there. And also on the, on the, on the repository of
PHP source itself. It also has the contributing file and it also has some nice
information and from these I would say to use your favorite editor to avoid any
religious wars. I did this. Um, so yeah, just use your favorite editor as long,
in my opinion, as long as you can debug properly with it, uh, feel free to
choose whatever you want to use.

### Writing tests

Then, writing tests. So, PHP tests look like this. This is a `.phpt` file. Um,
they have a pretty simple structure. So, basically, here we see it is, it has
the test name, the script itself, what it will do and what it expects. It's all
good. It will pass if the, what expects is not there, uh, it will fail. You
have another, you have a bunch of other sections. You can have EXPECTF instead
of expects to define it in a format. You can have INI to change some INI
setting specific for this test. Uh, you can have a SKIPIF so to, to run a
little snippet. And if, uh, if the output is "skip" - then the test is skipped
because the extension is not there or for whatever reason.

And then you can run the entire test suits if you run... make tests on the
repository or if you want to only test a specific test or a bunch of tests that
are related to each other. You can also use the run tests, um, well `make test`
uses run tests, but then you can use run tests directly to specify that you
will only want to run a certain type of or certain tests.

### Proposing changes

Then, proposing changes. So this is the RFC process. RFC stands for request for
comments. Um, and yeah, your approach should go a little bit like this. So the
first thing to do is to check what happened the last time that this was
discussed. PHP is a pretty large project. It has been around for a lot of years
and most likely what you are thinking about, somebody's already thought about
and maybe it was already discussed. So that's always a good starting point is
to figure out: Okay, what happened the last time that someone brought this up?

Then, if you're convinced, okay, nobody really, nobody talked about these are
the reasons why they rejected it before are not valid anymore. Uh, you should
email internals and introducing your proposal and listening to feedback a
little bit like this first phase of, uh, of feedback and actively listen to the
feedback and try to adjust your proposal to that.

Then, uh, you can write the RFC itself on the wiki and it will produce a fancy
document looking like this. Um, and yeah, and here you should really detail
everything that has to be said about the proposal, like a feature, uh, what,
what if it has any, uh, BC breaks. If it, if it has any future scope, like in
the future we could also do this, this and that, the proposal itself. So, yeah.
Um, and then, uh, preferably, this is not mandatory, but preferably you would
also implement our proposal.

Uh, but this is, as I said, it's not mandatory. It happens sometimes people
propose things even if they cannot implement it. Um, and then someone else will
pick up the implementation, most of the times. Um, versus the SameSite cookie -
a thing that I talked before, it was proposed by someone else, but then they
couldn't come up with a, with a implementation and I ended up, I ended up
implementing that myself. That with the options, a rare.

Um, then, you should start your discussion period, which should be at least for
two weeks. And, uh, well listen again to feedback, change things that have to
be changed or fight people until they said that, they say that it's not needed
to change anymore.

Um, and then you can start the voting period. The voting period should also
last for at least two weeks. Um, people go to the wiki and they can vote on it.
Uh, you can, uh, you don't need to have a php.net account to vote. You just
need to have a wiki account to vote and some people that are not directly
contributors to PHP versus Fabien has, has the, um, the possibility to vote on
the, on the RFCs. So, if it is accepted, the PR gets merged into, into PHP.

## Questions

And that's pretty much it. Um, I would appreciate any feedback that you have
about the talk in that JoindIn a link and a, yup, I would take questions. Does
anyone have any questions?

Uh, they will start at 0 if you don't say anything. They will start at 0
anyway. Uh, this will only change if for the first keys that you use are
negative, right?

No, sorry.

Okay. Yeah, that's why we'll wait for a major version. Right? So, instead of
like I propose these initially for 7.2 I think yet, and they're like: Oh, of
course not. It's not an a minor. Are you crazy? Oh, hey, sorry. Uh, then, well,
then I do it in a major. Is that fine? Yeah, on a major is fine, okay. Then
I'll do it in a major. So, yeah, so that's basically the reason why we will
wait for 8.0 to do this change. Because, yeah, it might break some things, or
some people if they write code, like if you write code, if you're using
implicit keys to add things to the array and then you use an explicit key to
access it - you're probably doing something wrong. Right? I mean if you don't
care about the keys, you don't care about the keys. Or either you care or you
don't. Make a choice. But, yeah. Anything else? Yup.

I shouldn't say. Come on, I did all the effort to not say, but okay, I use the
VS code. Uh, and it works really well for me. I use it on Linux and it
integrates really well with the GDB for debugging, so I can just add
breakpoints and it breaks and I can see the state of the entire application.
For me, it works really, really well. But yeah, I won't say more than that.
Anyone else? Yup.

No, not strong. Uh, actually like probably if I will have to write C outside of
the PHP context, I would be a little bit lost to be honest. Um, but yeah, it's
uh, I don't have a strong background in C. I learned a little bit about it in
school, but that was it. And by day I work with PHP, so that's what I'm used
to. But uh, yeah, as I said before, it's not rocket surgery. It's not that
complicated. It's, yeah, you'll get used to it quickly I would say. Anyone
else? Okay. Well, then that's it. Thank you.
