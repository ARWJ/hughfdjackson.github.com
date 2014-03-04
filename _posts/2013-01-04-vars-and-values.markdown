---
layout: post
category: javascript
tags: javascript variables values
---

Sometimes I hear JavaScript dev throw around the terms `vars` and `values` as synonyms for each other.  Realising that they're not was a serious *'aha'* moment for me - and making the distinction about it can make talking and thinking about code clearer by far.

# values

JavaScript values split neatly into two camps - objects and primitives.

### Primitives

*Primitives* are familiar to us all - in the form of strings, numbers, booleans, `null` and `undefined`.  They're immutable - meaning you *cannot* alter a value.  If you want to alter one, you have to create a new one by performing some operation on the original.

Primtives implement *value equality* - two primitives are considered equal, even if they were created separately, if they represent the same logical value: `1 === 1`.

### Objects

*Objects* are mutable by default, and are most commonly found in the form created by `{}` or `[]` - often referred to as objects and arrays (since the forms are sugar for `new Object` and `new Array` respectively).  Objects are mutable - meaning that you can alter the very value itself.

Objects implement (what I will term for the sake of convenience) *equality by identity*.  An object gets a unique identifier on creation, and is only considered to be equal to another object when that unique identifier matches.  Therefore, `{ x: 3 } !== { x: 3 }`.

# vars

A variable is simply a *name* for a value within a scope, that can be changed. They can be created by the variable statement, by a function declaration, or via a parameter.  You could think of it as a box that holds an address to a value - it's a conceptual model that I haven't managed to break when thinking about JS.

{% highlight javascript %}
// creating variable `a` via the var statement, and assigning the value `1` to it
var a = 1

// creating variable `b` via a function declaration,
// and the variable `c` as the parameter to that function
function b(c){
	var d = 'my value'
}
{% endhighlight %}


# Just a couple of examples

So how does this affect common, every-day situations that you want to talk to others about?

## for loops

The primary mechanic of a for loop is by replacing the value held by a var; not from altering the value:

{% highlight javascript %}
for ( var i = 0; i < 5; i = i + 1 ) console.log(i)
{% endhighlight %}


In the above example, variable `i` holds the value `0` intially.  In each subsequent loop (while the `i` still holds a value less than 5), the variable is updated to hold a new value, created by operating on the *old* value.

## passing arguments to functions

When you place a variable in the comma-seperated, parens-encapulated part of a function call, you're really simply taking the values variables hold, and passing *those* to a function.

{% highlight javascript %}
var add = function(a, b){ return a + b }

var num1 = 1
var num2 = 2

add(num1, num2) //= 3
{% endhighlight %}


In this case, `num1` and `num2` get substituted with `1` and `2` respectively.  Within the body of the function, `1` gets assigned as the value held by  variable `a`, and `2` gets assigned as the value held by `b`.

You can prove this is the case by changing the variable *within* the function, and then inspecting the variable that referred to the same value *outside* the function:

{% highlight javascript %}
var inc = function(a){ a = a + 1; return a }

var num = 1

inc(num) 	//= 2
num 		//= 1
{% endhighlight %}


The value assigned to a function's parameter is *identical* to the value passed in.  We've already seen that that means that a variable gets the same value as far as primitives are concerned - but the same is true of objects.  This means that any change done by a function on that object will be seen when any other variable that refers to that object is inspected.


{% highlight javascript %}
// takes an object, and adds the .isMutated property
var mutate = function(o){ o.isMutated = true; return o }

var myObject = { x: 3 }

mutate(myObject) //= { x: 3, isMutated: true}
myObject         //= { x: 3, isMutated: true}
{% endhighlight %}


This might trick you into thinking that the operation within `mutate` actually changed which value `myObject` holds - but the primitive example shows this is not the case; altering one *variable* does not affect another variable. it's the value itself that gets mutated.

# At the end of the day...

It's possible that every dev knows this - and those that use the terms interchangably do so out of laziness.  I'm often convinced this is the case when I talk to people I know are fairly up-to-speed on other aspects of programming.

Still, I think there's a lot of value in separating the two concepts, because it leads to a far clearer understanding of what you have to worry about changing where.

Deeper discussions need more crystal clearly defined terms - heaven knows we have the odds stacked against us by our choice of subject matter from time to time in any case.
