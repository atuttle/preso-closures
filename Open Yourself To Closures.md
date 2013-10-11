# Open Yourself To Closures

<p align="center">Adam Tuttle<br/>
@AdamTuttle<br/>
fusiongrokker.com

---

# My Qualifications

* I started programming when I was 14, in 1996.
* I started writing closures in 2005, with JavaScript.
* I first heard the word closures in 2008, triggering imposter syndrome.
* In 2009 I realized that I had already been using them for years.
* I grokked them by early 2011; and my JavaScript got much cleaner as a result.

---

# What is a Closure?

## More than just a function

A closure is a special type of function. It's a superset of normal functions. A Function _and_...

A closure is a function-object -- a function that can be passed as an argument to another function -- that always remembers its context.

---

# Context?

What do I mean by context? Let's look at ColdFusion scopes to answer that question.

	function callMe( name ) {
		return "Hi, #name#.";
	}

	callMe( 'maybe' );

When we run this code, what is returned?

	Hi, maybe.

Why?

---

# The Scope Chain

The Scope Chain is the list of all available scopes that ColdFusion will check when you reference an un-scoped variable.

	return name; //uses scope chain

	return arguments.name; //does not use scope chain

I wrote a blog post about scoping your variable references back in 2007, titled [**Scope Nazi**](http://fusiongrokker.com/post/scope-nazi), because I had seen a rash of bugs caused by developers not understanding the scope chain. I knew I was right, but I had _no idea how right I was_.

In 2009 I wrote another blog post about [**Scope Priority Changes in ColdFusion 9**](http://fusiongrokker.com/post/scope-priority-changes-in-coldfusion-9). I could probably do an entire hour presentation on the scope chain alone, but that's not what you're here for. The important thing to remember is that the scope chain is a crucial topic for developers to understand.

---

# A more complex example

	<cffunction name="callMe">
		<cfargument name="name" />

		<cfset local.name = "Uhura" />

		<cfset local.q = queryNew("name") />
		<cfset queryAddRow(local.q, { name = "Spock" }) />

		<cfoutput query="local.q">
			<cfreturn name />
		</cfoutput>
	</cffunction>

	<cfset variables.person = callMe( "Kirk" ) />

What's the value of `variables.person`? To answer that, we have to search the scope chain.

---

# Adobe ColdFusion Scope Chain

1. Function Local (if inside a function) or Thread Local (if inside a thread)
1. Attributes (if inside a thread, NOT custom tags)
1. Query
1. Variables
1. CGI
1. CFFile (if an applicable CFFile tag was used)
1. Url
1. Form
1. Cookie
1. Client (if client variables are enabled)

> _What about Application, Session, Request, and Server scopes? They're available but must **always** be explicitly referenced._

---

# So to answer the question...

	<cffunction name="callMe">
		<cfargument name="name" />

		<cfset local.name = "Uhura" />

		<cfset local.q = queryNew("name") />
		<cfset queryAddRow(local.q, { name = "Spock" }) />

		<cfoutput query="local.q">
			<cfreturn name />
		</cfoutput>
	</cffunction>

	<cfset variables.person = callMe( "Kirk" ) />

**On Adobe ColdFusion 10+:** Kirk (Arguments is first)<br/>
**On Railo:** Uhura (Local is first)

---

# How Does This Explain Closures?

Simply put: Closures _add a new scope to the scope chain_.

1. Function Local (if inside a function) or Thread Local (if inside a thread)
1. Attributes (if inside a thread, NOT custom tags)
1. Query
1. **<span style="color:orangered">CLOSURE CONTEXT</span>**
1. Variables
1. CGI
1. CFFile (if an applicable CFFile tag was used)
1. Url
1. Form
1. Cookie
1. Client (if client variables are enabled)

---

# What goes in the closure context?

Presumably, everything that was visible to the closure function at its creation is available when you later execute it. That includes application scope, and so forth, but the only three you should concern yourself with are:

1. Local scope of the closure-creator
1. Arguments scope of the closure-creator
1. Variables of the component (if in a component)

Any more than that and I would argue that you're misusing closures. Best practices for encapsulation still apply.

---

# Scope Chain Inside a Closure

1. Local scope of the closure
1. Arguments scope of the closure
1. Local scope of the closure-creator
1. Arguments scope of the closure-creator
1. Variables of the component in which the closure was created (if applicable)
1. Variables of the component in which the closure is executing (if applicable)

---

# Confused? Think of it visually

	component {

		variables.some_var = 5;

		function closure_creator( some_var = 4 ) {

			local.some_var = 3;

			return function( some_var = 2 ){

				local.some_var = 1;

				return some_var;

			}
		}
	}

---

# Can You Nest Closures?

Yes. It is technically possible and works as you would expect. The inner closure evaluates its closure context as part of the Scope Chain. Part of that closure context is the closure context of the wrapping closure, and so forth, until you've reached outside all nested closures.

<img src="inception-wallpaper.jpg" />

---

# Why did we invent them?

Closures were invented in concept only (as far as I can tell) in 1964, and then in 1975 were implemented as part of the Scheme programming language. If you've read much of the history of JavaScript, Scheme probably sounds familiar: JavaScript's key principles come from Scheme and Self.

Their earliest uses were in functional programming ("continuation-passing style", **a.k.a. callbacks**), to enable "information hiding" **a.k.a. private variables** where there really was no concept of public vs. private variables in a class.

Callbacks also engender asynchronous control-flow thinking for developers, even if the program flow is not asynchronous -- which in turn encourages better separation of control and encapsulation.

---

# How can I use Closures in my CFML?

## ColdFusion 10 Closure Functions

* ArrayEach, StructEach
* ArrayFilter, StructFilter, ListFilter
* ArrayFindAll

---

# How can I use Closures in my CFML?

## ArrayEach

	arrayEach([1,2,3,4,5,6,7], function(item){
		writeOutput( dayOfWeekAsString(item); )
	});

Prints:

	Sunday
	Monday
	Tuesday
	Wednesday
	Thursday
	Friday
	Saturday

---

# How can I use Closures in my CFML?

## StructEach

	structEach({ one:1, two:2 }, function(key, value){
		writeOutput( key & ": " & value );
	});

Prints:

	ONE: 1
	TWO: 2

---

# How can I use Closures in my CFML?

## ArrayFilter

	filtered = ArrayFilter([0,1,2,3,4,5,6,7,8,9,10,11,12], function(item){
		return (item % 2 == 0 && item % 3 == 0);
	});
	writeOutput( arrayToList( filtered ) );

Prints:

	0,6,12

---

# How can I use Closures in my CFML?

## ArrayFindAll

	data = [{ foo: 5 },{ foo: 15 }];

	filtered = ArrayFindAll(data, function(item){
		return item.foo < 10;
	});

Resulting Array:

	[{ foo: 5 }]

---

# How can I use Closures in my CFML?

## ColdFusion 11 Closure Functions

I asked if I could sneak-peek the new closure functions that are in CF11, but was told I could not. :(

## Functional Programming Libraries

* Sesame (Mark Mandel)
    * _groupBy
    * _unique
    * _queryEach
    * _times
    * _eachParallel
* Underscore.cfc (Russ ???)
    * Mostly-complete port of all of underscore.js

---

# How can I use Closures in my JavaScript?

---

<h2 style="text-align:center">Fín</h2>
<p align="center">github.com/atuttle/preso-closures</p>