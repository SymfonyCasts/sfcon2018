# Webpack Encore: Tips, Tricks, Questions & Best Practices

***TIP
SymfonyCon 2018 Presentation by [Ryan Weaver](https://github.com/weaverryan)

With Webpack, your JavaScript & CSS code can have superpowers you've only
dreamed up. And with Symfony's Webpack Encore, you can get all of this with
almost zero setup time!

In this talk, we'll quickly learn the basics of Webpack Encore, then, turn to
the lessons we've learned over the past year: answer popular questions and
explore common problems people run into when moving to Encore. We'll also dive
into a host of lesser-known best practices that you can follow to make sure your
frontend coding is as streamlined as possible. A modern frontend build system:
all in the time of one talk!
***

How many people have seen the SymfonyCasts videos? I was going to explain that
this is really my voice and the miracles of editing. It's really incredible what
we can with that audio. Okay. I am Ryan. Hello nice to meet you. I'm the the lead
of the documentation team. Also, a writer for KnpUniversity... wait SymfonyCasts.

## Hey Ryan!

Changing a domain is harder than you think. I'm basically, if you've been around
Symfony for a while, you probably know me, I'm a Symfony evangelist / fanboy.
The husband of the much more talented and much more popular at the Symfony
conferences, Leanna. How many people have met Leanna? Okay, yeah, okay. That's
so great, because that means, there's many of you have such a great opportunity
today to meet Leanna, give her a jumping high-five, like this style.

Leanna, can you raise your hand, just so everyone sees, right here? Yeah, so
Leanna's right here, and you can find her afterwards. She's going to come up
afterwards and wait for her high-fives. And the father to my much more handsome,
charming son Beckett. So now, I have a kid, so now I get to show you pictures of
my kid in the beginning. Whether you want to or not, so that's my son, Beckett.

## Webpack: Totally New Way or Frontend Dev

Okay, so let's talk about Webpack and Encore. We are going to do just a little
bit of explanation in the beginning, then talk about some lessons that we've
learned over the past year and a half. And also, some new features, because a
new version of Encore came out, only one month ago, and there are actually some
really exciting changes that we've made in that version.

Okay so first of all, Webpack itself. What is Webpack? It's a very simple
concept, it allows you to package all of your JavaScript and all of your CSS
into one file each. And it works via `require` statements, so you can have one
file `app.js`, require some other files, sort of like we used to do in PHP,
require some other files, and that's it. It's just going to turn all of those
into one `app.js`, and one `app.css` file. That's kind of cool.

Or we can explain Webpack as a totally new way of programming. This is actually
what makes it hard. You have to completely unlearn everything you've learned
about doing frontend programming. Because we've spent 15 years learning how to
program in JavaScript with global variables. That's insane.

How many of you regularly use global variables in PHP? There's one brave hand.
No, we don't program that way. How many of you do it in JavaScript? Everyone
that does JavaScript, right? Because it wasn't possible before. You have to
unlearn that. We basically have to retrain ourselves to program in JavaScript
the same way that we program in PHP. Once you do it, it's an amazing experience.

## Encore Basics

So, Encore is just a wrapper around Webpack. It's actually a Webpack
configuration generator. It's a Node library, but we recommend installing it via
a Composer package. I'll explain why later. There is a little bit of PHP in
Encore, but it's mostly Node. Of course, it has a nice recipe when you install
it: `composer require encore`.

Oh, actually see, things changed. That's slightly out of date because we should
not have the `––dev` there anymore. I'll explain why later. There is a small bit
of code that you do want to run at run-time now.

composer require encore, okay, it downloads a few things. This is what it gives
you. It gives you a `package.json`: that's the composer file basically for Node.
Two files: a CSS file and a JavaScript file just to get you started. And that's
basically it.

So, here is what the `package.json` looks like. It just requires two packages:
Encore and the `webpack-notifier` is just cool because it makes cool desktop
notifications when your build finishes. It's like ding, your build is done! It
just makes it more fun. That's it.

The `webpack.config.js` file is where you configure Encore, how you tell Encore
what you need. What you need is actually very simple. You just need to tell it
where you want the output path, where you are going to put the final build
files, and then you are going to point it, it's called an entry, you are going
to point it to a single JavaScript file.

I'm saying go read that `./assets/js/app.js` file. The first key `app`? That could
be anything: that will ultimately control the name of your *output* file. If that
first key, that first argument was `foo`, we would end up with a `foo.js` file.

But we're pointing it at `app.js`. What does app.js look like? Okay, here's a
simple example. We can requires some JavaScript files, or we can even require
CSS files. That's what we have here: we're actually requiring CSS from JavaScript.
I'm going to talk about that in a little bit: that's very, very important. Then we
actually need to install our dependencies. I really should have taken a
screenshot of ... This is basically me running `yarn install`. Yarn is, there are
two package managers in Node. Yarn is one of them. This is basically me running
`yarn install`.

For some reason, yarn lets you be so incredibly lazy that you can just say,
`yarn` and it installs for you. This is me running `yarn install` and the end
result is... boom! We end up with tons of files in a `node_modules/` directory,
which is the `vendor/` directory. At this point, we have basically just required
two packages from Node, and installed them. That's it.

To actually run Encore, you're going to run `yarn encore dev`. There is also a
`yarn encore production` you'll see later: that's the mode that minifies all of
your files and does other optimizations. Of course, there is a `--watch` so you
don't have to run every time. Every time you modify file, this is going to
re-output your files.

The end result of this is that you get two files. Actually, you guys are
probably good at counting, you get four files. The only important ones yet are
the `app.js` and `app.css`. What happened was: we told the Webpack to look at our
`app.js` file. It follows all of the require statements: all the JavaScript we
required, all of the CSS we required, and got all of that JavaScript and CSS,
and it finally writes one `app.js`, and one `app.css` file. Then we just need to
include those in our template, completely like normal.

In some ways, I want this to seem very underwhelming, very boring. Okay, cool,
you pointed at this file, and it outputs these files. Excellent. But this is
massively important. We could stop the presentation right now, because this
gives you the ability to have require statements in your JavaScript! Which means
if you have a file and you want to maybe split it into two pieces, re-factor your
code like we would do in PHP, before, we would maybe not do that. Because if you
split your JavaScript into two files, you have to add another `script` tag, or you
need to go into some build system and remember to say, oh, now concatenate this
file too, make sure they are in the correct order, all that kind of thing. It
just didn't make sense. It was difficult to split your files.

With Webpack, you can actually program correctly. You don't think about the
packaging of your files, you just think about: if I'm in one file, and I'd like
to use another file, I require it. Very much like, again, I don't mean this to
make it sound like the require statement is old, but in PHP, very much like we
used to do in PHP. If you needed something, you required that file instead of
just expecting it to be globally available.

## CSS as a Dependency of your JavaScript

I'm going to put up a couple of best practices guides through using Encore.
The first one, as you've seen, is you need to think of your CSS as a dependency
of your JavaScript app. The same way the, that if you were in a JavaScript file,
and you need another JavaScript file, you add the require statement. That's a
dependency. Do the exact same thing with CSS. You say, in this JavaScript file,
in order for the page to truly work, I need this CSS file. So, you require it.

## One Global Entry

Another best practice is, and this is exactly what we've seen so far, is that
you should have one entry, that means one source JavaScript file that's going to
live in your layout. It's not a rule, but this is a best practice. This is
exactly what we're doing right now, we have an `app.js` file, and that `app.js`
file is included in our `base.html.twig`. It contains all of the global JavaScript
and global CSS that we need for our site. Later, we'll talk about page-specific
JavaScript and CSS.

## Refactoring with require

The game-changing feature is that require statement. The ability to have that require
statement. Which by the way is called, as soon as you start using require statements
in JavaScript, suddenly your JavaScript files are called modules. When you hear
module, it's just basically a JavaScript file that works in this system. This is
just a simple example of how I would maybe re-factor my code. You have the
`app.js` on top, and then I'm just going to require another file and use its
function.

The key thing is, in the second file, is the `module.exports`. In PHP, when
you used require statements, you would just require a file, and just receive
whatever is inside that file. That's not the case with JavaScript, or Node. You
actually need to export something. You need to say, I want to export this
function, or I want to export this object, or something like that. We're
exporting that function, and then we require it to import it.

Notice there is no `.js` on the random word, and that's just because JavaScript
people really like making you be lazy. You can put the `.js` on the filename,
but you don't need to. That's why you see it not there.

## require() versus import()

Or, to make things a little bit more complicated, interesting. You see the
`require` statement a lot, but the `require` statement is not cool anymore. You're
supposed to now use this `import` keyword, and this `export` keyword. The reason
I'm telling you this is so that you know that they are exactly the same. They
are just two syntaxes that do the exact same thing. The `import` and `export` is
actually more of a standard, so that's the one you should use. It has a little
bit more flexibility, but really, they do the same thing. I'll typically use
`import` and `export`. It's all the same thing.

## Module-based Architecture... just like PHP

The best-practice now is to basically program like you would PHP. To organize
your code into modules. In Symfony, we would organize our code into service
classes to be well-organized, you can even unit test them. It's the same thing
in JavaScript now. It's like: okay, I have this function or this object with
some useful functions on it, let's just put that in its own file so that it's
organized better. We can reuse it, and then we can just require it from wherever
we need it.

Here's a slightly more complex example. This is a file that's going to just call
a function, but it's requiring this `displayRandomWord`. Okay, let's look at
`displayRandomWord`. That's just another function. Sorry, that exports a
function, but it imports another file, and that last file is actually another
function which actually gives us the random word. Just an example of the type
of thing we would do in PHP typically: isolate these things into small pieces.

Ah, but there's a problem. We've already talked about this. CSS dependencies.
Actually, let me go back real quick. This works great, but now I'm going to
re-factor my code so that the `displayRandomWord()` function is now going to
output HTML, and that HTML is going to have some classes on it. And dang it, I
need to bring in the CSS file to style those classes. What we could do, because,
remember our `app.js` file requires `app.css`, we could just go into `app.css`
and add our class there. That would make it into the final file.

But we can dobetter than that. A better way is to actually isolate this into its
own CSS file and require it directly from that module. What I'm saying here is:
you'll typically see your main JavaScript file, `app.js`, require a CSS file.
That doesn't mean that okay, great, let's put all of our CSS into that one file.
No, we can actually break things down. We can have some very, very deep module -
you don't even know what page it's going to be used on - but if that one module
needs some CSS, like this, then require the CSS file. Then, no matter who uses this,
you don't even care what page it's going to be used on, the CSS for that module is
going to be included in the final output CSS.

Again, it's programming correctly. That's all it is. You're actually defining all
of your requirements and dependencies in each little file.

## Each Module is an Isolated, Unique Snowflake

The way you need to think about it is that each module, each file, is its own
unique snowflake. Its own unique environment. When you look at a file, guys, I'm
going to keep saying this, it's just like we program in PHP. If you went into it
a file in PHP, you would never make any assumptions about what variables are
available to you, or anything like that. If you need something inside of a class
in PHP, you would make sure that you add a use statement, or use dependency
injection, or something. It's the same thing with these modules. If you need
something in a module, require it. Don't think: oh, it's okay, that CSS has
already been required by this other module, which will be on the same page.
That's bad. Put everything you need inside that one file.

To make things actually even a little more complicated, this is so cool, I love
this, now I've taken that same CSS file, which is required by one of my
JavaScript files, and now I need to point to a font file. Okay, what happens
when we do that? It just works. Because Webpack is going to notice that you're
referring to that font file, it's going to move it into the build directory for
you, and the final output CSS is going to have the correct path to point to
that font file. In fact, to Webpack, this looks like a require statement! It
actually sees this, and it looks, to it, like you're requiring a font file. And
so it says: oh, how do I handle font files? I move them into the build directory
and make sure the path is set. The font file is a dependency of your CSS file.

In your CSS files you can use the `@import` syntax. In the top of some CSS file,
you could say `@import` another CSS file. That's going to be read and parsed in
exactly the same way. You could be in a CSS file and `@import` a SASS file, and
that's going to go get processed just fine, and get processed through SASS.

## Page-Specific JavaScript

Alright, so cool story. But, so far, we've only been talking about one file, one
`app.js` file. More like we have one JavaScript file that we include on our entire
site.

A lot of us still have multiple pages, so you have page=specific JavaScript. You
have your main JavaScript and CSS, but let's say on your checkout page you have
some pretty big JavaScript you don't want to put into your main JavaScript,
because it's only needed on this one page. Okay, so now we're going to create a
second file, `checkout.js` next to `app.js`. Same thing: we'll import the CSS we
need. We'll import any JavaScript we need, write some code. This is just fake
code, but you guys get the idea. Then we're going to go back into our Webpack
config file. Remember, we haven't done much in this file yet. And we're going to
add a second entry. An entry is an important concept in Webpack. Let's see here,
and every entry results in one output JavaScript file and one output CSS file.

The result of this, since we added an entry called checkout, is that we end up
with a `checkout.js` with all the JavaScript we need, and `checkout.css` with all
the CSS we need. Then we just need to make sure that we put that on the checkout
page. Inside of our checkout template, we're going to add a `script` tag for the
one file, and a CSS `link` tag for the one CSS file.

The guidance on this is that, and of course, this is not an absolute rule, but
if you're looking for general guidance, this is how I do it: is to treat each
entry ... Well actually, let me say this a different way. Each page that needs
its own JavaScript is going to get its own entry. It's going to mean, in practice,
that every page is going to have the main entry, the `app.js` file that's in your
layout, and if you need it, one other entry for that page-specific JavaScript.
That entry file should contain all of the JavaScript, and all the CSS needed to
power everything on your checkout page.

The other thing is, that each entry, this is how Webpack wants you to think about
it, is its own standalone application. The Webpack, the authors of Webpack do a
lot of single page apps, so they almost think of every JavaScript file as a
single page app. You want to think of your `checkout.js` as its own app, which
means that you need to, once again, require everything you need. If you need CSS
require it, if you need some external library, require it. It's its own isolated
environment. It's different than before when we think about: I'll include some
script tags, and then these script tags will refer to some stuff that these
script tags did. They're really two isolated environments.

## jQuery: Things get Weird

Alright, so I want to talk a little bit about jQuery in case you're using it.
Because jQuery is actually where things get weird. Actually, jQuery is also,
because things get weird with jQuery, it's also a great example to understand
how Webpack is doing some things internally.

Okay, so you want to use jQuery. This is one of the best things about the new
way of developing. If you want jQuery, you just add it: `yarn add jquery –– dev`.
Just add it: just like we `composer require` stuff. We don't need to download, go
download something. We don't need to find a CDN. You just add it. Then you just
require it or import it. That's it.

This is really great. Outside libraries are just actually very, very easy to use.
This is not the problem with jQuery. This would work just fine. Anytime you import
a module that does not start with a `./`, it's importing it from the
`node_modules` directory. If you are importing a local file, you've noticed
I'm probably using `./` before: that means a file next to me. If there's no `./`,
it's coming from the `node_modules` directory.

jQuery is just like any other module, or React, or Vue. These are just modules:
and you just require them, and they work like normal. Here's a key thing though,
what's the difference between importing `$ from jQuery` and a `script` tag that
you put in your layout for jQuery? The answer is ... everything. Even if that
is literally pointing to the same file! Because, inside jQuery, and this is
common to many, many modules, this is not a jQuery-specific thing, inside jQuery,
it *detects* how it's being used and changes its behavior.

In Webpack, there are no global variables. Okay, just like PHP you can cheat and
make global variables. But if you're programming things correctly, there are no
global variables. When we include a script tag on the page, what does that do?
It gives us a global `jQuery` variable. If you require it, it detects that, and it
uses the `module.exports`: it returns a value. It does *not* actually give us a
global variable. The key thing here is that most JavaScript libraries are going
to change their behavior based on how they're being imported. It's just
something that you need to understand as you're going from the old way to the
new way of doing things.

A few minutes ago I talked about how you want your entry files, your application
and your `checkouts.js` to function like two separate applications. This is
interesting. If we `import $ from 'jquery'` in `app.js`, and its script tag is
included first, and then in `checkout.js` we don't import it, but we just start
using it, is that going to work? No. It's not going to work. Beautifully. That's
going to be an "undefined variable $ in checkout.js": it's an uninitialized
variable. In PHP, we would never expect that to work. It's not going to work.
That is excellent.

This actually is little bit more interesting. Well actually, it's the same
thing. Is this going to work? What I changed here is: okay, can I use the `$` in
maybe a template? I just need, hey, I'm getting lazy, and I need to put a script
tag in my template. Is that going to work? Nope. Same thing, there's no global
variable, so that's not going to work either.

One of the effects of using Webpack is that you will now have no JavaScript code
in your template. Yes, we were always supposed to do that but we got busy, and
lazy, and cheated etc. It's really not possible because you don't have any
global variables. Now yes, you can cheat. In our tutorial on Encore we talk
about that. When we upgraded SymfonyCasts to this, we actually set `jQuery` as a
global variable, a true global variable, so that our legacy JavaScript would
work. So that way, we didn't need to update everything all at once. Yes, you can
work around this as you're updating your application.

## jQuery Plugins: Weird gets Weirder

jQuery plug-ins are actually where things get truly weird. The weird thing about
jQuery plug-ins, so bootstrap, I'm talking about the Bootstrap's JavaScript, it
has some jQuery plug-ins in it, like tooltip.

The weird thing about jQuery plug-ins is that they don't return anything. They
modify jQuery and *add* something to it. They work fine with Webpack, but they're
clearly from a universe that existed before webpack. Because it's just weird.
It's weird to just import bootstrap and suddenly we have a `tooltip()` function.
It doesn't look obvious.

Internally, how does it do that? jQuery plug-ins, again, behave differently
based on their environment. This is very important. If you think about the
normal script tags, if you have a script tag for jQuery, then a script tag for
bootstrap, how does bootstrap know where to find `jQuery` so that it can modify
it? It only has one option, expect it to be a global variable. It just looks for
`jQuery` as a global variable and it modifies it.

Fortunately, most modern libraries, most modern jQuery plug-ins now have code
that looks like this. Where again, they detect their environment and they go:
okay, I'm in a Webpack environment. What do they do? They actually require
jQuery! One of the properties of the module system is that if two files require
the same file, they get the same thing. It's a bit like Symfony's container.
They'll get the same one instance of jQuery. It will actually modify jQuery.

Now, not all plug-ins are written correctly though. Here's a example. This is
just a silly jQuery plug-in I found, tagsinput. The cool thing is I can say
`yarn add`, so I can just add this to my project by saying
`yarn add jquery-tags-input`. Great. I'm going to import the JavaScript file. That,
in theory, will give me a `tagsInput()` function.

Also, this is a really cool thing. It's very common to have a JavaScript library
that also has CSS with it. So normally we have to add one script tag and remember
to go add the link tag. Now I just require the CSS file, and you're done. In
this case, when you have the name of the module then / a path, it's just
literally pointing at `node_modules/that directory/` the path of the CSS file.
Sometimes you have to do a little digging to be like, what's the path to the CSS
file? Once you find the path, you just require it. Done.

This doesn't work. We did the exact same thing as we did a second ago with
bootstrap, but it doesn't work. So you get `jQuery is not defined`. The reason is
that, inside that module, it basically has code that looks like this. It just
goes, jQuery. It just, hey, jQuery. It just expects it to be a global variable.
And it's not anymore.

This is still written the old way. It has no code in it to make itself work in a
module environment. It's broken. This is probably the most common and consistent
question we get about Encore. It's not even an Encore thing, it's just a Webpack
thing. It's important to understand how these jQuery plug-ins work, how they
load, so that you can debug what's going on.

Ok so how do we fix that? Magic. We go back to our webpack config file, and we
add one new line called `.autoProvidejQuery()`. This is incredible. And that
function is an Encore feature, but anything in Encore, we're just leveraging
features in Webpack. This is just leveraging a feature in Webpack. So what does
this do? It rewrites the bad code. It literally, as it's going through all of
the JavaScript files that you are working with, if it ever finds a `jQuery`
variable that has not been initialized, it replaces it with `require('jquery')`
and puts that in the output.

Yeah, right? It's incredible. This fixes it, and all of a sudden you can use these
old things. Brilliant, we fixed somebody else's bad code! Be careful, because now we
can write bad code too. This is now possible, so this is the downside of
`.autoProvidejQuery()`. So, don't turn it on if you don't need it. It now means you
can go into any file and just start saying `$`, or `jQuery`. You can instantly start
programming the old way with undefined variables. And Yay, Encore will also fix
our bad code. If you turn it on, try to not do this, even though you can.

This is important, if I'm in a Twig template, and I put an inline script tag,
that still does not work. Because `.autoProvidejQuery()` does not give you a global
`jQuery` variable. It rewrites any code that's processed by Webpack. If you just
have some random Twig template, there is still no `$` variable. This is just a
repeat from earlier, even if you can cheat with the `autoProvidejQuery()` plug-in,
remember that each file is its own environment, so remember to require it.

## Entry != script Tag (in the old way of thinking)

This is another thing, so a couple of lessons from the past year that we've seen
people do. How many people have had applications that look like this? Yeah,
yeah, me too, yes. Because this is how we used to program. Require shared, and
that creates these five global variables, and these 10 global functions, and
then vendor.

Okay, now we have a `jQuery` global variable. We just build on top of each other by
adding more and more global variables. Then we have some script on the bottom
that actually uses these. I have seen people basically take their old setup and
just move it directly into Webpack. I understand, every file that I have in my
old application is just an entry file, right? Because this is going to basically
result in the same output file. Yeah, first one requires a file, `vendor` is going
to go actually require `juery` and `react`. Okay, good, this works, right? This
does not work at all. Because what happens with that vendor entry, that vendor
entry is going to require `jquery`. Good for you, you just basically gave yourself
a jQuery variable and never used it.

It's like calling a function, setting a variable, and then going to lunch.
Because that doesn't set a global variable. That's why I don't follow this
model, I follow the model of one entry per page. Think about your checkout page
as its own JavaScript application. What is all the code that needs to go into
this to build this JavaScript application?

## addStyleEntry(): It's a hack

Another thing, another feature that we've had forever in Encore is the
`addStyleEntry()`. That's a way for you to say, oh, I want an output file, but
it's just CSS. I just need a CSS file. That's a hack. We added that as a way to make
people more comfortable, but you shouldn't use it. By shouldn't, I mean it's not
like a bug, buggy or something, it's just that if you're using that, you're
probably not organizing your code correctly. Because your CSS just shouldn't be
just some random, hey, I just need a CSS file. You should be thinking what:
JavaScript file is this CSS a dependency of?

In our `app.js`, we require the `app.css` file: this is the CSS that's needed for
our layout, so we think of it as a dependency of the JavaScript file that's in
our layout.

## Splitting Files / New splitEntryChunks()

Okay, now this is where things get really interesting and we're going to talk
about some of the new features of Encore. I realize I'm running a little low on
time, but I also realize there is a break afterwards. So sorry if I keep you for
a few extra minutes.

Alright, so Webpack says:

> hey, it's cool, just require stuff, I'll handle all the details, don't think
> about it. Really, it's like don't think about it. You just write code correctly,
> I'll take care of the rest. I'll build your files perfectly.

Great Webpack, except, wow, those are pretty big files, those JavaScript files,
yeah, 168 kilobytes. This is the development mode, so it gets smaller in production.
But jeez, those are big. Because... we have so many jQueryies. Because we required
jQuery from our app, and we require jQuery from our checkout, so what did webpack
do? It put jQuery in both. Perfect! Except for performance on your front-end.
How do we fix that?

This is not the way we fix it anymore. I wanted to show you this because you'll
see this. This is the way that we fixed this problem for the first year and
half in Encore. Because this is the way that you fixed it in Webpack. Again,
we're always leveraging Webpack features. Webpack 4 came out earlier this year
and changed a lot of things. This is one of the features that is still available
in Encore, but is not the recommended way to do it. Yeah, I'm not even going to
talk about it, but this is a feature that existed before. It still exists, but
don't use it.

Now what? What we do now? I said don't use that feature. Okay, so introducing
Webpack Encore 0.21.0. Wow, what a great version number for a big release! Wow,
congratulations! This brought a lot of new stuff. Webpack 4 support.
I'm going to talk about a few of these. I'll show you a couple of these, so I'm
not going to mention all of them here. `browserslist` is a really good one. That
allows you to put in your `package.json` file which browsers you need to support,
and different parts of Webpack will know how to rewrite your code to be compatible
with exactly those browsers. So, if you're using newer JavaScript features, or
you need some vendor prefixing on some new CSS stuff, then it will automatically
do that for you. Really, really cool. Webpack is always, and Encore has always
done that rewriting, but it was more difficult before to control the exact
browsers that you need for that. Yeah. The rest is not important, or we're going
to talk about.

## The Runtime Chunk

Real quick, this is not that exciting, but I want you to understand what it
does. There's a thing called a runtime chunk. By the way, "chunk" is what Webpack
calls a JavaScript file more or less. When you see chunk, that's kind of what it
means.

This is a function that you can call now in your Webpack config file. Okay,
cool, what does that do? It actually results in one additional file being
output. A `runtime.js`. It now means that you actually need two script tags when
you enable this. Why do we do that? There's two reasons. One is actually a
little bit for performance. I don't want to get into the weeds too much. The
`runtime.js` file contains some code that helps Webpack work. That code tends to
change more often than your code, so by isolating it into its own file, your
other files will change less often, and therefore can be cached longer by your
users. It also has a side effect that when you have the runtime chunk, if you
required jQuery from somewhere in `app.js`, and somewhere in `checkout.js`, they
get the same object. Whereas, if you don't have that, your entries are so isolated
that when you require jQuery, it's actually two separate objects. That may or
may not be important to you.

Oh yeah, I actually have an example of that. Will this work? I require bootstrap
in `app.js`. Down in `checkout.js`, is there a tooltip function? We required
bootstrap further up, right? Right? It depends. With `enableSingleRuntimeChunk()`,
yes, because they're actually using the same jQuery object. With
`disableSingleRuntimeChunk()`, no, they would be more isolated.

It's not... what's wrong... enable is more convenient. Disable is actually a little
more pure, because I said keep them isolated. It's up to you, I just want you
to be aware of the decision there.

## splitEntryChunks()

Okay, cool. Back to optimizing our build. We're not going to do this
`createSharedEntry()` thing anymore. We're going to go back to just, we have 2
entry files, or maybe 10 entry files. Cool. Now we're going to say
`.splitEntryChunks()`. This is all you need to do to optimize your build. You
don't need to think about anything. Just call that function.

Okay, what does that do? That, when you run Encore now, you're going to get a
little bit more output. And, if, you've probably never seen this before, but
notice it says we have an entry point called app, and we have an entry point
called checkout. Then it lists a bunch of files after each one. You'll notice
after app, if you look, there's actually one, two, three, four JavaScript files
listed after it. In order to get the app entry point to function, you now need
four `script` tags. I know, yeah, I just got some looks like hmm?

What happens with `splitEntryChunks()`, is when you enable that, you ask Webpack
to look at all of the files you're requiring across all your entry points,
identify code that you're duplicating between the two, and try to figure out the
most optimum way to split that into multiple JavaScript files.

It's really incredible how it does this. It takes into account your file size. If
you have some shared code, but it's maybe like five kilobytes, it's not worth
the extra web request to isolate that into a separate file. You can configure
all this, so I'm just telling you the defaults. If it's above 30 kilobytes, I
think that's the threshold that it decides that this is better in a separate
file. It also distinguishes between your code and vendor code, things in
`node_modules`, because it assumes that vendor code will change less often, so
it tries to isolate the vendor code into its own file. Because that will change
less often. Your users will, by isolating it, your users will be able to cache that
longer because it's not going to change all the time.

It's actually a pretty incredible algorithm it's using to figure out the optimum
way to do this. It also recognizes that your browser will make, it's either
three or four parallel requests to the same host at the same time. It will split
it to a certain point, but it realizes that if it keeps splitting it, then your
browser's actually going to be waiting to download the first few JavaScript
files before it starts the other JavaScript files. Yeah, it looks kind of crazy
at first, but it actually works really, really well.

Of course, the problem is how do we know what script tags to include? Also,
including four script tags, that seems like a real pain. Also, as you can tell,
these are going to change. If you start doing something different, it's going to
split it in a different way.

So, introducing the newest bundle in the Symfony family, WebpackEncoreBundle.
Amazing! Amazing! Wow. Probably the smallest bundle in all of the Symfony
ecosystem. It gives you two Twig functions and that's it. It's super cool. Now
you just say, `encore_entry_link_tags()`, you give it the name of your entry, and
it's just going to include all of the link tags, because the CSS is also split.
Then same thing with script tags. It will include all of the script tags you
need for that. When they change, will just get output.

How does that work? Magic? No. It's actually really boring. Oh, and same thing:
checkout, include the checkout. It works because Encore dumps a file called
`entrypoints.json` that looks like this. Just dumps exactly which files are needed
for every entry point. That bundle just reads those and it works. This means
that you just enable `.splitEntryChunks()` and you don't care. Things get split
into pieces. It re-optimizes itself as you do different, keep coding and make
more builds. Your code just works.

## 100% Cache Busting / Versioning

Also, I don't talk about here, but if you want versioning on your assets, so
like when you build they have hashes in their filename, you get that out of the
box. In the Encore recipe, we have a line in Webpack config that says
`enableVersioning()`. All of your files output with a hash in the filename. And
you don't even know or care, because those hashes end up in this file. And so
as long as you use the Twig functions, then you get free versioning. If you change
a file, then it's going to change the hash in the filename. You don't even think
about versioning anymore. You just get it for free.

## Long-Term Expiration Caching

That also means if you know a little bit about it, you can enable long-term
expires headers for your assets. You can configure your web server to say, for
all of my JavaScript and all of my CSS files, really, anything in this build
directory, let the user cache that for a full year. Because I don't care. If the
content changes, we'll change the filename. That can really massively speed up
your frontend performance.

## Dynamic / Async Code Splitting

One last thing I want to talk about, this is another feature that's not
technically new with the new version of Encore, but we made it work without
any configuration, is dynamic code splitting. It looks like this. Normally, we
have the `import` function, the `import` call on top, the top of our file, we
`import` all the things we need. Sometimes you have a situation where you have
a large piece of code, and you only need that code under certain situations.

The perfect example is if you have a React or Vue router, and it's like when you
go to this URL, render this component, go to this URL, render that component,
if you go to this URL, render that component. If you just code that, normally,
that means all of the code from all of your components will get packaged into
one single file. Maybe not that big of a deal. It depends on how big your
application is, how much you care. Instead, what you can do is you can use `import`
like a function. It basically looks like an AJAX call, because it is an AJAX call.
Import like a function, and then you give it the name of the module that you want
to import, and then you say `.then()`, that's a promise, `.then()` and you pass
it a callback. The argument passed your callback is that module. And then you use
it.

In this case, since we're importing external linker, that code will not be included
in the `app.js` file. Instead, the moment we need it, it's going to grab it via
AJAX, and then when it finishes, it's going to go execute it. It also includes
the CSS file, because our JavaScript down there required CSS. We don't care. It's
just going to work.

## In Conclusion

All right, so now putting all together, the things I want you guys to take away
from this, the best practices are: create one entry file in your layout that's
going to contain all your global JavaScript and CSS. Treat each entry you have
like an individual application, my checkout application, my product list application.
Organize your code. We can do that now. Finally. Finally we can actually organize
our code properly into pieces. Remember to treat each module like its own
unique environment. Require everything you need just from the very specific
place you need it. Require the CSS you need from that tiny little module that
actually needs it. Like I said, make your CSS a dependency of your application.

Okay, and I forgot this one: require jQuery like anything else, even if you are
allowed to cheat because you use `autoProvidejQuery()`. And, use the new
`splitEntryChunks()`, because that's going to make your life much easier. By the
way, I showed most of the presentation, I showed using normal script tags and
normal link tags. Then we re-factored to the bundle. In reality, you just use the
bundle. When you use Encore, you use those functions, then you don't have to
think about versioning, or `splitEntryChunks()`, or anything. It makes sense to
turn that on.

All right, thank you, guys! Thank you for letting me go a little longer.
Questions? I think I get to throw this. Cool. I was like, there has to be at
least one question, because I haven't been able to throw this yet.

Yeah, that was amazing. That was on me. That was a bad throw. That was a good
job over there.

Why are you wearing no shoes?

Why am I wearing no shoes? Well, now it's like my thing. A long time ago when I
was presenting, I had shoes with heels, and I got on the stage, and it was like
clunk, clunk, clunk, clunk. So, now I wear no shoes so I can be silent and slide
around the stage. If it's summer, I wear no socks also.

Question over there? Or was it the same question?

Okay, my question is I'm not familiar with JavaScript, and I was wondering why
you add `–-dev` everywhere.

Yeah, yeah, that's a really good question. Why when I'm doing the `yarn add`, like
the `composer require`, why was I adding `–-dev`? It doesn't matter. Technically,
it's just a technical detail. Technically, if you use Node on your application
for Webpack, but you also actually had a little bit of Node that you actually
ran in production. I don't know, your code actually went through and used some
Node code, then in theory, by adding Encore and those things as Dev
dependencies, when you're deploying, you would actually, that's, you would run
Encore, and your dev dependencies would be there, and you'd get that final
built `public/build` file. But then when you finally got to production, and you
needed to install your Node dependencies for whatever weird thing you're doing
at runtime, you could actually ask yarn to only install your not dev dependencies,
which would be whatever libraries you're using for your actual runtime code. But
you don't actually need Encore on production. It's technically correct as a dev
dependency because that's not something that you actually need on production to
execute your code. In reality, it doesn't matter because none of us, almost none
of us are going to want to go to production and install our not Dev dependencies
of Node because we have Node running alongside our PHP application. It's a weird
edge case. Yeah, good question.

Nice, oh good, you guys are responsible for throwing it now.

Hi, thank you for presentation. I actually have two questions. First you said
that `require` and `import` are equally the same, but they're not. You showed that
there is asynchronous fetching in `import`. It's a big difference between the
require also model splitting you can include only this part you need. The
second question is how this jQuery fixing tool affects building time.

Interesting. Yeah, first part, yeah. You guys saw, I said require and import
were basically the same, but import is a little bit better, but I didn't really
say why. One of the reasons, as you said, is the import has this asynchronous,
the import function thing. You can actually do that with require also, but the
import is the cooler way to do it. The other thing which we didn't talk about is
import and export, you have the ability to export multiple things from a module.
With the require and the module.exports, it's always module.exports equals, the
one thing that you want to export. But there's technically, with the export
syntax you can export like 10 functions from the same module. Sorry, the other
question was?

About jQuery fixing pack, or making them modules.

Oh yeah, the jQuery fixing thing. I don't think the jQuery fixing thing affects
performance, or at least affects performance very much. The reason is that
Webpack is basically already doing all that work to go through and parse every
file and look for the require stuff. We also go through a tool called Babel,
which actually reads your JavaScript code, and then outputs old JavaScript.

And changes imports to require, yeah.

Yeah, I don't think it affects much. Your builds can get big. It has to do with
the size of your project. If you're doing things like... this alone won't make
your builds slow, but if you were doing, if you were using SASS for CSS, and you
wanted to import bootstrap as SASS, that alone is not going to make your build
slow. But, if you start doing lots of those types of things, all of a sudden
you're saying, hey SASS, go parse this giant SASS file. That's the kind of stuff
that's going to make your build slower.

Thank you.

It's good you guys all sat together with the questions. It's very convenient.

I'm not into JavaScript very much, don't know very much information about it. In
my company, we already moved to Webpack a couple of weeks ago, but I don't know
why every time I get a new branch, without accessing or editing any JavaScript
or CSS files, I can see the `package.json` already changes. I don't know why.

The `package.json` changes?

Yeah.

Yeah, I don't know about that. As he changes branches, the `package.json` file is
suddenly modified. I'm just going to say that in case somebody else knows what
weird thing that happens, but that might be something specific to your project.

Okay.

Oh good, he can tell. Good.

Okay, so basically depends on the environment, so if some of your team, then you
can have some differences in output. You can find a stackoverflow on basically
how to fix it. You can disable it.

You better fix it in your project. So then you have no changes between
operational systems. In our company we have fixed it recently, and it's pretty
easy, you can just find it on stackoverflow.

Yep. All right, think we're past time. If you guys have more questions, come up
and say hi. Thank you, guys.
