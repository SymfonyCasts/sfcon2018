# The Patterns Behind Doctrine

***TIP
SymfonyCon 2018 Presentation by [Denis Brumann](https://github.com/dbrumann)

How does Doctrine talk to your database? What are Data Mapper, Unit Of Work, and
Identity Map? These are the questions I want to answer in this talk. We will
look at how Doctrine ORM implements them and what they are there for. Finally we
will look at how they compare to Active Record and what the benefits and
drawbacks are to help you choose which one fits your needs best.
***

A lot of things, um, I hope it's not too fast. But in any case I will still be
around afterwards, so if you want to talk about any of those topics I will be
talking about, we can talk about that later on in more detail. So first of all,
because I already know that in this room it's really hard to see the slides if
you're in the back of the room. The qr code goes to joined.in to the talk. I
already uploaded the slides with some additional pages in there that you just
have to skip over. And yeah, you can just go to the talk/c1b8c is the talk id
and, and you can just get the slides and you watch them on speaker deck as well
if you don't want to follow on the screen or things are hard to read.

## Hey Denis!

So, um, with that out of the way. Um, hello again, my name is Denis Brumann. I
work for SensioLabs in Germany as a software developer in Berlin. And you can
reach me via email denis.brumann [at] sensiolabs.de and also on twitter and
GitHub with my initial and last name.

## Disclaimer: Doctrine 2 Vs Doctrine 3

And before I start I want to make clear that everything I will talk about is
about Doctrine 2 not Doctrine 3. It's not out yet, but it's already under active
development. The master branch already points to Doctrine 3. Things might still
apply in some way or another, but, but it will probably not be the same. For
example, the unit of work is probably not stick around. Um, so keep in mind this
is how things are right now, but it's still helpful to know because if you want
to do this switch to Doctrine 3, you probably want to know how Doctrine 2 works
under the hood to basically know does this change affect me that they
introduced. And do I have to look at anything that maybe could cause a bug in my
code.

## Introducing The Domain: Order & OrderItem

So we will start by just looking at a simplified domain that we basically we'll
follow through the, through the whole talk. And I only have two entities in the
domain. So it's kind of easy. It's probably easier than all the entities setups
that you have in your projects, but it will be enough to to look at all these
things. So first of all, we have an Order that just has an id. It's an
auto-increment id, very basic stuff that you will probably all know. We have a
timestamp from when that order was created and then it's basically just a list
of OtherItems which we'll look at later. And everything is set via getters and
setters just like most people are familiar with. There are other nicer ways of
dealing with things, with like rich domain models, but where it's more
expressive, but we will just look at something that's kind of ordinary.

And the OrderItem is the same. It has the same auto-incrementing id. It's linked
back to the Order. Um, that's a necessity because OneToMany requires ManyToOne.
We will look at that. It has a name for, for the item that we want to store in
our Order. And it also has a price which is a string, which might seem weird at
first. But if you ever worked in an e-commerce system using floats, it's really
a pain. It just causes rounding errors and, and, and it's just horrible to work
with. So one workaround is to use ints, so integers. So basically instead of
using the euro amount, use the cent amount: 100 cents are one euro and that's
it. And that makes things a little bit safer. And I pushed it to the next level
and used string and you will see later on why we do this.

So let's look at the code. So you probably all have seen this. It's a very basic
entity. We have the usual annotations. In this case we also add a custom
repository class. You probably have seen that as well. We give our table a
specific name. For the `OrderItem` it doesn't matter as much. For the `Order`,
it's more important because `order` is a reserved word in most dbms. So that
kind of messes up things. But when you work with Doctrine, um, so it's nice to,
to give it an alternative name. As I already said, the auto-increment integer,
that's pretty much what you all know. Then we link to our Order. Um, the
interesting thing is that we basically tell the `JoinColumn` that this
`OrderItem` always needs to be attached to an `Order` because it doesn't really
make much sense to have an `OrderItem` just loosely hanging around in the
database. So we just ensure this with the `JoinColumn(nullable=false)`. And the
rest is just basic stuff. We get things, we set things. You probably all have
seen this, you can auto-generate it. So it's really simple.

With `Order` it's kind of the same thing. We have an auto-incrementing id, we
have the relationship this time a `@OneToMany` that maps back to the `OrderItem`
`$order` property. And if you think about the, the table layout, it kind of gets
clear why we needed the other relationship from the `OrderItem` back to the
`Order` because obviously the, the order ID is stored inside the `OrderItem`. It
links back to that. So if we were just to have this relationship with, on our
`Order`, that that wouldn't be a reference to: okay, where is information
stored? It's in this different table. So the, the `OneToMany` relationship is
the only one where you actually have to have a bi-directional association. In
other cases you can just have a unidirectional association.

And the other important thing here is the `cascade={"persist"}`, which come,
will come in later, but just keep it in the back of your head that this is
important. And yeah, for the timestamp I just use the `DateTimeImmutable`
object. It's one of the types that Doctrine supports. We initialize the items
that we want to put into our `Order` is an `ArrayCollection`. You probably have
seen it 100 times and more. Um, in my case, I return the `array` instead of the
`ArrayCollection`. This is just something I like, but there's no specific reason
to do that. Um, and it doesn't really come up later, so I just want to point it
out that this is a quirk I have, you don't have to do that. Um, what is
important is the `addItem()`, we basically, because we have a bidirectional
association, we have to make sure that both the `Order` and the `OrderItem` know
what object they are linked to. So in this case, I set the `Order` um, to
`$this` on the `$item` and also add this item to our current `Order` if that
makes sense.

Um, and yeah, this is where the string part for the price comes in. I use the BC
math extension for PHP. If you use modern php where you have strict-typing and
everything, you can't just pass in an integer or float, it has to be a string,
otherwise you will get a type error and that's why I use the string all the way
through and it will just add those strings, strings up from the items and then
you get the final total price. So it's really simple math. It's just doing plus
current total plus the next item.

## INSERT

Okay. So those are the basics. Um, I know that was kind of fast, but I, I kind
of hope that you got the structure in your head, you know, what we're talking
about in terms of what kind of objects we're dealing with because this is all we
will use throughout.

And yeah, just assume that with our very simplified e-commerce system that we
have now, we want to do a checkout. We select items from a page - I don't have
everything built for this case - it's just, let's assume it happened. And then
we pass things to a `Checkout` service. I don't use the service suffix here, but
this is just a basic service class. Um, I pass in the EntityManager because we
want to store the `Order` when we do a checkout with all the OrderItems. And
this is what the actual checkout method looks like. In my case I just get some
`$cartItems` which are basically products that, uh, in a DTO entity or whatever
that is used on the frontend for displaying my product items in the cart and
checkout. And now I want to transform them to, to an `OrderItem`.

It's basically to make sure that when the product changes, I change the price,
my `OrderItem` is supposed to still have that price because I will still want to
charge the customer the original price they used when they used the checkout.
And then I just do some mapping. I map the name and the price from the
`$cartItem` to the `OrderItem`. I add that `OrderItem` to my existing `Order`
that I created up there in the top, and then I just save that `Order` and I
flush the entity manager. So you probably have all seen this.

This is really basic stuff and still, there are some really interesting things
going on behind the scenes. Because if you think about it, first of all, we are
only persisting the `Order` but the OrderItems will be saved as well. So
Doctrine, for some reason knows it has to save those items into that separate
table. Um, and it also knows that has to do insert instead of just an update. So
let's look at why this happens? How does Doctrine know, which items to save?

## Unit of Work

And that is really interesting and it works based on the unit of work. This is a
really huge object and, and it's kind of a basic pattern. So, um, the, the
example on the right is basically what Martin Fowler has in his pattern catalog.
I omitted a few methods that we don't really care about, but basically this is
an object that stores our changes for the database and, and keeps them in memory
until we flush and we actually want to write them. And then this unit of work,
will figure out what writes it has to do: if it's an insert, if it's an update
and which order things have to happen. And that's really a lot of stuff going on
there and it's a huge class right now, which is probably why in Doctrine 3 that
they want to split it up a little bit. Um, and it's also kind of ugly to look
into it. Um, so it's fun to dive into it if you want to do it. If you have to do
it because things go wrong, then it's just a pain.

So I want to save you from doing that and just enjoy looking into it for the fun
of it. Um, and you can already see that there are some states for, for
insertions, for updates, for deletions. We have the `entityChangeSets`, so what
changes need to be written to the database. And more importantly we see some of
the methods that look a lot of similar to the methods on the `EntityManager`.
Um, so for example, we have a `persist()` method that looks basically the same.
The `remove()` method, the `flush()` method that kind of looks like the
`commit()` method. So that, there's some kind of overlap. And that's no, no
that's intentional because basically the `EntityManager` will pass its called to
the `UnitOfWork` and the `UnitOfWork` will store all those things.

So when we pass in the `Order`, the `UnitOfWork` will figure out: ah those other
properties that map to a database table, I have to basically save the data
that's in there for later change. And I will also have to remember that this
object that they wanted to persist, I have to basically write that to the
database as a new object based on the state.

And the reason why this thing works with the OtherItems as well as because we
have this cascade persist in there. Because when we persist our `OrderItem`,
what will always happen is that the `EntityManager` will also look through the
associations on our entity. So on our `Order` it will look at the the `items`
property and say, "Oh this is an association, the items in there I probably have
to save as well". And then just loops over them and sees either if we persist
them manually or if we have the cascade persist, then obviously Doctrine knows
everything that I put in there I have to store in the database as well on an
insert.

And yeah, this is basically what happens. So we pass in the `Order` and then
Doctrine figures out, the OrderItems in there, I have INSERT them as well and
have to write to the changesets. And this is roughly equivalent to adding a
second persist in our foreach loop for each item. There are some subtle
differences, but basically it works out the same. And if you don't do either of
those things, so neither the cascade annotation or persisting all the objects
manually, what you will get this error. Because, obviously, Doctrine knows that
some objects should be stored in the database because we have the association
there, but it doesn't want to write this unless we tell Doctrine to do so.
Because it would be kind of bad that when Doctrine thinks:

> yeah, I know Dennis is kinda lazy and he probably forgot to
> do the annotation. I will just save it for him.

And that could be fine, but if it just writes stuff to the database that I
really didn't want to have, then yeah, I'm in trouble. So Doctrine says no, I
won't touch that. Figure it out yourself. Do you really want to persist this?
Then either call persist or cascade or just move it out of the collection. So
this is something you have to be aware of when you work with Doctrine that
unintentionally non-persistent entities that come up in the changeset, Doctrine
will not touch them. You have to figure things out. It will give you a nice
error like you saw before where it tells you what to do, but still you have to
check if: is this really something that has to go into the database? Probably
yes, and then you have to act on it and do that. And also you have to add this
cascade annotation to make sure that this happens automatically for you. And
obviously only on the entity that is supposed to cascade. So on the `OrderItem`
we didn't cascade for the `Order`, because I decided that I always want to save
the `Order` with the items but not an item and then create a new `Order` for
that item. Because in my e-commerce context, that doesn't make sense. If you
have this context where this makes sense and your item can be the primary
object, basically, then you can just do that: just add the cascade and it will
also save the new `Order` through the item as well when you persist one item.
And Doctrine will also figure out in what order to do that.

## FIND

Okay. So much INSERT, there was already like really, really lots of stuff in
there. And it's basically getting worse, but at least we have nice pictures. So
let's think about like we're inserting new orders, we do our checkouts and, and
we now want to have a backend where we want to look at our orders. And we want
to have some additional details. So in our order list, we don't just want to
show the order number, we want to see how many items are in there, what will be
the total price for that. And what you can see on the bottom is that this will
issue 2 database queries which might seem odd to some. Um, because yeah, it's
just, we just wanted our order list and that could be one query.

And the reason this happens is you don't have to look at the full Twig template,
but this is basically the bottom part that that shows how many items are in
there and what they cost - the total price. And this is our `Order`, method,
`getTotal()` price that iterates over the items. So I'm not calling the items
themselves, the, the, the method does, but still Doctrine recognizes: ah, those
items I need to get somewhere. And it will just issue the second query for those
items.

And this can get bad if you have more than one item in your overview or one
order in your overview, because for every Order it has to fetch the items. So if
we have five orders, it issues one query for getting all the orders and then
another query for every order to get its items. This is often referred to as the
N+1 problem. So Doctrine will basically issue more queries than you might think
it will. And yeah, this is basically what it looks like in a profiler. So you
can see that on the bottom you have the query for the order list, fetching all
the orders and then for every order of 5 times you get the same SELECT
statement, um, for the items fetching all the items for that order.

## Lazy Loading

And the reason this happens is lazy loading. You probably have heard that
before. And how it works in Doctrine is it uses a so-called proxy pattern. So
when you call `getOrderItems()` or even just accessing the `items` property on
your `Order`, Doctrine has not loaded this beforehand, because we only wanted
the orders in the beginning and our controller. Instead it will see that those
items are not loaded, load from the database. And once that is done and will
return those other items and it will keep them in there. So it will not do the
same query on the same object over and over again. It just remembers: ah these
items were not fetched the first time and that's why I'm fetching it now. And
then subsequent calls will get those items we previously fetched.

And what's kind of responsible is the, I already mentioned the proxy, the proxy
objects that Doctrine generates - at least if you're not in debug mode, so, um,
it might create the folder but you will probably not see files on there, so just
do debug false and you will see those proxy files and we can look at them. They
are huge and ugly and, and I never had to look at them in any real cases. It's
just basically to show you what they look like. Um, you can already see that
it's just an object over our original `Order`. That's also why we can't have,
entities `final`, because something has to extend them - the proxy object. And
then have an initializer and then you have tons and tons of code that basically
is unreadable and, and you don't have to, it's automatically generated and you
don't have to care about it. But you can already see that if I want to get those
items, some initializer is looked at it, and it will invoke a method. And this
initializer is basically responsible for doing the querying in the background.
And then it will just get the items as our method on our parent object basically
described. So, um, this is the magic behind it, so to speak.

And this is how it looks as UML. I hope I got the arrows right the direction and
the arrow tip. I, I'm, I'm not a computer scientist, so I have never learned
this and I never get this right. Um, but yeah, this is basically what it looks
like. You have a wrapper on your `Order` object on the right that has the same
methods, it just has some additional stuff on there that you don't really have
to care about. You don't have to access yourself. Doctrine does this under the
hood for you.

## Fetch Modes

And this is fine if it's actually what you want. So for example, if you fetch
orders and you don't want to access the items, it's really good that doctrine
doesn't fetch the items every time you, you get this order. But when you want to
have the items and it issues these additional queries, then you kind of want to
address this in some cases where it makes the site slow.

And what you, what you can do is add another option to the annotation and say:
this relationship I want to fetch eagerly, so always fetch those items. Um,
obviously this has the same drawback as fetching things lazily does because now
everything is always fetched. And as I said, if we have cases where we just want
to work with the `Order` without items, we now do a query that we don't really
need. So this might not work as well, but, but it's a nice and easy way to
address this issue to, to not lazy-load.

The other kind of more involved approach is since we have our own repositories
and we can write our own DQL queries, we can just tell Doctrine to join the
items in certain cases where we define the method and we just call this method.
So in this case, `findOrderWithItems()`. And the interesting bit is the join
part. You can see that this is DQ, it works on the object level. So we're not
talking to the database table, we're talking about our objects. In this case,
the `Order` object that has alias `o`, it has some `items` property - we already
know that - and this item's property, everything that's in there, in the
association, that's what we want to join. And that's also what we want to select
because if you don't select then it will just join it and throw it away when
hydrating the object. That's basically if you want to ask something from the
items without actually fetching the items, then this can be useful.

And so this is a more involved query because you actually have to write the
query other than just doing `find()`, but, but it can already help you that I
can now figure out my cases where I want to do `Order` with items and `Order`
without items.

## Custom Object Hydration

Another approach that I personally like even better is custom object hydration.
Um, so take for example, a DTO object that looks like this. It's probably really
hard to read. So we have an `OrderSummary` and it describes all the information
we want to see in our Order list. We have the Order id that we want to display,
the created timestamp and then the item count and the item total. Um, and then
we have just getters, we don't want to change this data because this is
basically a read model on our database and we just want to fill this model with
the data. And we can still use the DQL query. Let's go through it, like not in
chronological order. So first we see that we fetch all the stuff that we need
for our `OrderSummary` and the interesting part here is that we count the items
in the database and also we use the SUM method in the database. So we don't have
to run the PHP query stuff. We can actually do this in the database which is
really fast for doing the kind of integer operations. Then we still fetch the
data as if it were the `Order` object. So we say every data that you would get
from the `Order` object, just just get it. And then we extract the data we want.
And then we just create a new object with our DTO in there. And basically it
will map the data not to the `Order` object but to our DTO. And this will also
perform only one query and it has all the data in there and we don't even need
all the item information that we don't need and we just need the stuff, we get
the stuff that we need. And this is what it would look in plain SQL. So probably
a little bit simpler, but yeah, the, you just have to translate that back to
DQL. Or you can even use sql and use the `ResultSetMapping` to map it back to
the object instead.

And so, just to remind you, this is the queries that we had before. And this is
what we get now: we have one query you, can see in the bottom, it kind of looks
like the SQL query we just saw, Doctrine just uses different identifiers. And
it's also a lot faster, like 3 ms, roughly 3 ms. And 3 ms doesn't sound like a
lot, but obviously it adds up. If you have more than six items, like 1,000
orders.

And yeah. So basically your takeaway for this is: Doctrine loads, associated
entities lazily by default, which can be good, but it can also be a problem for
querying. So you can do custom queries with a join and then select those items
yourself. You, you can do a custom hydrations, either into a DTO object or if
you just need a scalar value, like a count, you can just get this as well and
you don't even have to wrap it into an object. And this can improve read
performance tremendously for you.

## Identity Map

And something that kind of relates to that, to optimizing querying, is identity
map. This is also a pattern that that's wrapped inside the `UnitOfWork`. So
whenever you call the `EntityManager` `find()` method with the object class name
like `Order::class` and then the id you want to fetch. What will happen is, the
first thing that the `EntityManager` does is look into the `UnitOfWork`, into
the identity map to see if someone, well actually, if we performed this query
earlier before and we already have the object in this identity map. And if we
do, then it will just get the object from the identity map and don't even issue
a query. If not, then obviously it will just do the database query as before
the, the typical `find()` method and it's fine.

So, um, just to, to basically see how this works. This is very simplified code
with just some, some dump in there to see the results. So we do the same
`find()` twice and in between we look at the `UnitOfWork` and at the end we
looked at the `$order` and the `$sameOrder`, um, are actually the same object.

And yeah, this is what it looks like. So the identity map contains the class
name and then an array where the ID is the key and the object itself is the
value. And this is how Doctrine looks it up. Like, okay, I have this class, I
have this id, the user wants that, so I have to pass this object on. And on the
bottom you see that yea, both objects are actually the same identical object. So
every change you do to the object is basically kept and the when you do find
again, then you get the object that you already had in memory, it doesn't issue
an additional query. Um, and you can see that, on the bottom as well, there was
only one database query in this case. And this is basically for our `Order`
object with the find items, it will, at the same time also fetch those items as
well and put them into the identity map as well. So if you want to fetch an
`OrderItem` later on by its id, then it will also not issue a query because it
already got this item through the `Order` with the OrderItems. And obviously the
downside is that it keeps a lot of data in the memory. But also it's probably
doing one request, you won't have like dozens and dozens of entities that it has
to keep in the identity map and, and has to work with.

And yeah, so basically that's another important point when you want to improve
your read performance, you have to look out that when you use the repository's
`find()` method, it uses the criteria array thing, which is not the same as the
`EntityManager` `find()` method. So it will actually perform the query twice. It
will still fetch the data from the identity map, but it does the query because
it can't figure out that, even if you just fetch the id, the find method or your
custom query, the EntityManager, the UnitOfWork, don't know how to introspect it
and figure it out. But if you're using `EntityManager::find()`, then you can
actually have fewer queries than you would expect.

## UPDATE

Okay. So we looked at INSERT, we looked at the `find()` method. Let's also look
at updates. Um, so let's assume we have an object, we create it new. And then we
set the ID. Some people might assume that since my object now has an ID, I can
just update this object if it's already in the database. And the thing is,
obviously, this will not work, because the `UnitOfWork` keeps track of an
entity's state. And since it didn't know the object before, it, it doesn't know
what to do. It assumes it's a new object. And so it wants to insert that new
object. Because internally, Doctrine keeps track of entities not by the ID but
by the object hash. And so Doctrine has a set of states that it kind of looks at
and, and tries to sort entities into. Obviously, for new entities, the new
state, the managed state, when `UnitOfWork` knows of this object, it's already
managed somehow, that that's both new objects, updated objects and deleted
objects and detached objects. So objects we don't, no longer want to maintain
via the `UnitOfWork` and via the `EntityManager`. So all the changes we do to
that object, they will not be persisted, they will not be saved. And removed
objects, or something that's basically said that: once you do flush, we want to
delete this.

This is how it looks on the code. Um, there are some nice explanations in there.
And so basically what happens is, since we created a new object, not via the
`UnitOfWork` and the `EntityManager`, but by ourselves, the `UnitOfWork` doesn't
know what kind of object is this, and it says: ah, this must be new, I will give
it the new state and I will insert it. And on the other hand, if we fetch an
object via the `EntityManager`, the `UnitOfWork` now knows that this entity was
fetched before, it's already part of the `UnitOfWork` inside the identity map
for example. And that's why, when we do this, we no longer need to persist
obviously, because it's already in there. Now it knows that I can update this
object: this is not something new, this is something I have to update. And so
basically, for you, what you have to look out for is, just, kind of, keep in
mind that the `UnitOfWork` manages the state for you. And you don't, you have to
make sure that everything you want to insert is part of the inserts sets and so
on. So yeah, it's, it's just, it's a lot. I know last session, but we're almost
done.

## Active Record

So finally, which is kind of unrelated to everything before but, but I promise
it in the abstract, so I figured I have to deliver it, is a kind of comparison
to, to active record.

So I don't know how many of you know that Doctrine uses the so-called "data
mapper" pattern to map things. And another famous approach as active record.
Actually Doctrine 1 used it, Propel was another famous object relational mapper,
that uses active record pattern. In Ruby on Rails you have the, I don't know
what it's called. And obviously Laravel is famously using Eloquent, which is
also using the active record pattern. And what the active record pattern does is
it wraps an object in a way that everything related to the database is part of
this object. So usually what happens is you extend the base model, base record
or whatever, and in this base record you have all the database-related stuff.
You can query things, you can save things and then you create your new entity,
with some restrictions as to how the entity is created. And and then you can do
all the database operations on there. So basically the models allow for, for
querying the data and you don't need special classes like the `EntityManager` or
repository or anything else, you just do everything for the record.

So this is how it looked in doctrine 1: the abstract `Doctrine_Record` class.
This would be what you would extend. And you can already see that there are some
states in there, so it kind of still does all the same things that Doctrine 2
does with the keeping the entity states, for example. So there's this kind of
the same feature set, but everything is part of this basic model. And sure there
are things that work differently but there's even a proxy thing, so it kind of
looks the same. So just to give you kind of look at how you would act on an
active record kind of model. And I didn't put the, the entities on there as a
record, because based on the library they will look differently anyway.

But roughly, this is what would change. So first of all, since we don't have the
`EntityManager` anymore, all operations are on our model, we just call an
`$order->save()` and this bubbles down to the record and this is where
everything is implemented, it uses a connection. Same with an update. Basically
instead of getting the `EntityManager` to find our object, we use a static
method called for `find()` for example or query to get a new `Order` object. And
obviously the `flush()` has to go as well. We still do the `$order->save()`. and
so we don't need any additional classes, which might seem nice. We actually saw
that the code kind of gets a little bit more compact, so that's less stuff to
do.

And just just to keep kind of a record of what are the benefits and drawbacks of
it. Um, because it, it really depends on what your use cases. Um, so if you look
at the data mapper, um, what's really neat is that basically you don't have to
care about all these additional classes doing all these things. Everything is
inside my model and I don't have to to remember to inject anything. I can just
work with my object and do everything on it that I want to do, like saving stuff
which can be nice. It's, I don't have to use injection in the service to get the
`EntityManager`, I'll just pass the object and say save. And that's it.

And obviously this comes with a downside that, that my object has to fit some
conventions that this record gives me, to, to basically be able to figure out
what stuff to write and what not. And with Doctrine you do it very implicit, ah
explicitly. You put the annotations on there, you have the yaml file. But with
an active record, you basically have to figure out what do they want me to do?
Where do I have to store my properties, how do I give it the table name? Um,
maybe there are some limitations, like usually you can't use the constructor
because this initializes some, some basic database stuff which can be annoying,
when you want to do like, rich domain models for example, and you want to have
this constructor, you have to remember to, to call parent construct. And yeah,
the, the obvious benefit is that it's less code to write. It's just easy: save,
no `EntityManager`, no custom repositories, you can just go ahead and do things.

## Summary

And just to wrap things up. So what did we look at? First of all, um, the, the
Doctrine `EntityManager` uses underneath the UnitOfWork which does all the heavy
lifting and storing the entity changes until they are persisted. It takes care
of the commit order and everything. The proxies for, for finding stuff are
useful that it basically allows us to lazily load stuff if we want to, and if
not then we can just bypass it by writing custom queries. And the identity map
as well for finding stuff is really useful when it comes to minimizing queries
and to make sure that we, when we fetch an object from the database and it's
something that we worked on before that, that it still has the same state: we
work on a consistent object and not like one OrderItem of the name x and one
OrderItem of the name y, and then we want to save it, they don't really fit
together or the last one wins.

And yeah, that's basically it. So I don't know how much time I have left. I
think I rushed through it quite a lot more than I would have to. Um, so yeah,
plenty of times for questions I guess.

## Questions

Yeah. Wait, there's a microphone. Cool, thanks, thanks for catching that, so I
don't look bad.

(unaudible) Doctrine to handle concurrent persistent in different processes, the
same database?

Um, so, so again, uh, how does Doctrine handle different repositories?

No, if Doctrine is running into 2 processes, php processes?

In different PHP processes. Yeah. So, um, when you have different PHP processes,
you will obviously have different unit of works. So there, things will, will not
apply, in terms of, you will not have the identity map from one process work
with the other process. Um, I'm not sure if there's some common caching you
could use, like the l2 caching things like that. I haven't looked into that. Um,
but yea, so those things are really, if you work on one php process and, and you
just want to access your data there, which is kind of the most common use case.
I can't really tell you how it works with multiple PHP process. So, um,
everything is good. Um, I think there was someone in the back who had a question
or not.

Okay. Uh, since we all have some time, I don't know how eagerly you want to go
to, to the jeopardy, I kinda do. But, um, I, I have some code if you kind of
feel that this, this was like rushing through everything and just getting a
glimpse here and there. If you want, I can give you a quick rundown of the code
for, for example, the querying of how it looks in a real project to make it more
hands-on and a little bit more approachable. Um, I see someone not? Okay, then
let's just do that for, for five minutes. And whoever else wants to leave for,
for the, the Jeopardy or just to get some drinks or whatever, feel free to do
that.

So only two minutes. Okay then, then we will do it really fast. I, I'm used to
that. So I already did this with the talk. So, um, let's look at our list orders
controller. So in this case I used an invokable controller. So one controller
only has one action, the invoke action. Um, oh, it doesn't, wait a second. Well,
since the time it's up anyway. Let's do it differently. Whoever was interested,
you can come up and I will, I will show you on my notebook and since I can't get
the screen to work. And everyone else, thanks for staying here, for, for
listening to my talk. Please give me feedback. Um, and yeah, have a good day and
hopefully see you around.
