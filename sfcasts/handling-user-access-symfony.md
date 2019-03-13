# Security: Handling User Access with Symfony the Right Way

***TIP
SymfonyCon 2018 Presentation by [Diana Ungaro Arnos](https://twitter.com/dianaarnos)

We often overlook a central security requirement that any application needs to
meet: controlling users' access to data and functionality. Usually, we handle
user access through the combination of 3 security mechanisms: authentication,
session management and access control. We will take a look at the Symfony's
Security component powerful tools and see how to use them to handle user access
the right way.
***

Good morning everyone! Did you drink your coffee already? So, because you know,
if you feel a little bit sleepy I can scream really loud, and I maybe do that
sometimes during the presentation, please don't be scared. I promise I'm a
really nice person, especially when I'm sober, which is right now. So we are
fine. Uh, I must say sometimes I say some maybe bad and ugly words and if you
feel offended by it, (xxxx) you. Oh no, sorry. If you feel offended by it, okay.
I can ask, you can come and ask, I can ask you a, you can ask me for apologies
later. And no problem, I can give you hugs and everything is fine.

So, well, um, I, I must say also that I'm from Brazil, I'm Brazilian, so, uh,
I'm sorry if I mess up some words in English because, well, it's not my mother,
my mother English, it's really fine. That's not my mother language. So I'm
really sorry about that too. And maybe no, (xxxx) you again.

So I'm here to talk about security. Um, what I want to talk about is, uh, the
Symfony framework that have, um, amazing tool called the Security component. You
may use it with your whole application running on Symfony or you can install it
by small parts, by packages using Composer.

And while I was studying this component, it is extremely powerful, but I've been
searching for, you know, examples on the Internet. Um, examples of code, I, I
read, uh, lots of lines of code on GitHub. Um, I talked with people I know about
the component, the component, and I realized people, like, open them the
documentation, see some lines of code and it's always Ctrl+C, Ctrl+V. People
don't understand the concepts behind, um, some security problems and security
solutions. Okay? Um, if you don't know the concepts, if you don't know how
things work, if you don't know the context you're dealing with, I'm so sorry
you're a (xxxx) developer. Okay. Um, and this is not a problem. Everyone is a
(xxxxx) developer on some level or something like: I can do frontend. I mean,
css hates me, so that's why I stand in the back end stuff. And especially for
starting, if you're new you, if you're a junior developer, it's good to be a
(xxxx) developer, you know? Wake up every morning, look in the mirror and say:
I'm a (xxxxx) developer. Because this will make you try harder when you go to
work, when you try to study. Okay.

## Hi Diana!

So I'm going to play around here with some concepts. And I will present some,
some of the functions and tools you can use for, from the Security component to
handle user access specifically. We have some stages of user access, but before
I enter that, that's me. Um, there's this... Oh! My light doesn't work. So you
can find me everywhere with Diana Arnos. That's my name. Uh, I like Sec, Music,
I am tech leader from the startup in Brazil and I am evangelist of the user
group of php and São Paulo Brazil and the Brazilian chapter of PHP women that,
yes, we have a Brazilian chapter of PHP women, okay?

## User Access

Well, let's talk about user access. When we think about user access, well, it's
easy right? You just have to put some username, you have to match up the
passwords and let the user in. It's like, oh, this is you, so you can get in.
But is this really that simple? How, how do you think of when you imagine the
user entering your system, what things our user can do? You know, it doesn't
matter what kind of user we are talking about. If there's one thing we know,
every user is really good at, the user is good at (xxxx)ing up things. The user
will click where it shouldn't. He will try a URL it shouldn't. He will end up at
the page you didn't even imagine it existed, especially for dealing with legacy,
if you're working with legacy code.

So we have a few steps to think about, a few situations when we talk about
access. You have the user and you must, you can notice I'm a really good artist,
you know, and then you have to do the auth. Uh, it's interesting to notice that
when we talk about auth and we use only the abbreviation, it's because it's the
same for authentication and authorization, because it's, they are separate
things. Okay. Not many, not many people think about that, but they are separate
things. Okay, so you'll have the authentication step.

After the user is logged in, or it's authenticated, it will have access to many
features, whatever they are in your system. Those features enable the user to
access some data. It may be only visualization, it may be only to edit or
create, whatever. Every data on your system is accessed by the user through some
features. Yeah, if that doesn't happen, you're having a problem, around, like...
I will try to not talk about sql injection because you know, it's a very common
mistake people usually do, especially developers that they don't pay attention
to what they're doing. But I'll try it hold myself, it's not what I need to talk
about, but I really want to punch everyone in the face when talking about that.
So you have session, and you have an interaction with user authentication
session and data and feature all the time through all the system and actually
everything at the same time. It's much more complex than we usually think about.
And then you have the authorization process: that's involved around everything.
Basically the authentication process is... and the user data you have, they're
combined throughout the system to know, okay, does my, does this user have
access to this data or to this screen, to this button, whatever. You know, when
we talk about not even only user roles, but what can the user access, it may be
not only a simple visualization, you know, maybe as a low security-level user,
couldn't see the names of the high directors of the company, for example.

## User

So we have, everything begins with a user. You can't have an authentication if
you don't have the user. And let's suppose we are using Symfony here. It doesn't
matter how you create your user class. The question, you may use, you may cod it
by hand or you may use the MakerBundle, which is something very fun to, to play
with, but you have our user.

## User Provider

But then we have something really important for all the authentication and
authorization process that will be used by the Security component: that's a user
provider. What does the user provider do? It gets, um, like, let's say, support
methods that handles your user data in ways authentication can do with it. You
have two basic methods that must be implemented by your user provider and, but
you can also add a few more, once your start playing a little bit more with the
Security component and specially Guard authenticators that we'll talk in a
second, okay? So you have a user provider. The user provider does basically two
things,

### Reload from the Session

I must say, it *must* do at least these two things. First reload from session.
There is a way to disable that and I'll talk about that, but what is the reload
from session stuff? It serializes the User object and at the end of every
request, every request the user gets, the user object gets serialized in
session. And after, every time it begins a new request begun, or begins, sorry,
uh, it deserializes the object. But it's not only that, it deserializes the
object from the session, then it makes a refresh from the user based on the data
on the session from whatever your user data is: it may be an API, please don't
do that, let's do REST, but it may be an AP, it may be the database and it may
be, I don't know, memory, because it can do that too. Please don't do that, but
you can do that from memory too - configuration files. And then it compares:
it's like, okay, I have this user, then I'll get this user from my database. It
compares both and sees: Oh, if it's not the same user, it de-authenticates the
user: it makes it login again. There's some internal methods that verify this
that's outside from the, the part of the user provider. You can play around on
that too.

But it's a security measure to guarantee that, well, if you have some kind of
middle-man, middle-man, it's a very nice name, you know, a man in the middle
attack, okay, if you have any kind of man-the-middle attack, um, you have many
kinds of men in the middle, but one of that, one of the most dangerous attacks
on that is, right playing with the user session. Cause, you know that that
things start there. So let's suppose I am the man in the middle. Then I will try
to get the data from your session and then I'll start playing around because the
server will think I am you. And still, still the, the, the component doing that -
verifying if there was any change between the original user from the user
database and the data on the session, there's still ways you can play around and
(xxxx) this up. Because, well, if there ain't no time to updated the user or the
data didn't change anything, you can still have problems with that. And you have
ways to deal with that here too. Because, you can use a class that's a, an
interface you can put on User class and you can implement them a method called
`isEqualTo()`. So, instead of using the, the, the, the default method from
Symfony, you can write your own. But be careful, great power, great, brings big
responsibilities.

### Load User

And then you have the second method you must implement that is the load user.
Uh, you may look and say: but they are the same thing! No, they're not. The load
user does not do anything with the session. The load user, every time you have a
feature that needs the user data, it will be calling this method. Why would you
implement the load user? Well, like, if you don't want to use the default
methods and you are using like an API, the built-in user providers that Symfony
comes with does not give you the methods needed to get a user from an API. So
you can implement this here. Okay.

## Custom User Providers

And here, and this is one of many use-cases that can do, you have to get the
user from an API. And now I feel a little bit better talking about that because
ya know, you guys were watching Anthony's talk and he was saying to please don't
use microservices, and I was going to say if you're using a user micro-service.
So I can say we're using a big user service, okay? We're going to recover the
user data from the API. These are the methods you're going to mess around.
`loadUserByUsername()`, funny thing is, that's, the username is not necessarily
the username. It's the thing that comes if you have... It, it's the data that
comes when you use the method, `getUsername()` from the user. Oh, but if I'm
using the User class that's default from Symfony, you will get the username.
Okay. But you can also make your custom User class, so you can change your
username, I dunno maybe your identification of some kind. But then you can
create and code all the data that comes here and this gives you really power
over your application. They get, this gives power to: Okay my user is logging
in, I am authenticating this user. How am I to authenticate this user? And uh, I
know that most people doesn't care about that. People like, okay, I have used
the full functions. Don't do that, please.

Well, everything, as everything you can do on the Security component, you
configure it in the `security.yaml`: that's right:
`config/packages/security.yaml`. Uh, when you make your custom provider, you can
set them here and you can just add the id, user provider. Uh, I didn't show the,
that piece of code because I was showing the things you needed to, to code, but
at the end of the class, there's a method that supports class, that's called
`supportsClass()` that will show the framework you're using the class as a
custom user provider. But it's a, it's a, it's a common line. You can find the
line of code on the documentation itself.

## Authentication

After the user: Ok, I have my user, I have my user class, I have my user data
and I have a user provider. The user provider will always be used during the
request and authentication and authorization methods. Okay. I did this step. So
now I'm going to talk about authentication.

And we have authentication providers. Uh, I don't know, most people, first time
they're trying to mess with, uh, the Security component, they are trying to do
things from out-of-the-box, because you can do that. It'll just put: okay, I
created a project, I have my user, I can use the MakerBundle, everything's fine.
So, you need to fix or do something different with the authentication. And then
you start, like jumping from class to class and you're like: oh my God, this
authentication is magic because it's not in the controller, it's not everywhere.

The authentication on on Symfony runs before the controller is called. It's
almost like a, a middleware idea, it's not exactly a middleware, but it runs
before the controller. And for that, you can create authentication providers.
Symfony already has some authentication providers built-in. But if you read the
documentation, they will ask, they will say it's better if you create your own.
But here, you don't have custom authentication provider, you have what's called
a guard authenticator. But let's suppose we're going to, you have... When you
create a user provider, you have one user provider for your class. Here you can
have lots of authentication providers. And every authentication provider is,
they always runs before each request. Always. And so you may have, you need to
pay attention to the order things are happening. How many providers you have on
authentication. And you must take care to, don't let one mess up the other.
Okay?

Umm, okay, I won't use the built-in, please don't use the built-in, I repeat
this many, many, many times: please don't use the built-in authenticator. And
then, you have to do your own. You can create a guard authenticator. It runs at
the, it's the, on the same, the same context; they run every time before each
request. You can create lots of guard authenticators. But, this class, when you
create a Guard authenticator, it gives you full control of the authentication
process. You can, you can analyze and, you know, deal with data since the
beginning of the request to the end of the request. You will use an interface
that brings seven methods you need to implement, but you can put more, and you
can do a lot more.

There's an example. Um, I'm really sorry I messed up the PSRs here, because you
know, but it's just so they can fit all in. These are the seven methods you need
to use, at least, by extending the `AbstractGuardAuthenticator`. Okay? Uh, why,
um, why would you, need most authenticator, to create one by yourself? You know,
like if you want to use a JWT authentication, API tokens authentication, you
don't have those built in on Symfony. Anytime, basically everything you need to
do with an API, you have to create an authenticator, a specific authenticator.

So, here you come from, since the beginning, you know, this function here, the
`start()` one. It's basically, it tells the authenticator what to do when, okay:
this user isn't logged in. Or this user is incorrect. It runs the `start()`
authentication. Uh, `supportsRememberMe()`. this function, remember me, you find
on the user provider class or the user class, it's basically to get the user
from the session and keep it logged in. You may disable that if you want to. You
may check credentials, um, what we'll do an authentication success. If you look
at this class and if you look up on the Internet for examples of its
implementations, you'll find that some, a little bit of similarity with
programming by events, you know: when it does logged in, when the request
starts, when the request ends, when it's not the same user, when they users
change. Okay?

And to configure that, you can do that, or again, on the `security.yaml`. Here
is a firewall. I'm not talking specifically about the firewalls because it's a
very extensive subject and it's very fun to play with too. But let's suppose we
all know what we're doing with the firewalls. And, pay attention, if you don't
know what you're doing, you're going to mess up your whole application. But down
there, in my main firewall, I have some authenticators. And why is this
authenticators? You can have lots, as I said before. And then you just put the
namespace of your personalized authenticator. It's really simple when you do
that.

Uh, I would like to call the attention to the bottom lines here, because there's
an option in your firewall, in your configuration that's called `stateless`,
cause I'm talking a lot about recovering user from session. There is a
serialization that goes on session of the user, blah, blah, blah, blah. When
you'll turn on `stateless`, you can work with REST API's, because there's no
state on your session. And I must say it's one of the best ways to do that. I
try to, to think like that, because you can avoid a lot of security problems by
doing that. But you can do `stateless: true`. If stateless is true here, you
don't need to implement that method that, that's that load users from session.
You can leave it blank. We can leave it whatever message, because when your
application is configured for stateless, they won't be calling that method any
time soon. So.

## User Roles

And then, you have, I can say, talking some of, like, it's like a sneak peek on
user roles, because you know, it's again, a very extended subject, but before I
talk quickly about user roles, I will like you to think about, you're maybe
thinking, man that girl is talking (xxxx) the whole time, why I'm sitting here,
blah blah blah, user here, blah, blah, blah, blah again and blah, blah, blah,
stateless. But to get to here, because every, everyone loves to talk about ACLs.
Okay, I have to handle access. I have user roles. But notice, pay attention to
how many steps you must think before you get to user roles. If we're not
thinking about that, you may have problems. That... you *will* have design flaws
in your application and that can be, and will be, exploited by malicious, I
would say malicious attacker, but an attacker is malicious anyway, so.

Um, user roles: this is a quick configuration, You'll do that on your User
entity. Um, you have them, this method `getRoles()`. Um, the big thing here when
you talk about user roles is that you can have a hierarchy of roles, and, you
won't get this: Okay, I will make this admin role and then I will configure this
admin to have access to this page, that page, will visualize this data, that
data, but actually you begin configuring your roles from the bottom to the top.
So when you talk about the admin role, you just have to say that the admin role
have all the others. Uh, another interesting fact, when you login or access your
application, without being logged in or logged in as anonymous user, uh, the,
the, the component have a very interesting concept that, uh, it's like, it's not
exactly that, but it's like the anonymous user, it's kind of a user role. So
even if you're, okay, you're coding then, and you're and you're trying something
and you have the error on dev mode and you have the profiler bar under your
application, if you are like, anonymous, okay, I'm not logged in and I'm, I'm on
the poor mode of the browser. I mean the anonymous mode of the browser, and you
answered that and you go check if you're authenticated, because you have the
user information on the status bar, you'll see that it will show you that you
are authenticated as an anon. And basically, uh, the user role anon don't have
any permissions. So, every time you are not logged in, the application and
treats you like you are logged in with a user to have no, no permissions at all.
And this is a very interesting fact because when you're coding your application
and thinking about the security issues or access, or permissions, you don't have
to think on the "no" option. It's like: okay, I will remove this and that and
this guy can do that, no, he's logged in as anon. And you can even mess with
that and give the anonymous user access to some features of your system or your
application without being logged in. And you can do that simply by using a
(inaudible) the configuration, setting up some paths or URLs or patterns that
you're, the anonymous user can have access to. Okay? And you can mess around
with that too. You can, you've: okay, you have access here and I'm using the
Guard authenticator and for any anon user, it can only see this kind of data,
not that kind of data.

And this is where you would configure. The guy here: `access_control`. You have
your firewall configuration. I was, as I was talking, the anonymous user there
in the main. Then you have `access_control`, so, you can play here. This, this
makes your life a lot easier when you will filter access on your application.
You may give that: Okay, I will enter, um, I don't know the welcome screen after
the user has logged in, and I have a dashboard, but I have a specific dashboard,
I don't know with financial information that only the admins can see. So when I
have a low user accessing, I can author that information right on the template,
on Twig. You can call the user role that, it's like app user access.

But let's suppose you're not thinking of, on that kind of detail. You can do
something quick, or quicker, or faster. That's the better word to say that. So
you have it here: `access_control` and then you can say right here, like what's
the role? Which role have access to which pages? Or you can think about path,
you can think about URLs or hosts, that's the option you can use, and then you
can make everything just from the configuration file. If I have a simple
application, if I have, a with only like editing, I dunno employee's
information, you can set it up through here. And every time we need to change
something: Oh God, the director's said that the guy from the next department
have to see the page "B", now, and he can only see the "A". You don't have to
change on lines of code, you can do right here. But of course you can only do
right here easily if you pay attention since the beginning: user, user provider,
authentication, guard authenticator. How does, how do I handle my requests
before I reach the controller? Because with, sometimes we are used to doing
authentication on the controller. That's not one of the best ways to do that.
So. And that's how you can play with user roles. Okay.

Um, I could talk a lot more about that, specifically security, I like to talk
about that, but unfortunately I won't have time to talk about all the stuff that
comes from the security component. But I would like to say thank you. It's a,
thank you for being here. Thank you for listening to my bad English and I'm here
to answer all your questions. Not only here, but I'll be here all day. So come
and talk to me on Twitter and Instagram because I'm kind of hipster developer.
So if anyone has questions, or you may, I don't know, curse me? I can hear that
too.

Thank you. Thank you.
