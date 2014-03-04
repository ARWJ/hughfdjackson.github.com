---
layout: post
category : javascript
tags :  javascript jslint doug crockford
---

Doug Crockford's [JSLint](http://www.jslint.com) performs static analysis on Javascript, returning a list of what Crockford considers to be violations of best practice.  Contrary to the assertions of its creator, however, it is **not** a fit tool by which to judge the quality of 3rd party code.

### Crockford's Assertion

Crockford is an opinionated, experienced and charismatic guy.  As such people often do, he has gained significant influence.  In this post, I'm going to focus on what Crockford thinks the role of JSLint is, as typified by this [suggestion](http://therichwebexperience.com/blog/douglas_crockford/2006/05/choose_your_ajax):

>   "There is one reliable, objective metric of JavaScript code quality that can help you in making your choice *\[... of library ...\]*: JSLint. If JSLint finds problems in a library, then dump it and move on to the next one."

I firmly believe that this attitude is damaging; and worth focusing on given his potential influence.

# 1. JSLint's Mechanism and Limitations

JSLint enforces Crockford's list of *suggested practices*, where *suggested practice* can be defined as:

>   "A *suggested practice* is a rule of thumb to aid in the avoidance of a problem or problems"

Notice, the definition deliberately does not imply that this rule of thumb is the only effective one.  In software development, that's a cat you can skin many ways.  It's for that very reason the phrase *best practice*, which implies a solution's monopoly, has been avoided.


### 1.1 A Case Study in the Diversity of Suggested Practice

Javascript has a system of Automatic Semicolon-Insertion (ASI).  A Javascript engine will analyse the code that has been written, and follows a set of rules to determine if a statement has ended wherever it encounters a new line.  This lets you omit semicolons in many instances.

However, Javascript's ASI can confuse newcomers (and some experienced developers too). For instance, this:

{% highlight javascript %}
console.log('hi')

(function(){
    console.log('there')
})
{% endhighlight %}

will be interpreted as this:

{% highlight javascript %}
console.log('hi')(function(){
    console.log('there')
})()
{% endhighlight %}

Crockford's *suggested practice* is to avoid this problem by never using newline to *imply* a semicolon:

{% highlight javascript %}
console.log('hi');

(function(){
    console.log('there');
})()
{% endhighlight %}

However, you could achieve the same thing using a different *suggested practice*:

{% highlight javascript %}
console.log('hi')

void function(){
    console.log('there')
}()
{% endhighlight %}


Both practices will reliably produce the same results, across es3-5 supporting engines.  That covers every engine commonly in use today.


### 1.2 The Grey Areas

Not least amongst the limitations of JSLint as a judge of 3rd party code quality is the debatable nature of many of JSLint's restrictions.  For instance, Crockford eschews braceless, single-line `if` statements, such as:

{% highlight javascript %}
    var foo = true;

    if (foo === true) foo = false;
{% endhighlight %}

The argument is that Javascript users, especially inexperienced ones, are liable to forget that they cannot have more than one statement in the body of a brace-less `if` statement.  A counter argument may state that familiarity with the language's semantics mitigates this risk, and that the reduction of visual clutter can lead to easier maintenance; an argument that personal experience bears out.


### 1.3 A Note On Optional Parameters

If it is assumed that Crockford's suggestion implied code should be 'quality checked' in JSLint with its default options, a myriad of purely aesthetic constraints are applied as well.  For instance, of the three following `if statements`, *only one is acceptable in JSLint*:

{% highlight javascript %}
var foo = true,
    bar = false;

if (foo === true) {
    bar = true;
}

if (foo === true){
    bar = true;
}

if ( foo === true ) {
    bar = true;
}
{% endhighlight %}

Also note that reformatting the variable definitions will cause an error in JSLint:

{% highlight javascript %}
var foo = true
  , bar = false;
{% endhighlight %}

Whereas other *suggested practices* have been intended to guide the user from 'gotchas', these simply enforce the style preferences of JSLint's creator.


### 1.4 Global Detection in JSLint

In spite of the reservations expressed above, JSLint does offer the potential to detect, and warn about, global variables added by any means.


# 2. The Potential Damage of Treating JSLint as an Enforcer of Best Practices

In section 1, passing mention was made of the concept of *best practice* in relation to JSLint's role.  It was noted there that *suggested practice* was a more applicable label.  However, Crockford's comment suggest that he *does* regard the rules JSLint enforces as *best practices*.  The comments of his adherents in public forums often suggests that this is the way in which they view his suggestions.

This worries me deeply, for the following reasons:

#### 2.1 Discarding Good Libraries

The most obvious danger is that internally consistent code with robust behaviour will be discarded without due cause.  It may surprise some to know, for instance, that [jQuery fails Crockford's acid test](http://docs.jquery.com/JQuery_Core_Style_Guidelines#JSLint).

### 2.2 Ignorance in the Community About Javascript

The phrase *best practice* is interpreted by some as license to not learn the different sides of any given debate.  This may lead to a misunderstanding of language features such as ASI, and even including `with` and `eval`.  Unfortunately, this does not appear to be uncommon.

### 2.3 Causing Stagnation in *Suggested Practices*

If running libraries through JSLint becomes common practice, any library hoping to obtain users must adhere strictly its suggestions, *even against the better judgement of the author(s)*.

# 3. Better Practices In Choosing Libraries

In lieu of using the approach Crockford proposes, the following is suggested:

### 3.1 Test Suites as a Means to Judge Code Quality

Not only does JSLint enforce a suite of practices that range from personal taste to one possible solution amongst many, it does not enforce the quality that is of prime importance in any library; namely, that is behaves as you desire on the platforms required.

A far superior filter is a comprehensive suite of tests that can be inspected and run.  These act as both a description of the behaviour of a library and executable assurance that it performs as advertised.  

### 3.2 Environmental Impact

One possible interesting metric is the number of global variables/properties a library adds, or the number/nature of properties added to native or host object/their prototypes.  As mentioned above, this is one use case in which JSLint can help retrieve useful information.

### 3.3 Documentation, Community and Code Readability

If you can find an active community surrounding a library, or the documentation is clear and consistent, the time and frustration saved can be significant.  Sometimes code is the best documentation, so code that you find readable is a plus.

### 3.4 Active Maintenance

Look for signs that the project is being actively maintained, since it increases the chance that problems can be resolved as they arise.

### 3.5 Reputation

A crude, but often useful metric.  A popular library will have more written about it - both good and bad, making it easier for you to inform your choice.  It is also likely to behave consistently, as unreliable libraries are likely not to have survived the darwinian gauntlet of public opinion.


# 4. Conclusion

JSLint is not a tool without a purpose.  You may use it as a way to adhere to Doug's *suggested practices*, and so aid in avoiding those pitfalls.  It's also potentially worth considering in order to keep the coding style of a team consistent.

However, despite Doug's claims, it is *not* suitable for judging the quality of other people's code.

Nor is it a substitute for testing the behaviour of your programs.

Nor does it exempt you from learning about the language in which you're writing.

---

**Edit:** Removed the link to JSHint, as per [this comment](http://news.ycombinator.com/item?id=3585186)
