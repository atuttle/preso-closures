# Open Yourself To Closures

@AdamTuttle<br/>
fusiongrokker.com



<<<<<<< HEAD
# Shameless Plug

## Extra-Life

[tinyurl.com/donate-adam](http://tinyurl.com/donate-adam)



=======
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc
# What is a Closure?

## More than just a function

Note:
A closure is a special type of function. It's a superset of normal functions. A Function _and_...


<<<<<<< HEAD
A closure is a function-object

<span class="highlight">-- a function that can be passed as an<br/>argument to another function --</span>

that always remembers its context.



# Why were they invented?

* Invented in 1964 for Lambda Calculus (LISP) - Alonzo Church
=======
A closure is a function-object -- a function that can be passed as an argument to another function -- that always remembers its context.



# Why did we invent them?

* Invented in 1964 (concept only?)
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc
* Implemented in Scheme, 1975
* Functional programming: <span class="highlight">Continuation-passing style</span>
* Has since become the mark of asynchronous programming

Note:
<<<<<<< HEAD
- Scheme familiar? Scheme + Self = JS
- C.P.S. = Callbacks
- Closures solve problems in other languages: e.g. JS has no private class variables
- FP becomes popular
- People want FP = Closures in CFML



# Context, you say?

Note:
- Remember: Closure = Function + Context
- Context? Look at ColdFusion Scopes.
=======

* Scheme familiar? JS partly based on it.
* CPS aka Callbacks



# Context?
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc


# Context: Scopes
	function callMe( name ) {
		return "Hi, #name#.";
	}

	callMe( 'maybe' );

When we run this code, what is returned?

	Hi, maybe.

Why?

<<<<<<< HEAD
=======
Note:
What do I mean by context? To answer that, let's look at ColdFusion Scopes.

>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc

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
<<<<<<< HEAD
Application, Session, Request, Server = **always** be explicitly referenced

- 2007: [**Scope Nazi**](http://fusiongrokker.com/post/scope-nazi) ~ bugs caused by not understanding scope chain. I had _no idea how right I was_.
- 2009: [**Scope Priority Changes in ColdFusion 9**](http://fusiongrokker.com/post/scope-priority-changes-in-coldfusion-9).
=======
Application, Session, Request, and Server scopes are available but must **always** be explicitly referenced.

  * 2007: [**Scope Nazi**](http://fusiongrokker.com/post/scope-nazi) ~ bugs caused by not understanding scope chain. I had _no idea how right I was_.

  * 2009: [**Scope Priority Changes in ColdFusion 9**](http://fusiongrokker.com/post/scope-priority-changes-in-coldfusion-9).

  * I could probably do an entire hour presentation on the scope chain alone, but that's not what you're here for. The important thing to remember is that the scope chain is a crucial topic for developers to understand.
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc



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

<<<<<<< HEAD
Note:
- Railo refuses to fix this because ACF local scope includes arguments scope
=======
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc

## So far, no closures

So how do closures work, anyway?

<<<<<<< HEAD
Note:
- Everything so far was just to demonstrate scope chain

=======
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc


## Scope Chain + Closures

<<<<<<< HEAD
Simply put: Closures _<span class="highlight">add a new scope to the scope chain</span>_.<br/>
=======
Simply put: Closures _<span class="highlight">add a new scope to the scope chain</span>_.<br/><br/>
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc

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

<<<<<<< HEAD
<span style="color:dodgerblue"><strong>Why no Query?</strong></span>

Note:
- ANSWER: Because there's no way to do a query-based output loop (cfoutput, cfloop) inside a closure

=======
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc

# What goes in the closure context?

TL;DR: Everything, but...

Note:
<<<<<<< HEAD
- Just because you can doesn't mean you should.
=======

Just because you can doesn't mean you should. Everything visible during creation is available at execution.

Best practice for closures: Only use:

1. closure-Local
1. closure-arguments
1. creator-local
1. creator-arguments

If you need more, pass it in!
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc


# Closure Scope Chain

1. Closure-Local
1. Closure-Arguments
1. Creator-Local
1. Creator-Arguments
1. Creator-Component-Variables

<<<<<<< HEAD
Note:
- If you need more, pass it in as arguments!

=======
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc

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
<<<<<<< HEAD
It's really, really easy to remember the order
=======
Confused? Just remember a component with a function that returns a closure. Work your way out from the inside.
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc



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


<<<<<<< HEAD
# Nope
=======
# Nope.
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc

`Variable FOO is undefined.`


<<<<<<< HEAD
# This


## Pretty normal for CFML

`this` scope is not in the scope chain.

Wherever you use it, there you are.

Note:
"this" is not in scope chain; refers to current-component everywhere you use it. No need for "that"-hack.


=======

# Nesting Closures
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc

<img src="inception-wallpaper.jpg" />

Note:
<<<<<<< HEAD
Can you nest closures? **Yes.** Works exactly as you probably expect.

The inner closure evaluates its closure context as part of the Scope Chain. Part of that closure context is the closure context of the wrapping closure, and so forth, until you've reached outside all nested closures.
=======
It is possible and works as you would expect. The inner closure evaluates its closure context as part of the Scope Chain. Part of that closure context is the closure context of the wrapping closure, and so forth, until you've reached outside all nested closures.
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc



# How can I use Closures in my CFML?

<<<<<<< HEAD

=======
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc
## ColdFusion 10 Closure Functions

* ArrayEach, StructEach
* ArrayFilter, StructFilter, ListFilter
<<<<<<< HEAD
* ArrayFindAll\*, ArrayFindAllNoCase\*
=======
* ArrayFindAll
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc


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
<<<<<<< HEAD

		writeOutput( key & ": " & value );

=======
		writeOutput( key & ": " & value );
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc
	});

Prints:

	ONE: 1
	TWO: 2

<<<<<<< HEAD
Note:
- "...EACH" functions give you the opportunity to use every item in a collection

=======
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc

## Array Filter

	filtered = ArrayFilter([0,1,2,3,4,5,6,7,8,9,10,11,12], function(item){
<<<<<<< HEAD

		return (item % 2 == 0 && item % 3 == 0);

=======
		return (item % 2 == 0 && item % 3 == 0);
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc
	});
	writeOutput( arrayToList( filtered ) );

Prints:

	0,6,12

<<<<<<< HEAD
Note:
- Instead of using each item, now we filter based on the result of a truth-test callback

=======
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc

## Array Find All

	data = [{ foo: 5 },{ foo: 15 }];

	filtered = ArrayFindAll(data, function(item){
		return item.foo < 10;
	});

Resulting Array:

	[{ foo: 5 }]

<<<<<<< HEAD
Note:
- In this form, same as ArrayFilter: truth test callback
- ArrayFindAll & ArrayFindAllNoCase also take a string as 2nd argument (undocumented) -- this is where "no case" applies


## None of these <span class="highlight">require</span> closures

Note:
FP is about writing utilities that delegate part of complex processes. Often that delegated part is decision making.



## ColdFusion 11 FP Additions

=======

## ColdFusion 11 Closure Functions
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc

I'm not allowed to sneak any CF11 features &nbsp; :o(

<p><br/></p>

<<<<<<< HEAD
ColdFusion Bug [#3595198](https://bugbase.adobe.com/index.cfm?event=bug&id=3595198) Requested Map and Reduce
=======
Bug [#3595198](https://bugbase.adobe.com/index.cfm?event=bug&id=3595198) Requested Map and Reduce
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc

for Arrays, Structures, Queries, and Lists.

<br/>Current Status: "ToTest," Reason: "Fixed."

Note:
<<<<<<< HEAD
Those are the lines. Reading between them is left as an exercise for the viewer.


# This was my<br/>proposed syntax


## Map

	updatedArray = arrayMap([1,2,3], function(elementValue, index, originalArray){

		return elementValue * index;

	});

## Reduce

	sumColB = queryReduce(foo, function(memo = 0, row, originalQuery){

		return memo + row.B;

	});



## Open Source<br/>Functional Programming<br/>Libraries for CFML
=======
Those are the lines. Reading between them is up to you.


## Functional Programming<br/>Libraries for CFML
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc


## Sesame

By Mark Mandel

* groupBy
* unique
* queryEach
* times
* eachParallel
* ... and much, much more.


## Underscore.cfc

<<<<<<< HEAD
By Russ Spivey

Mostly-complete port of underscore.js

* max
* find
* reduce
* reduceRight
* zip
* once
* ... and much, much more.



## Closures<br/>in JavaScript


## Do Much jQuery?

	function doStuff(){

		var theId = "#foo";

		$.get('http://www.google.com', function(data){
			$( theId ).html( data );
		});

	}


## Same thing

	function doStuff(){

		var theId = "#foo";

		$.get('http://www.google.com', function(data){
			$( theId ).html( data );
		});

	}

<p></p>

	function doStuff(){

		var theId = "#foo";

		var callback = function(data){ $(theId).html( data ); };

		$.get('http://www.google.com', callback);

	}
=======
By Russ ???

* Mostly-complete port of underscore.js



# Using Closures in JavaScript
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc


## Underscore.js

<<<<<<< HEAD
Prefix array items:

	function getGreetings( people ){
		var prefix = 'hi, ';

		var greetings = _.map(people, function( item ){
			return prefix + item;
		});

		return greetings;
	}

	console.log( getGreetings( ['mom','dad'] ) );

=&gt; `[ 'hi, mom', 'hi, dad' ]`

Note:
Community has standardized on underscore.js. Most depended on module in NPM, by almost 1k (25%).


## Functional Programming:<br/>More than just closures


### Reduce:
=======
Filter:

	var evens = _.filter([0,1,2,3,4,5,6], function( item ){

		return item % 2 === 0;

	});

	console.log( evens );

=&gt; `[0, 2, 4, 6]`


## Underscore.js

Reduce:
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc

	var sum = _.reduce([1,2,3,4,5,6], function( memo, num ){

		return memo + num;

	}, 0);

	console.log( sum );

=&gt; `21`

<<<<<<< HEAD
Note:
Notice I'm still using underscore.js, but not referencing any external variables.

FP is about writing utilities that delegate some portion of complex processes.


### Debounce:
=======

## Underscore.js

Debounce:
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc

	var scrollListener = _.debounce(function(){

		console.log( document.body.scrollTop );

	}, 250);

	$( window ).on( 'scroll', scrollListener );

Note:
Scroll is a noisy event. Only run when it's been &gt;= 250ms since the last run.



# Questions?



# Thank You

<<<<<<< HEAD
[github.com/atuttle/preso-closures](https://github.com/atuttle/preso-closures)

Extra-Life: [tinyurl.com/donate-adam](http://tinyurl.com/donate-adam)
=======
github.com/atuttle/preso-closures
>>>>>>> c75ca36c412b4ee90f4efa80e8204f1bc5cdb4dc
