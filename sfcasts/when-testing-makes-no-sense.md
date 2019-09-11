# When Testing Makes No Sense

***TIP
SymfonyCon 2018 Presentation by [Miro Svrtan](https://github.com/msvrtan)

If you look at the stage of any conference in the PHP world, people are
preaching testing, testing, testing ... If you on the other hand look at the
community, the percentage of people writing tests is really low.

As a person who went from 'How can I ask for more time/money/resources for
testing?' through 'ask for forgiveness instead of permission', to person who
writes tests a lot, I still believe testing doesn't make sense. No, it doesn't
make sense for all and everyone, often enough it makes no sense for me too.

This talk will explore that fuzzy line when you have to shift your mind from one
side to the other: in both directions.
***

Everybody's got their caffeine? I know it's the last session of the first day,
so, everybody needs it, right? No? Somebody can live without coffee. I need some
help with how to do that.

## The Back Story

So, first off, I see a lot of young faces in the crowd, so, do you know what
this thing is? That's how old when I started developing professionally. So,
almost 20 years now. So, I started off doing small websites because at that time
people just wanted to have some internet presence. And far away from any kind of
rocket science, even by those days, and for most of you today, it would be like,
yeah, really what's the problem there? But that was the time that we didn't have
cool things like frameworks, templating engines, database abstraction layers or
ORMs. We had to handle all of that ourselves, but this was actually the fun part
of the job. The problems started when you had to make it work for things like
Netscape or Internet explorer 5.5. And if any of you are doing front-end these
days and think you're screwed, trust me.

And, I got so uninterested in development that I actually went back to
university doing some freelance on the side, earning some money. But about 10
years ago I found a team, I found a company, I found a really great job. It was
a small product. It started off, let's say, like a dog house - really, really
small. As we kept working on it, it kept growing in both features, customers,
money. So, I was good with, let's say, carpentry.

So, in few months we had a shed, which meant more and more people could fit in.
They wanted more and more, more money was coming in, more customers, more
content, more everything. So it's grew again. You understand that you can't use
wood to build skyscrapers, right? You have to change your base. So yes, I had to
learn how to use concrete to get a big, better stuff done.

But what actually, my code looked like, something like this. You know, and to be
honest, probably, in this case, this was done by somebody and then by somebody
else. And for most of it was us, us. And, okay, it was ugly, but it worked. But
what our customers wanted was bigger and bigger and nicer and not now concrete.
It's aluminum, it's glass, it's shiny.

But as we kept growing, we kept failing. Because, hey, I didn't have the
experience needed for that project. I used to work in small websites, small
apps, not 100 million user visits a month. And what customers actually paid and
what I thought was, oh, this is what we're doing. But in honesty, most of the
developers don't understand that businesses want a nice feature, another nice
feature, they really really want something to just glue them together. By
design, because businesses don't know always what they want.

So, this project has took about five years of my life. And the next project that
I came on, yeah, it's going to grow like that, right? It's going to be that
small product is going to be the biggest thing ever. Unfortunately, what I was
suffering from was Stockholm Syndrome. Because in reality, not every project or
product becomes big.

So, let's say that you want to buy a dog house, like a real dog house. You have
a dog and you want to buy a dog house. How would you feel if I came to you and
told you that the price for this nice small dog house includes something like
this? Because it's going to be a skyscraper so I want to do it before and avoid
all of the problems. Right? Found yourself any time in that situation? So,
Donald Knuth said: Premature optimization is the root of all evil. And, that's
why I want to talk to you about when testing makes no sense.

My name is Miro Svrtan. You can find me on Twitter using this handle. So why am
I talking about buildings and testing? What's the connection? So, who here is
testing? It's an easy question, right? Okay. Who here isn't testing? Again, so,
this was a partially loaded question. Sorry for that. But actually all of us
test. You write some code, you open your browser, you check that it works - it's
also a testing. So, I hope nobody just writes the code and pushes it to
production. To be honest, I did it few times. But not as a normal way of
working.

So, let's say that we have a small website. No Mars rover thing, just a small
website. And we have a bug. How much time do you think after somebody reports it
will take us to fix it? Five minutes? An hour? Half a day? How much time does it
take you to deploy? Probably 5 to 10, maybe 15 minutes. Right? So, why would we
test and invest in testing when we can fix something quickly and deploy it
everywhere?

But the problem was that about 7-8 years ago, a lot of web developers became
phone developers. Been there, done that. A few of my colleagues went that way,
but they didn't have the mindset of testing. Which is pretty nice for Google
store because it takes about 3 days after you fix it for people to get it. But
what if it's an iPhone? iOS app? 6-7 years ago it would take from 2 weeks to 3
weeks, sometimes even a month to fix something.

What if you're doing desktop applications? You have a bug and now all of your
customers have to be somehow contacted to download the new version because,
okay, maybe Mac users do have App store, but Windows users not so much. How hard
does it then to fix a bug? It might not be a critical bug, but it might be
something that bothers your customers, your users.

Hey, we have hardware. But that's at least easy, right? Because, this is a
router, so it's connected to internet, right? So it's easy to fix it. If we have
a bug, we can just ship it to all the routers. What if the bug is in the piece
of code getting the update? Are you going to send all of your staff to the whole
continent, to the whole world to fix people's routers?

## When should we write tests?

What if it's... what if it's a washing machine? Hopefully, it's not connected to
internet, but you find out that you shipped some of it and you can't wash cotton
on 60 degrees. So, you now have to call all of your mechanics, all of your
service departments, to contact the customers to flash the washing machine. So,
when should we write tests? When it's a high cost to fix? Because, maybe,
sometimes it's cheaper to test than to try to fix it once it happens. Maybe when
the cost is larger.

Let's go back to the website. So, should we ever test websites? We can fix it
easily, right? Yep. Right? Anybody? Nobody? Everything test? But banks do use
websites as well. I mean I hope you don't experience a bug where you send the
money instead of receiving it. The other way around, probably, you'd like it.
Right?

So, let's maybe look at it as when it brings value. Because value is not just
the money, it's how much your consumers love the product, how much your clients
are happy with your work. Maybe for you, if you're a development company, maybe
because of, maybe, it would bring value in the sense of people not leaving the
company often because they would be more happy? There's a lot of cost when a
disgruntled employee decides to leave because then you have to get somebody in.
It might take about 6 months to get the new person in. So, let's define the
value as benefit minus the cost.

So, what I'm trying to say that if, let's say, you have a tomato farm that
produces a lot of tomatoes, you probably shouldn't use the same tactic - you
might use on your own tomato garden. You maybe like your tomatoes a lot, but
would you, um, would you create a whole lab? Would you test it every day?
Probably for tomato farm you would because if one goes down - you might lose
everything. Everything, if you get the disease. In case of your tomatoes in the
garden, you can just go to a farmer's market or a supermarket and get some
others. Okay?

So, value is benefit minus the cost. Of course, I do understand that some people
might invest a lot of money into their tomato gardens because it's a hobby and
we tend to spend a lot of money on our hobbies, but let's look at it as a
business. The benefit minus the cost. So there is some, before I go further,
there are lots of different things about testing. So what I'm going to try to
focus on is writing test, automating the tests, the ones that help you ensure
that the app does what was expected? Okay?

So, who here writes tests in that way? Okay. Who tests everything? Okay. 6 to 7
hands. So, actually, can you raise your hands up? Who tests everything? How much
security testing do we do? Performance testing? Visual regression testing? That
all is testing, right?

So, why I decided to do this talk was people came to a stage like this and said
we test everything. And a lot of people understood that, yeah, to be a good
developer you have to test everything. And they started testing, but they
started testing the wrong things. I've done it that way. And, 6 months ago I did
this talk in Amsterdam and I spoke to another speaker afterwards. He wasn't at
my talk, but he was interested in what I was talking about and he told me: Hey,
yeah, well no, you shouldn't be talking about when not to test because we test
everything. I said: Okay, so performance? Oh, no. Security? Some. Visual
regression? Nah, we actually TDD everything. Wait, you're using Symfony and you
TDD everything? Yes. I was really interested because I found TDD to be very
wrong for Symfony, the Symfony part, like using the Forms, using the Doctrine
and stuff like that. And I was wondering, okay, how do you do that? Oh, we
don't. So, wait, you just told me you test everything and you do it TDD way, but
you don't do the UI, the infrastructure and stuff like that? Nope, that's the
UI. So if I wasn't really interested in that part, like how do you do that? I
would think, yeah, I'm the stupid one because I don't test everything. Please,
don't be.

## Fallacy #1: It makes my project more expensive

There are a lot of fallacies with testing. 6 or 7 years ago, when I wanted to
start writing tests, one of the biggest problems I had, I was a team lead at the
time, was... So similar feature was about 100 hours. Now I need to set an
estimate of about 150 hours because, well, writing the code, writing the tests,
right? So it's gonna take more money from the clients. They're not going to be
happy. What I didn't think of - do you charge for manual testing? So, do we say:
Hey, it's gonna take me 7 hours to write the code and X amount of hours to run
it in the browser as I click save on the code every time. We think of it as the
same thing, it's part of development.

And, if you have to explain to anybody, try to explain to them the bugs cost
too. So most of the companies should have, should solve the bugs for free.
Right? I mean I know of companies whose whole business plan is to charge for
bugs, for bug fixing. But you might notice that that's a pretty bad business
model because somehow they want to produce more bugs because that means more
work for them. Right?

Anybody here has a car? A new car maybe? Or somebody's buying a new car?
Recently? So, let's say you want to buy this car. You go to, I think it's an
Opel dealership, and you say: Hey, I want this one. How much does it cost? Oh,
20,000 euros, let's say, I have no idea, please, don't get me on the numbers.
Okay. So, it's tested, right? Yeah. So can I get the version that's not? It's
going to be cheaper, right? Because why would I pay for testing?

So products don't come out without those things. I mean, when you buy food, you
expect it to be tested, right? You don't want to be poisoned. Try to ask for
forgiveness, not permission. Just try to do it. Try to do it in a small way.
Start building up your tests. Every new feature, add more tests. You don't have
to do everything the first time.

## Fallacy #2: It takes a lot of time

Another fallacy is that it takes a lot of time. That might be true for a first
timer. Because there is a cost of learning. I mean, I guess, most of you that
had a car that are driving had to go to school for driving, right? It took like
20 to 40 hours. Right? So, you don't take that into the cost of every time you
go to the shop. Right? It's learning. Yes when your first project, it's going to
cost you, your company, your product team, whatever. But it's learning. You wish
don't learn.

## Fallacy #3: It takes extra time

It takes extra time writing the code, writing the test. Right? So, if I rock, if
I write 10 lines of code, I need to write some lines of tests as well. So, that
takes time, right? Manual testing takes time as well because you have to Alt+Tab
or Ctrl+Tab something, check your browser. So, if you write your tests as you're
writing your code, you don't have to do that manual check. You can just run the
test and it will tell you that, what you expected, works.

## Fallacy #4: Ship PoC to production

So, for some years I thought that if I implement something that's ready for
production, let's say, you want to implement a new social login or a new payment
gateway and you get one PHP file where you see that it works, then that's it,
right?

### Phase 1: Exploration

But actual feature lifecycle is the part where you explore. You look how that
system works. It could be a library, it could be a new database, it can be a
third party integration, it could be whatever. That is the part where you don't
test. You should explore, you should enjoy, you should understand what you're
doing.

### Phase 2: Modelling/architecture

Then you start modeling. Again, we're not testing, you're using tests but you're
not testing - you're modeling. And this is a very confusing thing, when I try to
explain it to people. If I'm writing tests, right? How is that not testing? So,
if you are in a car, it doesn't mean that you are the driver. Okay? Uber and
Taxi doesn't work that way. Okay? So, yes you could be using tests just like you
use a car, but that doesn't mean you're driving it.

### Phase 3: Development

The third part is development and that's where testing needs to be done. A lot
of it. Because you're not sure about all of the edge cases that could happen.
Here tests have a great value.

### Phase 4: Production

And usually, when I go to production, I delete some of the tests. Because when I
was writing them in the previous phase, I wasn't sure which of the tests will
have value later and I don't want my tests to run for hours just because during
the development phase I wasn't sure if that can break or not. I'm not saying
delete all of the tests, but be... feel free to delete some of those that you do
not need.

## Fallacy #5: 100% code coverage

100% code coverage is something that I had a lot of problems with. And I hear
from the others. If I don't check the code coverage, how would I know if
something is tested or not? So you want to go and have 100 percent because in
that case you know your app is tested. So, what usually people start is, well,
let's get the code coverage as high as we can, as cheap as we can, which means
testing getters and setters. It can be fun for like one or two entities,
five-ish is Ballmer's peak. Oh, need of Ballmer's peak. About 10 - I'm happy if
everybody doesn't quit their company.

So, should we test something like this? I really don't see the value. When I
moved to using commands and events, command and handler events, command handler
patterns, I found that I spent a lot of time on testing those setters and
getters, once again. What now I usually do is if I do use code coverage, I just
put all of the commands and event namespaces into ignore. And I just don't care.
I want to achieve the higher percentage, but I don't want to know about those
DTO classes.

## Fallacy #6: We have "tests"

We have tests. I've heard some people complaining a few years ago about they
wanted to participate in open source project. And they implemented the changes
because implementing those changes was rather easy, but making all of the tests
work was the hard part. You don't want you and your fellow developers to be in
the position that, to change the code, is so hard because of the tests. Because
people in that case will not test. I mean if you have a car on the bottom of a
lake, what's the value of it?

## Fallacy #7: Writing tests later is OK

I mean, and that most often comes from: Let's write tests later, which is most
often coming from management. And as a disgruntled developer, I might come, I
might build the whole feature. I know everything works and before merging it, my
manager comes and says: where are the tests? We're not merging that without
tests? No. Everything works. I don't care. Everything works. No. You have to
write those tests.

What do you think? What kind of tests am I going to write? I'm just going to
look. I'm going to split my screen. I'm just going to look at the code on the
left side and replicate it on the right side. Because that's going to get shit
done. So, yea, I'm just gonna add a simple case for this, maybe.

## Fallacy #8: TDD has something to do with testing

Who here knows what TDD is? Anybody practicing it? Okay. If I say, if I say test
driven development - is that TDD? No, sorry, another loaded question. Test
driven design. Not development, design. You're supposed to use TDD to build your
architecture, to model it. You can write your tests first and then your code -
that's not TDD. TDD is that small red, green, refactor, red, green, refactor,
red, green, refactor.

Anybody's remembering this tweet? Anybody's remembering the shit storm coming
after it? Where everybody came with: Haha, you see, testing makes no sense. You
shouldn't test anytime. Why were you forcing me to test? People don't understand
the TDD and testing are completely different things.

Uh, do you all know who DHH is? So, he's an author of Ruby on Rails framework.
He works for a small company called Basecamp, used to be called 37 signals and
they have a few small products, really, really small products, really cool
products, but really, really small products, that they have been building for
about 10 or 15 years. What kind of modeling would they need when they know what
they're doing? There's no modeling in it. They just need to test that what they
want works. It's not really modeling. And, just to try to explain to you, that's
one of the rare companies that would come to their clients, when they would ask
for features, they told them: No. We have our product, we love it, as simple as
it is. If you want more features - use Jira. Use this, use that. Thank you.
Here's the door. So, don't forget the environment that this comes from. They
don't build that many features.

## Fallacy #9: TDD all the way

If you ever looked into TDD, one of the things that even the people who invented
it said, you have to do everything the TDD way. Only in the last few years they
started saying: Well, we were wrong. Because trying to build everything TDD way
takes a lot of time. And if you're yet another payment gateway, yet another CMS,
yet another E-commerce and you know how everything has to look, what part of
modeling do you have to do? It's yet another house. Maybe has another color, but
that's about it.

I'm going to guess that some of you changed the company a few times. Have you
ever come... have you ever changed it to company that has good tests? Whoa.
Really? Wow. Three hands. Wow. So, my previous employer had such a great
coverage that I started shipping the next day. I came on Monday and started
working on Tuesday. Because it was easy to onboard me because I could have gone
through the tests to understand what and why are things happening and even if I
screw it up, because it's my second day, people, the CI says: Hey, this doesn't
work anymore. It's much easier to add new features because you know that you
didn't break older ones. You know that the older ones still work.

So, what do we had? This, the previous exam? We know this suck. We have to add a
feature now. The feature is that if you're not from, um, from that European
Union country, I'm using Portugal since we're here, you shouldn't charge a VAT
if it's a company. So, maybe this is the point where you should test. Switching
from rather simple if, to another.

What if you have to refactor something? So you have all of your tests. They run,
you just change the code and all of your tests say: Hey, it still works. Still
works. It's okay, everything's good.

So let's make this bit more readable to avoid nested ifs. So this is what
happened. This is what I have. Can anybody spot the bug? Where? Here? Nope. You
always have to charge VAT for that country if you're selling something in your
country. Any others?

So, I just wasted two minutes of your life trying to understand if this was okay
or not. So if we had tests for this, you don't have to look at this. You know it
works. It's easier to rewrite things. And running tests means that we can do
some fun while they're running. I mean at least for us in PHP world because we
don't have compile times, which I'm something that I'm actually missing. But
that's a long story.

Cleaner code. Trust me, it's really hard to test shitty code. If you write your
tests as you're writing your code, you'll will write cleaner code. Because for a
15 line code, 15 lines of code, you might have 300 lines of test. And then you
add an if, which means 600.

But maybe if you're building a small app, something that you have built often
and that you understand. I don't think that testing brings any value. Let's say
you are bringing, you're building a one-off app. Again, you can easily manually
test it. That's okay. You know it works, but what happens a lot is, that simple
app becomes a bigger one and a bigger one and a bigger one. When that one-off
app for one concert in Lisbon becomes an app for all the concerts of that group
in the world or all the concerts in the world for every group, you can't use the
same thinking. When you're exploring something, if you're trying to learn a new
language, a new library, a new framework, a new something - don't waste time
testing. Enjoy it, learn how to use it.

Last but not least: job security. If you're a key person in your company and you
have no tests, you cannot be fired. You can ask for whatever you want - you'll
get it. I want a new car, I want the best Mac, I want 5 month vacation - you'll
get it. What people keep forgetting is that at some point they might look for
another job because they're not happy where they are or they have to move to
another city or they get the shitty manager because their previous one decided
to go somewhere else. You will not find a senior position that says that doesn't
have listed testing experience, testing knowledge. I didn't trust people when
they started telling me that 6-7 years ago, but for last 3 years, no senior
position ad that I saw, came without testing experience. So, it's gonna be
really, really hard to move to find a new job and it's just getting worse and
worse and worse. So if you're like, my grandma had a few chickens. No fancy lab
to test them. Please, don't think that you can use the same principles to have a
chicken farm. It's different mindset. Thank you.
