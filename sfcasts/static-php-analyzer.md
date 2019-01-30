# How Static PHP Analyzer Changed the way I Look at Code

***TIP
SymfonyCon 2018 Presentation by [Nicole Cordes](https://github.com/IchHabRecht)

Let me introduce you to the world of static PHP code analyzers. I'd like to show
you which tools exists, how to use them and how they help you to improve your code
quality.
***

Yeah, thanks for joining my session, or my talk today. As you can see, it's
about static php code analyzers and what I learned by using them. My name is
Nicole Cordes. I'm a freelancer right now. I started freelancing for PHP
applications and backend stuff this year. I used to work in a Berlin agency and
next year I will worked for Tideways. Maybe some of you know, it's like
profiling php.

Well when I started to use static php code analyzers, I noticed finally after a
few weeks, few months I changed my mind on how I code. And I just want to talk
about that and to maybe to give you some reasons why you, to start using code
analyzers. And before I want to talk about how I changed my, uh, my, my, um, my
expectations of my code, I want to introduce you to php static code analyzers
and the tools I'm using to analyze my php code.

## Tools

And I chose four of the available tools maybe you know, some of them, maybe you
didn't hear about it and maybe you even feel familiar with one of those tools
on your own. So let's start.

## PHPStan

When I, the first time I heard about static php code analyzer was when PHPStan
was released on its GitHub page and this was my personal getting in touch with
PHP code analyzers. As you can see PHPStan is available on GitHub. You can just
install it as a global composer package and finally run it locally on your
machine and start analyzing projects. I prepared some screenshots to show you
how it will look like. In my point of view, it gives you a lot of information
when you run it, but it's strange. The type-safety checks, like does one
function return the same type, the other function expect and can deal with.

So as I told you, I have prepared multiple php code analyzers, and each one in
my point of view has its own strength and it's own way of showing and dealing
with the php code. So when you install it, PHPStan, and run it locally on your
project you just run it on the cli and get some output grouped by class. And as
you can see, you get some information about ??? names, you get some information
about the return type statements. You will get a lot of information, I'm sure
in your project. And um, I chose some of them in my later slides and will not
stick too much to the output, but I just want to show you how the tools will
look like when you're using them in your own project.

## PHPMD

The second one after dealing with PHPStan I found there are different projects
and services dealing with PHP code analyzers. The second one I want to
introduce is the PHPMD the PHP Mess Detector. Maybe some of you already heard
that because this is, I think very long-living project already. Maybe even so
long-living that it's already was forgotten that it's available. But it still
exists and still maintained. And what I like most on the PHP Mess Detector, it
can handle your code structure and measurements, I think in a more better way
than PHPStan can do it, or at least output the information about those
measurements and code analyzations. It's available as composer package and PHAR
file as well. And if you execute it you get some more or less written text, as
well grouped by file. It's not that nice way like PHPStan outputs, but you can
find all necessary and available information in this text and you can even
change the kind of output. PHP Mess Detector can output an HTML page where you
can navigate through open issues and files as well. So depending on what you
need, you can switch the output.

## Code Climate

The third one I want to introduce is some kind of service. It's called Code
Climate and it's subtitle it has itself is: "Automated code review and quality
analytics". As you can see, this is published as a code analyzation as a
service, and it has a nice website. You can connect with your GitHub account
and you can just use all your open source projects for free and couple it to
your GitHub account and GitHub projects. So this is quite nice to even use it
in the existing projects because you just hook in with your GitHub, give some
credentials to the Code Climate application, and you can start right away
analyzing available projects with just one or two clicks.

After you add your existing project, you will get some really nice information.
You can see some nice summary. They use the A to F level to just give you an
overview how maintainable your code currently looks like and what they found
out. They even can analyze the duplicate code. So you will get some
notification if you have multiple files using the same code and you can just
refactor your code and exclude duplicate code into own classes and object. So
this is, I think this is a more nicely way to get those information then on the
cli. So I'm more or less using the services that I'm introducing to you right
now.

And just to give you an overview, how you get, how the results are more
structured, presented. You can even see the codebase where the information is
available. So you can not just see some lines of php files and some information
about your code structure, but you can even see the code and maybe compare the
information or the remark and the code itself. So it get, you get a better
overview what you have to do to resolve the remarks. And you are able to even
group depending on some criteria so you can just say, okay, I want to care
about naming things and want to ignore all the complexity stuff like that. So I
think it's worth to have a look at this.

## Sonar

And to be honest, my preferred one is the SonarCloud. Maybe you even heard of
SonarQube before. SonarCloud is the SonarQube as a service, it's free for open
source as well and it's even, now it's not that easy to run it because you need
some preparation in your repository, but you connect with your GitHub account
as well. And finally you're ready to start. You have to add some configuration
file and some execution in your project. Currently on my project I run some
building process or testing process on TravisCI. I think TravisCI or GitlabCI
is known by everybody in this room. And one step in the CI configuration is to
send... to generate the information and to send it to SonarCloud.

And SonarCloud itself offers like Code Climate, a nice overview in some website
style of course, it's a web application. And you have the information about
bugs so that trying really to analyze php on multiple levels of php code you
have bugs, you have even vulnerabilities. So they have kind of security checks
as well. You have those code smells. It's the things I want to talk about
today. And um, you have some section even for PHPUnit code coverage. So you can
even include your php code coverage reports and the SonarQube and SonarCloud
information and see everything for one project on just one page.

Just to give you some other expressions, we can then open the issues so you get
more or less a list of issues found for the project. And um, well you have
those, you have those grouping and check marking options and sorting filters as
well as on the other page. And when you get one step deeper, uh, you even see
more, more lines of code of your project for each remarks. So it's, really
underlying the problems and you can get some more information about the
remarks. They have a huge database where they all describe what, what remarks
they are offering to your code and this is really a nice way to be informed
and, uh, get some information and impressions how static code is analyzed.

## Measurements: Naming

And because I just used some, some newly introductory words, I want to
introduce you to the world of static code analyzation. First of all, we have
the topic of measurements. Measurements means it's about numbers more or less.
So we have a remark, so information about our variable names. We can, we can
execute checks: are the names too short? Are the names too long? Do they stick
to lower camel-case names? Other services analyze method names. So they do they
have a proper length? To methods names which are too short, should maybe be
extended to be more readable. There are some tools even can check the uses of
the constructor method instead of the class name and they can even go a step
further and check the return type of your method and offer some suggestion to
rename the method from `getFoo()` to `isFoo()` or `hasBar()` if your method
called `getFoo()` returns a boolean. So there are a couple of information you
get about just naming things.

## Measurements: Lengths

And as I said, it's all about numbers. Those tools are, also measure your
method length. So how many lines of code does your method have? How many lines
of code does your class have? How many parameters does your method have? And
you can, you can configure your personal values or you can configure your
personal likings and say, okay, I want to stick to parameters list of three
parameters because if I need more parameters, maybe the method that's, that's
too much and I need to refactor my code and group maybe parameters into other
objects and stuff like that. It even has some measurements about field count
and public field count of public methods. So like saying your class has a lot
of public method, you can just configure, okay on my classes should have five
public methods, otherwise they doing stuff too complex and I need to rethink or
restructure my objects and class information.

## Code: Structure

Then they are analyzers concerning your code, your php code. So one of the
analyzation is: do you have a lot of commented code? Because I think all of us
use some kind of versioning system. We can just simply remove code, there's no
need to comment code out. And, personally I would say there's no need to have
commented out code for, maybe "I need it later". So just remove that. You can
restore code according to your local history in your versioning system.

They analyze how many return statements does a method have? Because when you
have a lot of return statements throughout your, um, throughout your method,
maybe it's not that practical and it gets too complicated to read your method
and to understand where does which return come from. And so my recommendation
would be return early but only once. So more or less all my methods have an
early return. But afterwards I just have one final return and storing the value
in, some variables. Or even I don't need some variables if the return can be
passed from conditions. So I can just, I don't need to use extra variables to
store the return value and then have the last statement return value. But I can
just say: okay, return if empty or if this and that is ????. And if you return
early, I consider or I would recommend to prevent superfluous else parts. I
think we all have written code where we have, if some condition is fulfilled,
return true and then I'm, I used to, I used to stick to else, return false. But
if you have an early return, you don't need the else part, you can just say,
okay, if I'm not inside this condition that returns true, I just want to return
false. There's no need for the else part.

And um, when you're dealing with return type statements, and this is, maybe
you, you remember I said the PHPStan, is a really good tool to deal with return
type statement and dealing with: does the return type match the expectation.
When you have too many return type possibilities, those tools cannot deal with
the proper, with the proper type-hinting and say, okay, I'm not sure what this
value here is and maybe it's not valid and they give you a wrong, false
positives. So stick, or try at least stick to one return type. And my personal
recommendation would be try even to prevent nullable return types. So really
use empty arrays use empty objects, stuff like that just to have one return
type in one method.

## Code: Structure II

As I said already, they, the tools are useful to give you numbers of how many
methods you are working. If you have the recommendation or if you have the
information that's that one of your class uses too many methods, you maybe
should think about multiple objects or you maybe should think about refactor
the object and split task into two separate objects just to reduce the numbers
of methods in one class. One other recommendation from the analyzers is: throw
dedicated exceptions that can be caught outside the code itself. Don't throw,
don't throw RuntimeExceptions, but throw `ConfigurationIsNotSetException` or
`ValueIsNotExpectedException` that can be directly caught by some other code
around those methods.

And for multiple reasons the recommendation of those static php code analyzers
is avoid static calls. Just try to use dependency object that have own method
you can call and own return statements that you can deal with. This makes
working with your code a lot easier when you have to deal with testing. So
static calls in the manner of you want to test your code, you don't want to
have that. You don't want to have static calls in your code.

## Complexity

And my favorite topics of all this php code analyzers is a whole topic about
complexity of your code. So complexity of your code, it's about how hard is
your code readable or how hard is it to understand your code. If you have a lot
of structure controls inside one method - so like if else, foreach, switch and
you have really huge kind of levels inside a method - your method is not
understandable anymore and you have too many branches and too many conditions.
So this is really, really good information from the static tool analyzers. I
got, in my first weeks using them, to learn that they completely changed the
way I finally looked at code because, as I said, each control is considered as
one logical point: they have a number that increased per control structure. And
even if you have, for example, an if statement and using some else or or inside
one if statement, even those logical operators increase the count of
complexity. And even ternary operators and null coalesce operators account as
own increasing point for the, for the complexity.

So if you have a really long method with huge conditions and use logic
controls, it's hard to understand them. And I'm sure nobody then nobody else
and you will understand the code properly. And so the recommendation from my
point of view is merge nested if statements like you don't need to open if
condition one, then if condition two then if condition three, but you can just
group them into one if statement. So you can merge the if conditions. And use
helper functions to resolve those conditions in the first time. So it's more
readable to have three proper name method for your conditions. So if: is
condition 1 met or is condition 2 met or is condition 3 met, it's much more
readable than having those three conditions on one line and not having a clue:
okay, what is this test about? What is this condition about? So we factor
conditions in own helper method and named them properly. This really helps you
a lot to understand what's going on in your own code.

## My Conclusions

So after I introduced you on some magic php code conclusions, I want to give
you my conclusion, how my code changed via using the PHP code analyzation and
what I try to do now for my code and what I maybe want you to ask or you offer
to test on your own. Like I already said, I pay more attention in naming
things. I know the, one of the main issues in development is naming things. And
when not common with naming things and we have still used problems with naming
things. So it forced me to pay more attention in how do I name my variables.
And even more attention in how do I name my methods to make it more readable
and to make it more speakable so that even other developers who use my code -
or you have to work with my code - understand what this method is about,
without even having to look into the method, but just by reading the name.

As I said I already refactor conditions and own methods just to proper combine
them into one if statement and have all the single conditions and own readable
methods and even my methods just return this simply line of code. But because
I'm able to read the method name, I know what this check is about and I know
why I'm checking this and this explicit condition. So, there's this one thing I
try to do each time and as I said, I really try to prevent multiple return
types because I think and, and those, those, those two paths go hand-in-hand
because I can combine multiple conditions, I can just have one early return and
then go on with my proper code of application controller logics and have just
one, one line with some conditions that can catch up the preconditions of this
method, if the preconditions are not met I can simply return. And afterwards I
can just go on and I'm, I can be sure all my preconditions are met because I
named the preconditions and already checked it. And well, as I already told
you, I really try to use and force myself to only use one return type per
method and to even to prevent return nullable return types because this makes
dealing with, with the return values a lot of easier even in the, um, in, in
the parent code above, uh, outside my method. So I, I don't have to deal with
multiple values anymore. I can just say, okay, if this exists or if this
property has some kind of value, I can go on. Otherwise I don't care, I don't
have to care about that.

So maybe what you think the conclusion is: I really start thinking about code
before I writing it and I think this is the most important point in my point of
view. I'm not just writing code. I think about how to deal with the code. I
think about how, how I can better maintain my code even in the next few days or
few weeks or few month. And I think it's even easier for others to deal with
the code I'm writing. And the benefits of thinking about how to write code is I
think my classes and concepts for applications got a lot of better because I
just care about naming thing. I just care about concepts because I want to
group them in the proper task and proper the proper groups of methods. And as I
already told you, I think my code is more readable because I have more speaking
code. I don't need to get into method just to understand what they're doing.
And my code gets a lot of testable. So fighting tests, unit tests, integration
tests gets a lot easier because I have all those methods available and can
really test everything on its own and I can decouple my objects from, from the
testing point of view and can offer some mocking objects there because I use
dependency injection and stuff like that. So even testing gets a lot easier
with the proper code.

## Warning

After I give you so many information about why you should change the way you
were working with code and why you should start using php static analyzer, I
just want to give you some words of - I call it warnings. So when, when I
started working with a php code analyzers, my initial goal was to resolve
everything. So every remark has some valid point of view and I need to change
my code to just resolve every remarks from each and every tool. And, I, I
really would ask you, or what I noticed after I did months with static code
analyzers is: stop over-engineering. You don't need to resolve any problem or
any remark given by this tools. I noticed not all remarks are really valid
because you may be depending on some kind of frameworks and this is your code
is exactly that way the framework expected to be then you don't have any chance
to resolve those problems coming from the code analyzers. Or depending on your
time and money in the project, that's maybe not that possible to work on the
project anymore or to have the time to make a clean up after you've written the
code.

But I really would like to ask you if you see, if you see the output of the php
static code analyzer and you don't have the time or you don't have the money to
clean up your code, just use those information for your new code. So, I can
totally understand that there are projects and, where you don't have maybe the
effort, maybe the time, maybe the money, I don't know. There are multiple
reasons why you don't work on existing code anymore. But knowing those issues
and not learning from them, I think it's the most worst case you can have So if
you get the information that some older code has problems or some older code
should be refactored, use those information for your new code to prevent those,
uh, those, uh, uh, errors, those problems you had before.

And, well, as I said, even sometimes in my code analyzations of my open source
projects, I noticed there are really wrong false positive information. So, the
tool has given me some kind of information, it's about complexity or it's about
naming things. And I think, okay, no this is not that valid. For example, one
of the remarks from the SonarQube code analyzer is to store every string value
that you use multiple times within one class in constants, to refactor the code
and use constants instead of the string values. And this is something I cannot
agree with. So this is not a valuable recommendation. And, I, I'm more or less
excluded this checks from my project because I think this is some information I
don't want to have each time because this is something I won't change in my
code.

And as I said, all the knowledge you gained from this PHP code analyzation,
even if you change existing code or not, the knowledge, use it for upcoming
code to write really clean code. Think about it. Think about how others may see
this code and how other would understand, or if others would understand your
code. And when you're working on open source projects or have public available
projects on your own, just think about contribution, make it more easier to
contribute, make your code more understandable and start really using php code
analyzers.

So I hope you get some impression or you get some useful information about code
analyzing. Thanks for joining. And maybe one or two have some questions or I
even prepared some output. So if you want to get into some output information,
we can have a look in the real life example, if you're interested.
