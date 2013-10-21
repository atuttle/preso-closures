# Open Yourself To Closures

@AdamTuttle<br/>
fusiongrokker.com



# Shameless Plug

## Extra-Life

[tinyurl.com/donate-adam](http://tinyurl.com/donate-adam)



# Closure vs Callback

Note:
- Need to clear the air: callbacks !== closures
- Closures are easy after you get callbacks
- Foundation of closures


## jQuery async callback

	$.get(
		'http://api.openweathermap.org/data/2.5/weather?q=Las%20Vegas,%20NV'
		, function(data){
			console.log(data)
		}
	);


## Node.js async callback

	var fs = require('fs');

	fs.readFile('data.txt', function(err, data){
		if (err){
			console.log('there was an error');
			console.error(err);
			return;
		}
		console.log('file read complete');
		console.log(data);
	});


## Callbacks since CF5

	<cfscript>

		function myCallback(){
			return "called back!";
		}

		function do_something_and_call_back( cb ){
			cb();
		}

	</cfscript>

	<cfoutput>#do_something_and_call_back( myCallback )#</cfoutput>

Note:
- When CF added UDFs in CF5, this syntax became possible
- Currently not possible to use a built-in function as a callback (e.g. WriteOutput)
- It's rare to see this in CF code: callbacks are typically a sign of async... CFML is synchronous


## Anonymous Functions since CF10

	ArrayEach([1,2,3], function(item){
		writeOutput( item );
	});



# History of Closures

* Invented in 1964 by Peter J. Landin
* Implemented in Scheme, 1975
* Work well with Functional Programming's <span class="highlight">Continuation-passing style</span>

Note:
- Scheme familiar? Scheme + Self = JS
- C.P.S. = Callbacks
- Closures solve problems in other languages: e.g. JS has no private class variables
- FP becomes popular
- People want FP = Closures in CFML
- Very few problems in CFML require closures



# What is a Closure?

## More than just a function

Note:
A closure is a special type of function. It's a superset of normal functions. A Function _and_...


A closure is a function-object

<span class="highlight">-- a function that can be passed as an<br/>argument to another function --</span>

that always remembers its context.

---

All functions are closures.

Note:
- Context? We'll talk about that in a bit.



# Closure Example


## Pseudocode: Before

	function times10(n){ return n * 10; }

	tens = [1,2,3,4,5].map( times10 );

Result:

	=> tens: [10,20,30,40,50]

Note:
- Pseuedocode because CFML doesn't have syntax like this


## After (with closure)

	function times(multiplier){
		return function(n){
			return multiplier * n;
		};
	}

	tens = [1,2,3,4,5].map( times(10) );
	hundreds = [1,2,3,4,5].map( times(100) );

Result:

	=> tens: [10,20,30,40,50]
	=> hundreds: [100,200,300,400,500]

Note:
- Here we're using closures to get rid of the hard-coded multiplyer of 10


# Another Example


## Create a truth-test

	function matches(str){
		return function(ss){
			return (s == ss);
		};
	}

	['Bender','Goldy','Findi','Oliver'].filter( matches('Bender') );

Result:

	=> ['Bender']



# The Scope Chain

The Scope Chain is the list of all scopes that ColdFusion will check when you make an un-scoped variable reference.

	return name; //uses scope chain

	return arguments.name; //does not use scope chain

Note:
- Socpe Chain is a Personal Pet Peeve of mine
- 2007: [**Scope Nazi**](http://fusiongrokker.com/post/scope-nazi) ~ bugs caused by not understanding scope chain. I had _no idea how right I was_.
- 2009: [**Scope Priority Changes in ColdFusion 9**](http://fusiongrokker.com/post/scope-priority-changes-in-coldfusion-9).


## ColdFusion's Scope Chain

1. Function/Thread Local
1. Attributes (Threads, _NOT_ custom tags)
1. <span class="highlight">+Closure (if applicable)</span>
1. Query
1. Variables
1. CGI
1. CFFile
1. Url
1. Form
1. Cookie
1. Client

<span style="color:dodgerblue"><strong>See anything missing?</strong></span>

Note:
- Application, Session, Request, Server = **always** be explicitly referenced


## Simplest Scope Chain Example

	function callMe( name ) {
		return "Hi, #name#.";
	}

	callMe( 'maybe' );

When we run this code, what is returned?

	Hi, maybe.


# Why?

Note:
- Scope Chain! function's argument name is 2nd in scope chain.


## Complex Scope Chain

	<cffunction name="callMe">
		<cfargument name="name" />
		<cfset local.name = "Uhura" />

		<cfset local.q = queryNew(
			"name", "varchar", [{ name: "Spock" }] ) />

		<cfoutput query="local.q">
			<cfreturn name />
		</cfoutput>
	</cffunction>

	<cfset variables.person = callMe( "Kirk" ) />

What's the value of `variables.person`?


## What's the result?

**Adobe ColdFusion 10+:** <span class="highlight">Kirk</span> (Arguments)<br/>
**Railo:** <span class="highlight">Uhura</span> (Local)

Note:
- Railo refuses to fix this because ACF local scope includes arguments scope



# Confused?
Note: This can be a weird topic to wrap your head around.


![](take-this.jpg)
Note:
- Here's something visual that will make it clear


# Dungeon Map:

	component {
		variables.A = 5;

		function closure_creator( A = 4 ) {
			local.A = 3;

			return function( A = 2 ){
				local.A = 1;
				return A;
			}
		}
	}

Note:
It's really, really easy to remember the order this way



# This
Note:
- Familiar with closures in JS = wondering about `this`


## Pretty normal for CFML

The `this` scope is not in the scope chain.

<br/>Component `this` scope, if any.



![](inception-wallpaper.jpg)

Note:
Can you nest closures? **Yes.** Works exactly as you probably expect.

Just be careful how deep you go. You may never get back out. ;)



## ColdFusion 10 'Closure' Functions

* ArrayEach, StructEach
* ArrayFilter, StructFilter, ListFilter
* ArrayFindAll<span class="highlight">\*</span>, ArrayFindAllNoCase<span class="highlight">\*</span>


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

Note:
- "...EACH" functions give you the opportunity to use every item in a collection


## Array Filter

	filtered = ArrayFilter([0,1,2,3,4,5,6,7], function(item){

		return (item % 2 == 0 && item % 3 == 0);

	});
	writeOutput( arrayToList( filtered ) );

Prints:

	0,6

Note:
- Instead of using each item, now we filter based on the result of a truth-test callback


## Array Find All

	data = [{ foo: 5 },{ foo: 15 }];

	filtered = ArrayFindAll(data, function(item){
		return item.foo < 10;
	});

Resulting Array:

	[{ foo: 5 }]

Note:
- In this form, same as ArrayFilter: truth test callback
- don't be fooled by **Array Find All No Case**!


## Array Find All (Alt)

	data = ["Adam","ADAM","adam","aDaM"];

	filtered = ArrayFindAll(data, "Adam");

	=> ["Adam"]

	filtered = ArrayFindAllNoCase(data, "Adam");

	=> ["Adam","ADAM","adam","aDaM"]

Note:
- Undocumented!
- This is the reason I put an asterisk next to them
- ONLY place that the "no case" part of the name applies - not with callback


## Real Closures

Curry:

	function _curry(func, args){
		var argMap = {};
		var counter = 1;
		ArrayEach(args, function(it) { argMap[counter++] = it; });
		return function(){
			var newArgs = StructCopy(argMap);
			var len = StructCount(argMap);
			StructEach(arguments, function(key, value){
				newArgs[len + key] = value;
			});
			return func(argumentCollection=newArgs);
		};
	}

Note:
- Copied and condensed from Mark Manel'd Sesame project


Using Curry:

	function orig(a,b,c,d){
		return "#a# #b# #c# #d#";
	}

	wrapper = _curry( orig, ['hi', 'there'] );

	wrapper( 'cfsummit', 'attendees' );

Returns:

	"hi there cfsummit attendees"



## ColdFusion 11 Functional Programming Additions

I'm not allowed to sneak any CF11 features &nbsp; :o(

<p><br/></p>

ColdFusion Bug [#3595198](https://bugbase.adobe.com/index.cfm?event=bug&id=3595198) Requested Map and Reduce

for Arrays, Structures, Queries, and Lists.

<br/>Current Status: "ToTest," Reason: "Fixed."

Note:
- Those are the lines. Reading between them is left as an exercise for the viewer.


# This was my<br/>proposed syntax


## Map

	updated = arrayMap([1,2,3], function(elValue, ix, origArray){

		return elValue * ix;

	});

## <br/>Reduce

	sumColB = queryReduce(foo, function(memo = 0, row, origQuery){

		return memo + row.B;

	});



## Open Source<br/>Functional Programming<br/>Libraries for CFML


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


## Underscore.js

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

	var sum = _.reduce([1,2,3,4,5,6], function( memo, num ){

		return memo + num;

	}, 0);

	console.log( sum );

=&gt; `21`

Note:
- Notice I'm still using underscore.js, but not referencing any external variables.

- FP is about writing utilities that delegate some portion of complex processes.


### Debounce:

	var scrollListener = _.debounce(function(){

		console.log( document.body.scrollTop );

	}, 250);

	$( window ).on( 'scroll', scrollListener );

Note:
Scroll is a noisy event. Only run when it's been &gt;= 250ms since the last run.



# Wrapping up


## Closures are just functions


## All functions are closures


## Callbacks != Closures


## Scope your variables


## Remember the Dungeon Map

	component {
		variables.A = 5;

		function closure_creator( A = 4 ) {
			local.A = 3;

			return function( A = 2 ){
				local.A = 1;
				return A;
			}
		}
	}



# Questions?



# Thank You

[github.com/atuttle/preso-closures](https://github.com/atuttle/preso-closures)

Extra-Life: [tinyurl.com/donate-adam](http://tinyurl.com/donate-adam)

Please fill out session surveys!
