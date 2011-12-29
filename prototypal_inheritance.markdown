# Prototypes: the short(est possible) story

Prototypal inheritance is a brilliantly simple concept.  Javascript's syntax, however, often confuses new-comers.  Let's solve that, ehy?

## What is a prototype?

A prototype is a _fallback_ object:

```javascript
    
    // let's pretend that person is the prototype of me :

    var person = { has_legs: "true" }
    var me = { name: "hugh" }

    // that means :

    me.name         // => "hugh"
    me.has_legs     // => true

```

The steps for property lookup (e.g. `me.name`) are very simple: 

1) Look in `me` for the property.  If found, return the value of that property. else, step 2.

2) Look in `me`'s prototype object for the property.  If found, return the value of that property.

## But wait!

"`person` is an object", I hear you cry, "what if *it* has a prototype?".  Fear not! you simply repeat the steps above for finding a property in `person`.  Consider this:

```javascript
    
    // let's pretend that animal is the prototype of person : 

    var animal = { is_alive: true }

    // and, as before, person is the prototype of me : 
    var person = { has_legs: "true" }
    var me = { name: "hugh" }
 
    
    person.is_alive     // => true
    me.is_alive         // => true

```

`me.is_alive // => true` may be a little baffling, but it's really rather simple when you think it through:

1) look for property 'is_alive' on `me`.

2) it hasn't been found, so look on `person`.  

3) it hasn't been found, so look on `animal`.

4) it has been found, so return the value `true` to `person`.

5) `person` returns `true` to `me`.

6) `me` returns `true`.

*N.B. This is just a mental model for thinking about how the lookup works; the actual implementation may vary*

This 'chaining together' of prototypes is known (unsurprisingly) as the `prototype chain`.  Think about it! This means you can add a property *at any time* to `person` or `animal` and that will be available on `me` for all to see. 

## How to use them!

Up till now we've been pretending that `person` is the prototype of `me`, but it's all been a lie.  Let's resolve that:

### Using Object.create

```javascript

    var animal = { is_alive: true }

    var person = Object.create(animal)
    person.has_legs = true

    var me =  Object.create(person)   
    me.name = "hugh"

```

Yup, it really was that straightforwards.  This is the nicer syntax modern JavaScript engines provide us.

### Using Constructors

This is the old, nasty, horrible syntax.  It's the only reason anyone gets confused about prototypal inheritance; of that I'm convinced.  

Constructors are functions that, when called in conjunction with the `new` keyword, return a fresh object.  If you want to use a function as a constructor, the convention is to use a capital first letter:

```javascript

    function Person(){
        this.has_legs = true    //  When a function is called with `new`, `this` refers to the object we're creating. 
    }
    
    var person = new Person()

    person.has_legs // => true

```

So, now we have a `person` object, how do we make sure that it is the prototype of `me`?  Like so:


```javascript
    
    function Person(){
        this.has_legs = true
    }

    function Me(){
        this.name = "hugh"
    }
    Me.prototype = new Person() 

    var me = new Me()

    me.has_legs // => true
    me.name // => "hugh"

```

When you call `Me` with the `new` keyword, the fresh object (represented by `this` inside of the constructor function) has it's prototype set to Me.prototype.  After that, the body of the `Me` function is run, and the object represented by `this` is returned.  *Phew*.  What a load of confusion that causes.


*Why did I leave out `animal` in these examples?  Well, because this technique requires a lot of boilerplate, and I didn't want to make it any more confusing than necessary.*

## Further reading!

This overview is *very* short.  Heck, I haven't even mentioned methods! I wanted to make a super short intro, though, because there is [a really awesome in-depth overview of JavaScript OOP already](http://killdream.github.com/blog/2011/10/understanding-javascript-oop/index.html). This article was intended a light-weight compliment to that one.  Go read it.  Now.
