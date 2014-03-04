---
layout: post
category : javascript
tags :  javascript functional object-orientated
---

Objects and methods are a big deal.  They're the core way in which JavaScript defines functionality against a type; which is reflected in the standard library, in many of the libraries we choose to use, and how production code tends to be written (at least, in my experience).

Compare these two solutions for returning the name of all the audiophiles in a contact book:

{% highlight javascript %}

var contacts = [
    { name: 'Jen', interests: ['audio', 'gaming', 'running'] },
    { name: 'Ben', interests: ['skating', 'eating', 'tap-dancing'] },
    { name: 'Ken', interests: ['partying', 'life in plastic', 'audio'] }
]
{% endhighlight %}


A first-pass underscore solution might look something like this:

{% highlight javascript %}
var audiophileP = function(contact){ return _.contains(contact.interests, 'audio') },
    getName     = function(contact){ return contact.name }

_.map(_.filter(contacts, audiophileP), getName)
//= ["Jen", "Ken"]
{% endhighlight %}

The same can be achieved by chaining native methods:

{% highlight javascript %}
contacts.filter(audiophileP).map(getName)
//= ["Jen", "Ken"]
{% endhighlight %}

The second is far cleaner, and easier to read; especially as transformations grow beyond a handful of steps.  Many libraries recognise this, providing APIs that are designed to be chained; commonly called Fluent APIs.  Underscore even ships its own version, allowing you to use all of underscores methods with any object, at the price of a little boilerplate:

{% highlight javascript %}
_.chain(contacts).filter(audiophileP).map(getName).value()
//= ["Jen", "Ken"]
{% endhighlight %}

The ordering is clear, but the intent is somewhat obscured.

## Code Organisation Beyond Toy Examples

In these examples, it's clear that method chaining, either with native array methods, or a wrapped underscore object, is up to the job of generating reasonable, concise code.  The disparity between real-world use-cases and toy example is that real-world software *grows* as time goes on.  It grows a whole lot.

Different use-cases often call for different behaviours against similar data, which leads to a proliferation of necessary moving parts; and need for reasonably concise, readable code is as great as ever.  The question of where to put this behaviour, and how to share it, is one of the essential debates in the nitty-gritty of software engineering.

If we give the above transformation the name `getAudiophileNames`, we can see this debate in action:

Object-Orientation would hold that the actions against the data should be shipped *with* the data.  Adding a `getAudiophileNames` method in es5 might look like this:

{% highlight javascript %}
var contactsOO = _.extend(contacts, {
    getAudiophileNames: function(){
        return this.filter(audiophileP).map(getName)
    }
})

contactsOO.getAudiophileNames()
//= ["Jen", "Ken"]
{% endhighlight %}


Note that we can compose these parts as easily as ever:

{% highlight javascript %}
var yellString = function(string){ return string.toUpperCase() }

contactsOO.getAudiophileNames().map(yellString)
//= ["JEN", "KEN"]
{% endhighlight %}


I find myself shying away from using methods as the default unit of work in my code; each essential data type has so many fundamental units of work associated with it, it would require adding methods of many different granularities to get the work done.  These methods would be globally available, which would increase the complexity of working with that object type across the code-base.  It also increases the risk of subtly breaking code elsewhere; especially if that code uses duck-typing to establish what kind of object it's dealing with.

Instead, a lot of work happens at the level of fine-grained functions (pure ones, as often as I can).  These functions are either generic and shared, or very specific and scoped within the area to which they specifically relate.  `getName` and `audiophileP` are two such examples, as is the following `getAudiophileNames` implementation.

{% highlight javascript %}
var getAudiophileNames = function(people){
    return people.filter(audiophileP).map(getName)
}
{% endhighlight %}

When used with chainable methods (such as `filter` and `map`), these work fantastically.  However, interoperability when using functions that take the *entire* object is more troublesome.  In most cases, it requires breaking the chain:

{% highlight javascript %}
var evenP = function(num){ return num % 2 == 0 },
    capitalizeEven = function(letter, i){ return evenP(i) ? letter.toUpperCase() : letter },
    capitalizeEveryOtherLetter = function(string){
        return _.map(string, capitalizeEven).join('')
    }

var firstAudiophile = getAudiophileNames(contacts)[0],
    log = console.log.bind(console)

log(capitalizeEveryOtherLetter(firstAudiophile))
// "JeN"
//= undefined
{% endhighlight %}

## Tapping into the Method Chain


Using functions in this way seperates the concerns about what we're doing and where it's available,  but it's not nearly so easy to read.  One simple, but drastic, way to improve this can be fit into just one line (in a minimal implementation):

{% highlight javascript %}
Object.prototype.tap = function (fn){ return fn(this.valueOf()) }
{% endhighlight %}

This gives us the ability to clean up our example significantly:

{% highlight javascript %}
contacts.tap(getAudiophileNames)
        .tap(_.first)
        .tap(capitalizeEveryOtherLetter)
        .tap(log)

// "JeN"
{% endhighlight %}

Now we happily borrow functions and use them as though they were chained methods; meaning that we could write the entire transformation from front to back as an easy-to-follow chain:

{% highlight javascript %}
contacts.filter(audiophileP)
        .map(getName)
        .tap(_.first)
        .tap(capitalizeEveryOtherLetter)

//= "JeN"
{% endhighlight %}

Compared to the equivalent, unbroken, naive functional equivalent:

{% highlight javascript %}
capitalizeEveryOtherLetter(getName(_.first(_.filter(contacts, audiophileP))))
//= "JeN"
{% endhighlight %}

### Taking extra arguments

Many generic functions require more than a single argument; so it would extend `.tap`s usefulness to cater to these functions:


{% highlight javascript %}
Object.prototype.tap = function(fn){
    var val = this.valueOf()
    if ( arguments.length == 1 ) return fn(val)

    var args = [].slice.call(arguments, 1)
    return fn.apply(null, [val].concat(args))
}
{% endhighlight %}

Now there's really no difference in flexibility between tapping a function into the method chain, and a regular method:


{% highlight javascript %}
var shallowExtend = function(o, o2) {
    for ( var p in o2 ) {
        if ( o2.hasOwnProperty(p) ) o[p] = o2[p]
    }
    return o
}

contacts.filter(audiophileP)
        .tap(_.last)
        .tap(shallowExtend, { friendType: 'bestest' })
        .tap(JSON.stringify)

//= "{"name":"Ken","interests":["partying","life in plastic","audio"],"friendType":"bestest"}"
{% endhighlight %}


## Trade-Offs and Dragons

Normally, touching the global objects, let alone prototypes, raises the hackles of any serious JavaScripter.  This is for good reason; it impacts *all* code; yours, your libraries, your users; everyone will have .tap.  Also, in older browsers, we don't have the option to specify that the method should be non-enumerable, so there's risk of it showing up in loops, or using functions like `_.extend`.  Finally, since duck-typing is the defacto way to specify interface constraints in JavaScript, adding this method risks subtle breakages.

However, if there ever *was* a good case for offending this well-honed sensibility, I think that a `tap`-like method has it.  JavaScript is essentially built on two behaviour sharing mechanisms; first-class functions and objects.  At the moment, interoperability in a clean, consistent way isn't supported out of the box.

At the price of one global method, we've basically separated the concerns of concise, readable code and the scope of a unit of of work: functions hidden or shared as needed, vs a method always tied to an object type globally.

## EDIT: A Real Implementation

An implemenetion of this is available on [github](https://github.com/hughfdjackson/tap) and [npm](https://npmjs.org/package/tap-chain).  Notably, this implementation *does not* change the global object (or any others).  Instead, it provides a `mixin` function that adds tap to an object passed in (making the property non-enumerable if possible).
