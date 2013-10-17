# Open Yourself To Closures

@AdamTuttle<br/>
fusiongrokker.com



# Shameless Plug

## Extra-Life

[tinyurl.com/donate-adam](http://tinyurl.com/donate-adam)



# What is a Closure?

## More than just a function

Note:
A closure is a special type of function. It's a superset of normal functions. A Function _and_...


A closure is a function-object -- a function that can be passed as an argument to another function -- that always remembers its context.



# Why did we invent them?

* Invented in 1964 (concept only?)
* Implemented in Scheme, 1975
* Functional programming: <span class="highlight">Continuation-passing style</span>
* Has since become the mark of asynchronous programming

Note:

* Scheme familiar? JS partly based on it.
* CPS aka Callbacks



# Context?


# Context: Scopes
	function callMe( name ) {
		return "Hi, #name#.";
	}

	callMe( 'maybe' );

When we run this code, what is returned?

	Hi, maybe.

Why?

Note:
What do I mean by context? To answer that, let's look at ColdFusion Scopes.


# The Scope Chain

The Scope Chain is the list of all scopes that ColdFusion will check when you make an un-scoped variable reference.

	return name; //uses scope chain

	return arguments.name; //does not use scope chain


# CF Scope Chain

1. Function/Thread Local
1. Attributes (Threads, _NOT_ custom tags)
1. Query
1. Variables
1. CGI
1. CFFile
1. Url
1. Form
1. Cookie
1. Client

<span style="color:dodgerblue"><strong>What's Missing?</strong></span>

Note:
Application, Session, Request, and Server scopes are available but must **always** be explicitly referenced.

  * 2007: [**Scope Nazi**](http://fusiongrokker.com/post/scope-nazi) ~ bugs caused by not understanding scope chain. I had _no idea how right I was_.

  * 2009: [**Scope Priority Changes in ColdFusion 9**](http://fusiongrokker.com/post/scope-priority-changes-in-coldfusion-9).

  * I could probably do an entire hour presentation on the scope chain alone, but that's not what you're here for. The important thing to remember is that the scope chain is a crucial topic for developers to understand.



# Complex example

	<cffunction name="callMe">
		<cfargument name="name" />

		<cfset local.name = "Uhura" />

		<cfset local.q = queryNew("name", "varchar", { name: "Spock" }) />

		<cfoutput query="local.q">
			<cfreturn name />
		</cfoutput>
	</cffunction>

	<cfset variables.person = callMe( "Kirk" ) />

What's the value of `variables.person`?

To answer that, we search the scope chain.


## What's the result?

	<cffunction name="callMe">
		<cfargument name="name" />

		<cfset local.name = "Uhura" />

		<cfset local.q = queryNew("name", "varchar", { name: "Spock" }) />

		<cfoutput query="local.q">
			<cfreturn name />
		</cfoutput>
	</cffunction>

	<cfset variables.person = callMe( "Kirk" ) />

**On Adobe ColdFusion 10+:** <span class="highlight">Kirk</span> (arguments)<br/>
**On Railo:** <span class="highlight">Uhura</span> (Local)


## So far, no closures

So how do closures work, anyway?



## Scope Chain + Closures

Simply put: Closures _<span class="highlight">add a new scope to the scope chain</span>_.<br/><br/>

1. Function/Thread Local
1. Attributes (Threads, _NOT_ custom tags)
1. **<span class="highlight">CLOSURE</span>**
1. <del>Query</del>
1. Variables
1. CGI
1. CFFile
1. Url
1. Form
1. Cookie
1. Client


# What goes in the closure context?

TL;DR: Everything, but...

Note:

Just because you can doesn't mean you should. Everything visible during creation is available at execution.

Best practice for closures: Only use:

1. closure-Local
1. closure-arguments
1. creator-local
1. creator-arguments

If you need more, pass it in!


# Closure Scope Chain

1. Closure-Local
1. Closure-Arguments
1. Creator-Local
1. Creator-Arguments
1. Creator-Component-Variables


![](take-this.jpg)


# Dungeon Map:

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

Note:
Confused? Just remember a component with a function that returns a closure. Work your way out from the inside.



# No Access to Executor Scopes


## Easy Mistake

obj.cfc:

	component {
		function getClosure(){
			return function(){
				return foo;
			};
		}
	}

foo.cfm:

	variables.foo = 42;
	variables.obj = new obj();
	variables.closureFn = obj.getClosure();
	writeOutput( variables.closureFn() );

Does this output `42`?


# Nope

`Variable FOO is undefined.`


# This


## Pretty normal for CFML

`this` scope is not in the scope chain.

Wherever you use it, there you are.

Note:
"this" is not in scope chain; refers to current-component everywhere you use it. No need for "that"-hack.



<img src="inception-wallpaper.jpg" />

Note:
Can you nest closures? **Yes.** Works exactly as you probably expect.

The inner closure evaluates its closure context as part of the Scope Chain. Part of that closure context is the closure context of the wrapping closure, and so forth, until you've reached outside all nested closures.



# How can I use Closures in my CFML?

## ColdFusion 10 Closure Functions

* ArrayEach, StructEach
* ArrayFilter, StructFilter, ListFilter
* ArrayFindAll


## Array Each

	arrayEach( [1,2,3,4,5,6,7], function( item ){

		writeOutput( dayOfWeekAsString( item ) & "<br/>" );

	});

Prints:

	Sunday
	Monday
	Tuesday
	Wednesday
	Thursday
	Friday
	Saturday


## Struct Each

	structEach({ one:1, two:2 }, function(key, value){
		writeOutput( key & ": " & value );
	});

Prints:

	ONE: 1
	TWO: 2


## Array Filter

	filtered = ArrayFilter([0,1,2,3,4,5,6,7,8,9,10,11,12], function(item){
		return (item % 2 == 0 && item % 3 == 0);
	});
	writeOutput( arrayToList( filtered ) );

Prints:

	0,6,12


## Array Find All

	data = [{ foo: 5 },{ foo: 15 }];

	filtered = ArrayFindAll(data, function(item){
		return item.foo < 10;
	});

Resulting Array:

	[{ foo: 5 }]


## ColdFusion 11 Closure Functions

I'm not allowed to sneak any CF11 features &nbsp; :o(

<p><br/></p>

Bug [#3595198](https://bugbase.adobe.com/index.cfm?event=bug&id=3595198) Requested Map and Reduce

for Arrays, Structures, Queries, and Lists.

<br/>Current Status: "ToTest," Reason: "Fixed."

Note:
Those are the lines. Reading between them is up to you.



## Functional Programming<br/>Libraries for CFML


## Sesame

By Mark Mandel

* groupBy
* unique
* queryEach
* times
* eachParallel
* ... and much, much more.


## Underscore.cfc

By Russ Spivey

* Mostly-complete port of underscore.js



# Using Closures in JavaScript


## Do Much jQuery?

	function doStuff(){

		var theId = "#foo";

		$.get('http://www.google.com', function(data){

			$(theId).html( data );

		});
	}


## Underscore.js

Filter:

	var evens = _.filter([0,1,2,3,4,5,6], function( item ){

		return item % 2 === 0;

	});

	console.log( evens );

=&gt; `[0, 2, 4, 6]`


## Underscore.js

Reduce:

	var sum = _.reduce([1,2,3,4,5,6], function( memo, num ){

		return memo + num;

	}, 0);

	console.log( sum );

=&gt; `21`


## Underscore.js

Debounce:

	var scrollListener = _.debounce(function(){

		console.log( document.body.scrollTop );

	}, 250);

	$( window ).on( 'scroll', scrollListener );

Note:
Scroll is a noisy event. Only run when it's been &gt;= 250ms since the last run.



# Questions?



# Thank You

[github.com/atuttle/preso-closures](https://github.com/atuttle/preso-closures)

Extra-Life: [tinyurl.com/donate-adam](http://tinyurl.com/donate-adam)
