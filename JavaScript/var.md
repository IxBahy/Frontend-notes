# The Var keyword

var and let might do almost the same job
but under the hood var is a different bread

Variables declared with var, are either function-scoped or global-scoped. 
They are visible through blocks => no block scope. 
and they are always hoisted and tolerate redeclaration ' That's the difference in a nutshell '

- a block scope means the scope in the if conditions and the loops
```js
if (true) {
	var test = true; // if statement is a block and it should have a lexical environment 'modern JS feature'
	//But the var break this rule
}

console.log(test); // true, the variable lives after if. it should be undefined
```

note: If a code block is inside a function, then var becomes a function-level variable:
- it's hoisted but what does it really mean

hoisted => raised
so for example if we will assign a value to a variable that we declared with var at the first line of our code
but the declaration is far down this normally should cause an error but it won't
So this code:

```js

function anything() {

    variable = "Hello";// this should give an error => refrence error cant access before init but it works

    console.log(variable);

    var variable;

}

anything();

```

…Is technically the same as this (moved var phrase above):

```js
function anything() {
	var phrase;

	phrase = "Hello";

	console.log(phrase);
}
anything();
```

note: declaration is hoisted but the assignment is not
so if we declare a variable and assigned it below but referenced it above it will run because the declaration is raised
but it won't be assigned any value so it will run but the variable will equal null

-With var, we can redeclare a variable any number of times. If we use var with an already-declared variable, it’s just ignored:

```js
    var user = "Pete";

    var user = "John"; // this "var" does nothing (already declared) // ...it doesn't trigger an error

    console.log(user); // John
```

## So how did they manage to work around this in the old days ' IIFE '

first before even saying how they managed to avoid the var
why did they really want to avoid it?
what did it cause

### Global namespace pollution

- first we have to talk about something you didn't even know before
    JAVASCRIPT HAS A GARBAGE COLLECTOR
tho it might seem obvious but probably not a lot of people read about it
why?
because memory management in JavaScript is performed automatically and invisibly to us
have you ever asked
What happens when something is not needed anymore? How does the JavaScript engine discover it and clean it up?

**reachability** In a nutshell it means that all the variables we can currently reach and use will be put in the memory the others are discarded

values:
- root values:
    -The currently executing function, its local variables and parameters.
    -Other functions on the current chain of nested calls 'event loop', their local variables and parameters.
    -Global variables.
    -(there are some other, internal ones as well)

- reachable values:
Any other value is considered reachable if it’s reachable from a root by a reference or by a chain of references.

-unreachable:
removed from memory by the garbage collector

**Example**

```js
let object = {
	value: "hi", //now the object is in the stack and it references the value stored in the heap
};
//this now is a reachable value (some address in the stack that references the value in the heap )
```

but if we do this in the same code

```js
object = null;
//the address in the stack is now deleted and the value in the heap has nothing that points to it => unreachable
//and here is where our friend Garbage Collector works and junks it to free the memory
```

if the references are interlinked more details here: https://javascript.info/garbage-collection#interlinked-objects and you remove the root object that points to all the values the removed values are called **Unreachable island**

**digging deeper into the GC** the internal algorithm:
it's called 'mark-and-sweep' at it works as follows:
- The garbage collector takes the global object as a root and “marks” all the references from it.
- then it considers each mark a new root and marks all the references from it 'will look like a tree'
- And so on until every reachable (from the roots) reference is visited.
- All objects except marked ones are removed.

the JS engine applies some optimizations to this algorithm:
        
- Generational collection: objects are split into two sets “new ones” and “old ones”. In typical code, many objects have ashort life span: they appear, do their job and die fast, so it makes sense to track new objects and clear the memory fromthem if that’s the case. Those that survive for long enough, become “old” and are examined less often.

- Incremental collection: marking the whole object set at once, it may take some time and introduce visible delays in theexecution the engine splits the whole set of existing objects into multiple parts and then clears these parts one afteranother There are many small garbage collections instead of a total one. That requires some extra bookkeeping between them totrack changes, but we get many tiny delays instead of a big one.

- Idle-time collection: the garbage collector tries to run only while the CPU is idle, to reduce the possible effect on theexecution.

#### Back to global namespace pollution

As variables lose scope, they will be eligible for garbage collection. If they are scoped globally, then they will not be eligible for collection until the global namespace loses scope, and trust me you don't want to have a lot of global scope variables.

a code like this one 

```js
for (var i = 0; i < 2003000; i++) {
    var arra = [];
    arr.push(i * i + i);
}
```
will add about 10,000 kb of memory usage that will not be collected and yes we used a var here 
so why is the var wrong? 
you might encounter some cases when you define variables in a block scope 'if statement or a loop' but in the old days it will be scoped in the global namespace 

but if it was like this 

```js
for (let i = 0; i < 2003000; i++) {
    let arra = [];
    arr.push(i * i + i);
}
```
it would've been collected as soon as it exited the for-loop scope 

**Always remember** keeping the values in a closure will ensure they are collected 
don't know what closure is don't worry we will talk about it another time


### So how did they manage to avoid this ' IIFE '    
IIFE => immediately-invoked function expressions
it's not something you should use but you have to know that you can do this 
an IIFE is 
```js
(function(){
    var something='idk'
    console.log(something);
})();   //here it invoked as soon as it was created and we use it in this function context to do all what we need and then 
        //it's removed from the memory by the GC
```
Question 1: why is it wrapped in parenthesis?
- Javascript engine when encountering the keyword "function" understand that this is a declaration and thus it needs a name so our code above will cause an error -> SyntaxError: Function statements require a function name
Question 2: why don't we add a name?
- Javascript doesn't allow the function declaration to be called immediately 

there are other ways to do this in JS it doesn't have to be a parenthesis 

our history lesson now is done :)
