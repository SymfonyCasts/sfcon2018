# Back to the Basics (Symfony local Web Server)

***TIP
SymfonyCon 2018 Presentation by [Fabien Potencier](https://github.com/fabpot)

Keynote presentation covering Symfony's local web server.
***

Okay. Welcome. Good morning. Thank you.

So if you are in the stairs over there, I can see empty seats everywhere here
in the front. So if you can come, as you like. Okay. So today's talk is about,
it's a series of talks. I'm going to give over the next few conferences. Um, I
talked about emails during SymfonyLive London a few months ago and today it's
going to be another talk about getting back to the basics. And the topic is a
local web server. Are you excited about that? No. Okay. Too bad anyway, Even if
you're not excited about that, it's going to be what I'm going to talk about
today.

## Local Web Server?

Okay. So a few month ago I talked about and, and, and actually I asked people
on Twitter about which a web servers that we're using on our local machines,
uh, that's the results. So as you can see a lot of people are still using `php
-S` which is a builtin a php web server. Um, and also using the symfony
server:start, which is exactly the same because we understand that we are using
`php -S`. Anyway, a bunch of people are using Docker, Nginx, Apache and
Vagrant. So the problem with that is that `php -S` is great. It is very easy.
You need to install the WebServerBundle and then you can get started really
easily. The problem is that it comes with many limitations and I'm going to
talk about that in minute.

Docker, well may be slow depending on the platform, especially on a Mac. I mean
it is for me at least. Um, and it's kind of inconvenient and I'm going to talk
about that as well. Um, Nginx and Apache, it depends on your skills on those
um, uh... programs, but it needs some setup and it's not that easy. It's not
really nice if you ask me. The great thing about using Nginx or Apache on your
local server, it's probably the same that you're going to have on your
production machines. So that's really nice and Vagrant is also not very
convenient because it's, you know, something you need to install and then you
have some kind of, virtual machines. So. Okay. So that's my setup.

## Ideal Local Server Requirements

The setup I would like: something that is really fast and when I say fast, um,
I want the web server to be really responsive. So I don't want to wait for
something to happen. And convenient. And by convenient, I mean I want my files
to be locally on my laptop. I don't want them to be on a Docker share or NFS or
whatever. Or on the virtual machine because then it's going to be slow. I don't
care about services so I don't care if my MySQL database or Postgres or Redis
is on my laptop and actually I don't want to install anything on my laptop. So
I like to be able to use remote services, it can be Docker, can be Symfony
Cloud, whatever. And I want something that works on all platforms. Not that I
have three different laptops, but just because I care about people in the room
not using a Mac.

How many of you are using Windows? One, two, three, four, five, six, seven,
eight, nine, 10 to 12. Okay. Linux? Much more. And a Mac? About the same. Okay,
cool.

So I was talking about the PHP built in server, it's almost perfect, but it has
many limitations. So the first one is that it can only do one connection at a
time, which is kind of annoying sometimes when you have a bunch of ESI's for
instance and things like that, your website can you kind of slow, even locally.
No support for HTTP-2, no support for tls, which is kind of a requirement
nowadays. And if you are using Symfony 4, with environment variables, you might
know that it does not reload environment variables from one request to the
next. So if you change your variables then you need to stop and restart your
web server. And also it needs some tweaks to make it work with Symfony. So if
you are just using `php -S`, like that is not going to work well. It kind of
works, but we have a special router to make it work. That's why we have the
WebServerBundle. But it's nice because it's local, it's using my local files.
It's really fast and it just works.

## Introducing the "better" local web server

So in the last few weeks I've been working on a better local web server and
that's what are we going to talk about. And it's available via the the new
symfony binary. I'm not sure if you are all aware about that. I Tweeted about
that yesterday or earlier today. I don't remember. And we had a blog post about
Symfony cloud a few days ago and everything is available via a new symfony
binary. So the new symfony binary is not just about Symfony Cloud, it's about a
bunch of things.

## symfony security:check

So I talked about two of them last week in my blog post about Symfony Cloud.
The first one is security checks. So if you want to check your composer.json,
for security issues, you can use security.symfony.com if you want and the API.
Or you can use `symfony security:check`, which is nice because it does not
connect to the API. So it's, everything happens locally. So it clones a GitHub
repository but then everything happens locally. So that's a nice way to not
depend on security.symfony.com.

## symfony new

And symfony new is a way to create a new symfony project. It's just a thin
wrapper on top of a composer, so you don't have to remind the composer
create-project symfony/skeleton thing. You can just use `symfony new`. By
default it creates a project with symfony/skeleton and then you can say I want
a demo or a full website which is using the `symfony/website-skeleton`. And
then it does a few other things. The first one is it does `git init`. It also
configure everything for Symfony Cloud if you are interested in that. Okay.

## symfony server:start

And of course, as of today, it's also about the symfony local web server. So
that's how you can use it. You can just say `symfony server:start` and it
should just work. So it uses the first available port. Instead of 8000 you can
configure it if that makes sense for you. And then you have all the logs from
PHP and, and a bunch of other things. You can also start it in background with
`-d` and the great thing about that is that then you can just fire and forget.
And the great thing - and that's very different from what you get from `php -S` -
that you can just say `symfony server:log` to tail the logs of the server,
which is not possible with `php -S`. And then you can say `symfony open:local`
to open a website in the browser.

## Symfony server Features

Okay, so some of the features from the symfony local web server. The first one
is don't need to be root to run it. It runs on a port more than 8000. So that's
not a problem.

It uses `php-fpm`, which is very important because it's more similar to what
you would get in production. But it does more than that because I realized
while working on that and I wanted to make it work on Windows. And now I realiz
it was not really necessary because you are not that many. Um, it took me a lot
of time really. But now I have a virtual machine on my laptop with windows,
which is kind of nice. I mean, Windows is much nicer than it was 10 years ago,
so that's great. But php-fpm does not work or it is not available on windows.
So when you don't have php-fpm, it falls back to php-cgi, which is also
fastcgi. And then `php -S` if there is nothing else. It supports HTTP-2, it
supports tls, some push support. So if you have some link headers and - the
support and we do have support in Symfony - then it's going to work with this
web server.

It works with any PHP project, there is no, nothing special. So there's no
configuration, nothing. It just works with any PHP framework or CMS's or
whatever. You can also use it, use it for a plain HTML websites, if you have a
nodejs thing, um, or something that is only on, on, on, on, on the frontend.
And hopefully it's a great developer experience. And we are going to talk a bit
more about that.

## Managing Multiple local versions of PHP

So the first big feature I wanted to mention is that it also manages your local
php version. So it does not install php itself, but if you have more than one
version on your machine, then you can use them. And that's very interesting
because it allows you to test a new version of PHP, uh, out of time. So if you
want to test your website on php 7.3 for instance, that's a use case. Or if you
want to test a different configuration with, with, you know, a different
`php.ini` file or something like that. And of course if you're working on
different projects, you might need more than one php version. Um, and actually
when I started to work on that, I realized that most people do have several php
versions on their machine even if they are not able to access them. So for
instance, if you are using the Mac and you're using Homebrew to install your
php versions, you do have more than one version because whenever you are
upgrading to new version, the oldest ones are still there on your machine. And
you can switch from one to the next one, but that's the global configuration.
So I wanted something that is local and not global. So there is a command to do
that. So if you run this command, you will see all the things that you have on
your machine. That's what I have right now on my machine and it detects the
kind of things that you have: php-cli, php-fpm, php-cgi and things like that.
And you can see which one is going to be used for the web server. And a default
one.

So if you want to change them, if you want in a directory for a project, if you
want a very specific version of PHP, that's how you can do that. So you can
create a `.php-version` file. And then, um, I can say I want php 7 or php 7.1
or php 7.1.13 or whatever, and it's going to use that specific version in that
specific directory. So it's not going to be global, it's really local. Okay. So
that's great. Um, so when you, when you run `server:start` depending on the
current directory, it's going to check for a `.php-version` file. If there is
one, it is going to use the version from that, if not, it falls back to
`.symfony.cloud.yaml` and by default it uses what you have in your path.

Um, it also, it's also about having the best backend possible. So again,
php-fpm by default, if it is not available, then php-cgi, and as a last resort,
it uses `php -S`. Okay, it also works on your terminal. So if you want to run
PHP and have the same version of the local web server, that's how you can do
that: `symfony php` and whatever. It uses the exact same algorithm as the web
server of course. So that's very nice to be able to have the same version for
the web server and also for the command line. Um, and you can also configure
PHP per directory. I talked about that earlier. So if you have, if you need a
special time zone, for instance, for a project, that's how you can do it. You
can create a `php.ini` file when you have some kind of php configuration there.
And then it's going to be loaded automatically by `symfony php` and also by the
web server. So you have the exact same configuration for the web server and
also for php in the command line. Okay. And of course, that's something you can
do as well. You can just copy the symfony binary and name it php, pecl,
php-fpm, php-config, and it's just a thin wrapper. And it's going to act as
`symfony php`, which means that that's exactly what I have on my laptop, which
means that whenever I type php something, it's going to use the right version
for the, for the directory I'm in, which is kind of nice.

## Logs, Logs, logs

Okay. I talked about logs. Logs are very important if you wanted to debug
things, you need to have all the logs failure but very easily. Um, so that's
how you can do that `server:logs`. And then it's going to tail all the logs
from php, of course, from php-fpm, but also for from symfony. So as you can see
here, we have some logs from symfony, uh, and as you can see, it's not exactly
the same as the logs that you have from the `var/log/dev.log` file, because we
actually parse all the logs, we convert them to semantic JSON, and then we kind
of humanize them so that it's easier to spot problems. I don't know for you,
but sometimes when I'm tailing logs, especially for the production environments
and there's some kind of a problem I need to debug the problem, it's kind of
hard to figure out where the problem is and where you got your exception. So
here we make it very clear. So if there is an error like we have here, you have
this red thing so that you can spot the problems really easily.

You can also use it even if you are not using the web server by passing a file
directly to the command and it's going to read the file and parse them and, and
you manage them.

## Using TLS Locally

Okay. The next thing is TLS. How many of you have a website in production
without https support? Ah, you can raise your hand. Okay, I'm going to raise my
hand. Nobody really? Okay. The, so that's, just one, he's not here, he's
somewhere in the room. So TLS is very important nowadays. You need to have
https support so I think you need to have support for your local, local server.
It's more similar to production. You can detect mixed content early on which is
already, always a nightmare if you not detect that before going to production.
And sometimes it's also a requirement for some JavaScript libraries. For
instance, if you are using stripe, if you don't have https it's not going to
work. Okay. So the first step is, and it works in the exact same way on the
three platforms, you can use `server:ca:install`. So what it does, behind the
scenes, it creates a local certificate authority. It is going to register it on
your system trust store, which is different depending on the platform. It
registers it on firefox because Firefox is using its on mechanism for that. So,
uh, that's also something we do and then it creates a default certificate for
localhost. Uh, so that when you actually access your website is https, you have
a default certificate to work with. So it's everything is stored under your own
directory, under the `.symfony` directory, and then `certs` and you can see,
um, the, the local certificate authority and a default certificate.

And done. So now when you start a server and you have this tls support, you are
going to be using https by default. And by the way, we have http and https on
the same port. So you don't need to remember two different ports for that. And
by default we redirect http to https.

## Local Domain Names / proxy:start

Okay, so the next step is having local domain names. And that's also nice
because of a bunch of reasons. So the first one is that, you know, some project
depend on the domain name. That's the case for the symfony live conference
website for instance, we have Lisbon, we have Paris, two different domain
names. And depending on the domain, the website is not going to be exactly the
same. You don't need to remember the ports, so don't need to remember that this
website is on port 8000 or that one on another one. So you have something that
is stable, it can be stable across machines as well. We can use the same domain
names on two different machines and it uses regular ports. So it's, you're
going to use port 80 and port 443 for https, which is again more similar to
work you would have in production. And that's always a nice to have.

It's totally optional, so you don't need to use it. If you want to use it, you
can start a proxy. So `symfony proxy:start`. you don't need to be root. And
that's very important for me. You don't need to be root to use the proxy even
if you are using port 80 and 443 just for a reason. And the reason is that we
are using a proxy configuration that works in the exact same way on the three
platforms. Uh, so you have nothing to install, you don't have to install DNS
mask or whatever. You don't need to mess with your local DNS configuration, you
don't need to edit your `/etc/hosts` file, for instance, and it does not
intercept regular traffic because that's the configuration. So if you want to
have a look at how it works, you can curl this URL and you will see piece of,
JavaScript, and if the domain name is wip, which is the default TLD we are
using for the proxy, then it's going to use the proxy. If not, it's just, you
know, doing nothing and, and, and, and not intercepting traffic.

Okay? So, so the first thing you do is you attach domain to a directory so you
can do to your project directory. Then you attach a domain. So here I'm going
to attach symfony.com On this symfony.com website, and then you start the
server if it's not already done, and it's going to attach a domain name with
the right port. So that's done automatically. You don't need to configure that.
You can edit all the configuration under the `proxy.json` configuration file.
So wip is the default if you want to use .sf that's also possible. I think
that's kind of nice. Um, we have wildcard support, so if you need more than one
domain for a project that's also possible and all the tls certificates are
going to be created on demand locally, so we don't need to be connected to the
internet to make it work. It works locally, so it's not using a, it's not using
a letsencrypt, for instance.

Uh, the nice thing about the fact that you have a proxy for the whole tld is
that if you are hitting something that does not exist, then you have, um, a
good error message saying that, okay, it's not linked to any directory. If you
want to do that, you can run this command to make it work. Um, and if you go to
the proxy directly, you have the list of all the domains that you have in all
the projects that you have on your machine that can just click on them. And it
does work with curl as well, so you can just export `HTTP_PROXY` and
`HTTPS_PROXY` with proxy url and it's going to work for curl or, or whatever
you're using because of course, not of course, but curl does not use your
system cert store. So you need to do that to make it work, if not, you're going
to have an error.

## Workds, Daemons

Okay, so now I'm going to talk a bit about the integration with Symfony because
in the Symfony world and it's a trend that we can see over the last few years,
I would say, where we have more and more long-running commands in the Symfony
world, right? We have one for a dumping, the var-dumper, things that you might
have in your project. We have one to consume messages from the Messenger
component. We also have, Encore, if you are using Encore or yarn or whatever,
you want, you might want to watch what's going on in your project so that I can
compile their assets whenever they change. So we do have support for that as
well. A `symfony run`, I'm not sure I mentioned symfony run before. So `symfony
run` is a way to run a command with some nice features and I'm going to talk
about them after the slides. Anyway, so `symfony run -d` daemon and then you
can type whatever you want and it's going to run in the background for you and
manage the background process.

The nice thing about that is that when you are using `server:log`, then you are
also going to have the command log, so all the logs from php-fpm, Symfony
project and also all the commands that you run via the `symfony run` command.
And if you say `symfony server:status`, you can see that you have the web
server and the command that is running on this PID. And you stop the server
then it's going to also stop the command. So you can run any numbers of
commands and they are going to be attached to your project.

## Integrated with Symfony Cloud, Optimized for Symfony

Okay. Of course it's also fully integrated with Symfony Cloud and optimized for
Symfony projects. So the previous one - you will be able to take a photo in a
minute - at the previous one, so you know encore it's, it's not really linked
to Symfony, so you can use it for any project. But now I'm going to talk about
things that are really optimized for a Symfony project. So I talked about logs
and Symfony logs that we parse. But we have more than that.

So the first one is that, and I talked about that at the beginning of the
session, I want to be able to use my local server with my local files, but I
don't want to install a database server or redis or whatever on my machine. So
I'm going to show you how you can do that with Symfony Cloud. And I think
that's a very nice scenario. So let's say that we have a project hosted on
Symfony Cloud with the demo.com domain name. That's how you can get started.
You have your project, you run `project:init` so that you generate a default
configuration for Symfony Cloud. Then use `symfony deploy` it's going to deploy
a cluster of containers, for php of course, for all your services. It's going
to provision a tls certificate as well, and then when you deploy, it's going to
be your master branch, which is the production environment in terms of Symfony
Cloud. And then you can attach domain more or less like what we had before for
the proxy and it's going to attach a demo.com domain to your project. And by
default, of course, the environment is production and debug is set to false.

Now, let's say I want to work on a new feature or I have a bug in production
and wanted to debug the problem. Instead of working directly in production and,
uh, um, or working on my local machine with fake data, I want to have a
replicate of what I have in production. So what you can do is, you can use the
`env:create` command. It's going to create a new git branch for your code
locally, of course. Then it's going to deploy a new cluster of containers,
which is going to be the exact same cluster as what you have in production. So
the same services, but also it's going to copy the data, the exact same data
that you have in production, and then you can use `symfony open:remote` to
get... the website is going to be exactly, exactly the same as production with
environment production as well. Uh, let's say that you have a problem, you can
use `symfony:log` to get all the logs from the remote web server, so that's the
same as `server:log`, but for the remote services. But by default we want to
replicate the production environment. So the environment is production, even
for this branch. If you want to switch to the debug mode you can use `symfony
debug` and it's going to switch to the development environment and set debug to
true.

## Tunneling Services Locally

Okay, so that's remote. I have all my services remote now. I want to debug
things locally on my machine. So I have attached my local web server to the
`demo.com.wip` domain name. I'm running a local web server and if I opened the
website I'm going to have an error because I don't have the database locally.
Right? And I don't want to install the database locally. So what I want to be
able to do is develop locally with my file on my laptop, but I want the data
from the production environments.

So the way to do that is that you open a tunnel between your local machine and
the remote services. And that's all, that's all. You refresh the page, don't
need to restart anything and it just works. It just works because behind the
scenes we are getting the information from the remote services and we are, we
are exposing those information to the web server directly. And as you can see
on the, on the right side of the screen here, we inject a piece of code in the
web debug toolbar. It tells you that the tunnel is open for the `some-bug`
branch. So that's what it is using. And if you have a look at the profiler. So
that's my `.env` file here in my project. You can see I'm using the sqlite
database, uh, and I've removed the file. So that's why I had an error. And here
you can see that it's not in the .env, uh, on the right side, which is the
profiler, it's not there because by default the local web server detected the
tunnel and it exposed the service as environment variables in php-fpm
automatically. And as Symfony and Symfony Cloud share the same conventions, it
means that Symfony can read the environment variable and boom just works.

Um, it doesn't work by default if you are doing the same with the master branch
because we are talking about production. I don't want you to mess up with the
production data. So by default it doesn't work. You can make it work by setting
a very long environment variable to be sure that that's really what you want to
do. But by default it's disabled.

So let's recap. So you have your web server in production, on Symfony Cloud
with your services. Here, I just have a database. I use a `symfony env:create`
to duplicate my environment so it's going to duplicate all the files and the
infrastructure in a local git branch. Then we open a tunnel which exposes the
`DATABASE_URL` here for a database to PostreSQL, which is a link to the
database for the `some-bug` branch. And then that's read by the `server:start`,
which means that locally I have the web server, but it points to the db from my
`some-bug` environment.

Which means that now I can debug locally with the production data, which is
kind of nice, which is what we did 20 years ago when I started to work, uh,
with the web. Whenever we had a problem, uh, we had to connect to the machine
and mess up with the code directly in production to find a bug and, and fix
that. That's not needed anymore. But the experience is exactly the same. And
from time to time, if fixing the bug takes time, what you can do is you can use
the `symfony env:sync` command, which is going to sync the production
environment data back to your, uh, `some-bug` branch, which, which means that
you will get that also on your local machine. Okay? So you work on the problem,
you can fix the code, you can update a database schema if you want, you can
change some services, so you can add service to fix the bug or add a new
feature, we can upgrade to a new version of the database or whatever you
commit. So when you commit, you're going to commit the infrastructure changes
and also the code and then you deploy. So it's going to deploy on the demo....
uh, the `some-bug` branch. You can use `symfony open:remote` to check that
everything is good. Um, and remember the big difference here is that remotely
we are using the production environment. So it's nice to be able to check that
in the production environment, it works in the exact same way as the local one.
And when you're ready, you can checkout master, you merge your git branch and
then you deploy again. And the nice thing about that is that it's going to be
really fast because we are using the exact same container images from the
`some-bug` environment. So when we deploy to `some-bug` environment, we created
a container with all the information, the code and, and, and, and, and the
configuration and everything, and it reuses this image when you deploy to
production so you can deploy with confidence because you checked that
everything worked well into the `some-bug` environment.

Okay. It also works in a terminal. So if you open a tunnel and then if you say
`symfony php`, and I want to get the `DATABASE_URL` environment variables, you
will get the one from the remote service. If you want to run something with
symfony console, you can say just `symfony console`. It also works with some
commands. So for instance, if you say `symfony run psql`, it uses the binary,
the `psql` binary that you have locally, but behind the scenes, it sets all the
environment variables needed for a PostgreSQL to actually connect directly to
the symfony cloud database.

And that's, what I wanted. I wanted something that is fast, that is convenient.
So I have everything locally, but the remote services. It works on all the
platforms, including Windows. I spent, I spent *so* much time on that. You're
welcome.

And you can test it today. You can go to
[symfony.com/cloud/download](https://symfony.com/cloud/download). So again,
it's not just about Symfony Cloud, we will probably change the URL at some
point. You can download it for all the platforms and you can test the web
server. It's still an early version of that so any feedback is very welcome.
And I think that's all for today. Thank you for coming again. Thank you for
listening and enjoy the conference.
