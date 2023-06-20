# the Var keyword

    var and let might do almost the same job
    but under the hood var is a different bread
    Variables declared with var, are either function-scoped or global-scoped. They are visible through blocks => no block scope. and they are always hoisted and tolerates redeclaration 'that's the difference in a nutshell'

        - a block scope means the scope in the if conditions and the loops

```js
if (true) {
	var test = true; // if statment is a block and it should have a lexical enviroment 'modern JS feature'
	// but the var break this rule
}

console.log(test); // true, the variable lives after if. it should be undefined
```

    note :If a code block is inside a function, then var becomes a function-level variable:

    - it's hoisted but what does it really means

    hoisted => raised

    so for example if we will assign a value to a variable that we declared with var at the first line of our code
    but the declaration is far down this normally should cause an error but it wont

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

    note: declaration is hoisted but assignment is not

    so if we declare a variable and assignet it below but refrenced it above it will run because of the declaration is raised but it wont be assigned any value so it will run but the variable will equal null


    -With var, we can redeclare a variable any number of times. If we use var with an already-declared variable, it’s just ignored:

    ```js
    var user = "Pete";

    var user = "John"; // this "var" does nothing (already declared) // ...it doesn't trigger an error

    console.log(user); // John

    ```

## so how did they managed to work around this in the old days ' IIFE '

    first before even saying how they managed to avoid the var
    why did they really wanted to avoid it ?
    what did it cause

### global namespace pollution

    -first we have to talk about something you didn't even know before
        JAVASCRIPT HAS A GARBAGE COLLECTOR
    tho it might seem obvious but probably not a lot of people read about it
    why?
    because memory management in JavaScript is performed automatically and invisibly to us
    have you ever asked
    What happens when something is not needed any more? How does the JavaScript engine discover it and clean it up?

**reachability** in a nutshell it means that all the variables we can currently reach and use will be put in the memory the others are discarded

    values:
        -root values:
            -The currently executing function, its local variables and parameters.
            -Other functions on the current chain of nested calls 'event loop', their local variables and parameters.
            -Global variables.
            -(there are some other, internal ones as well)

        -reachable values:
            Any other value is considered reachable if it’s reachable from a root by a reference or by a chain of references.

        -unreachable:
            removed from memory by the garbage collector

**Example**

```js
let object = {
	value: "hi", //now the object is in the stack and it refrences the value stored in heap
};
//this now is a reachable value (some address in the stack that refrences the value in heap )
```

but if we do this in the same code

```js
object = null;
//the address in the stack is now deleted and the value in the heap has nothing that points to it => unreachable
//and here where our friend Garbage Collector work and junk it to free the memory
```

if the refrences are interlinked more details here : https://javascript.info/garbage-collection#interlinked-objects and you remove the root object that points to all the values the removed values are called **Unreachable island**

**digging deeper into the GC** the internal algorithm:
     it's called 'mark-and-sweep' at it works as following:
        -The garbage collector takes the global object as a root and “marks” all the refrences from it .
        -then it consider each mark a new root and mark all the refrences from it 'will look like a tree'
        -And so on until every reachable (from the roots) references are visited.
        -All objects except marked ones are removed.

    the JS engine apply some optimizations on this algorithm:
        
        -Generational collection: objects are split into two sets “new ones” and “old ones”. In typical code, many objects have a short life span: they appear, do their job and die fast, so it makes sense to track new objects and clear the memory from them if that’s the case. Those that survive for long enough, become “old” and are examined less often.

        -Incremental collection: marking the whole object set at once, it may take some time and introduce visible delays in the execution the engine splits the whole set of existing objects into multiple parts and then clear these parts one after another There are many small garbage collections instead of a total one. That requires some extra bookkeeping between them to track changes, but we get many tiny delays instead of a big one.
        
        -Idle-time collection: the garbage collector tries to run only while the CPU is idle, to reduce the possible effect on the execution.

#### back to global namespace pollution

    As variables lose scope, they will be eligible for garbage collection. If they are scoped globally, then they will not be eligible for collection until the global namespace loses scope and trust me you dont want to have alot of global scope variables.

    a code like this one 
```js
for (var i = 0; i < 2003000; i++) {
    var arra = [];
    arra.push(i * i + i);
}
```
    will add about 10,000 kb of memory usage that will not be collected and yes we used a var here 

    so why is the var bad? 
    you might encounter some cases when you define variables in a block scope 'if statment or a loop' but in the old days it will be scopped in the globale namespace 

but if it was like this 

```js
for (let i = 0; i < 2003000; i++) {
    let arra = [];
    arra.push(i * i + i);
}
```
it would've been collected as soon as it exit the for loop scope 

**always remeber** keeping the values in a clouser will ensure they are collected 
    dont know what a clouser is don't worry we will talk about in another time


### so how did they manage to avoid this ' IIFE '
    
    IIFE => immediately-invoked function expressions
    it's not something you should use but you have to know that you can do this 
    an IIFE is 
```js
(function(){
    var something='idk'
    console.log(something);
})();   //here its invoked as soon as it was created and we use it in this function context to do all what we need and then it's
        //removed from the memory by the GC
```
Question 1: whey is it wrapped in parenthesis ?
    javascript engine when encounter the keyword "function" it understand that this is a declaration and thus it needs a name so our code above will cause an error -> SyntaxError: Function statements require a function name
Question 2: why don't we add a name ?
    javascript doesn't allow the function declaration to be called immediatly 

there are other ways to do this in JS it doesn't have to be parenthesis 

our history lesson now is done :)