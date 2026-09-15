##### Contains Functions , this, Closures, Decorators, Generators and Scheduling

---
### Functions
Functions are the main building blocks of the program. They allow the code to be called many times without repetition.
### Function Declaration
To create a function we can use a _function declaration_.
```javascript
function showMessage() {
  alert( 'Hello everyone!' );
}
showMessage();
showMessage();

```
- A variable declared inside a function is only visible inside that function.
- A function can access an outer variable as well
```javascript
let userName = 'John';
function showMessage() {
  userName = "Bob";        // (1) changed the outer variable
   let message = 'Hello, ' + userName;
  alert(message);
}
alert( userName ); // John before the function call
showMessage();
alert( userName ); // Bob, the value was modified by the function
```
## Parameters
- A parameter is the variable listed inside the parentheses in the function declaration (it’s a declaration time term).
- An argument is the value that is passed to the function when it is called (it’s a call time term).
```javascript
function showMessage(from, text) { // parameters: from, text
  alert(from + ': ' + text);
}
showMessage('Ann', 'Hello!'); // Ann: Hello! (*)
showMessage('Ann', "What's up?"); // Ann: What's up? (**)
```
## Returning a value
- A function with an empty `return` or without it returns `undefined`
- Never add a newline between `return` and the value
```javascript
function sum(a, b) {
  return a + b;
}
let result = sum(1, 2);
alert( result ); // 3

function checkAge(age) {
  if (age >= 18) {
	return true;
} else {
	return confirm('Do you have permission from your parents?');
  }
}
let age = prompt('How old are you?', 18);
if ( checkAge(age) ) {
   alert( 'Access granted' );
} else {
  alert( 'Access denied' );
}
```
## Functions == Comments
Functions should be short and do exactly one thing. If that thing is big, maybe it’s worth it to split the function into a few smaller functions. Sometimes following this rule may not be that easy, but it’s definitely a good thing.

The first variant uses a label
```javascript
function showPrimes(n) {
  nextPrime: for (let i = 2; i < n; i++) {

    for (let j = 2; j < i; j++) {
      if (i % j == 0) continue nextPrime;
    }

    alert( i ); // a prime
  }
}
```

The second variant uses an additional function `isPrime(n)` to test for primality:
```javascript
function showPrimes(n) {

  for (let i = 2; i < n; i++) {
    if (!isPrime(i)) continue;

    alert(i);  // a prime
  }
}
function isPrime(n) {
  for (let i = 2; i < n; i++) {
    if ( n % i == 0) return false;
  }
  return true;
}
```
## Function expressions
It allows us to create a new function in the middle of any expression.
```javascript
let sayHi = function() {
  alert( "Hello" );
};
```
### Function is a value
We can copy a function to another variable:
Please note again: there are no parentheses after `sayHi`. If there were, then `func = sayHi()` would write _the result of the call_ `sayHi()` into `func`, not _the function_ `sayHi` itself.
```javascript
function sayHi() {   // (1) create
  alert( "Hello" );
}

let func = sayHi;    // (2) copy
func(); // Hello     // (3) run the copy (it works)!
sayHi(); // Hello    //     this still works too (why wouldn't it)
```
**A Function Declaration can be called earlier than it is defined.**
If it were a Function Expression, then it wouldn’t work:
```javascript
sayHi("John"); // Hello, John
function sayHi(name) {
  alert( `Hello, ${name}` );
}


sayHi("John"); // error!
let sayHi = function(name) {  // (*) no magic any more
   alert( `Hello, ${name}` );
};
```
Think of JavaScript as reading your code in **two phases**:
1. **Preparation phase (before running the code)**
2. **Execution phase (running the code line by line)**
### Preparation phase
JavaScript only knows that
```
let sayHello;
```
exists.
It **does NOT create the function yet**.
Memory:
sayHello → uninitialized
No function is stored.
### Execution phase
First line:
```
sayHello();
```
JavaScript asks:

> "What's inside `sayHello`?"
Nothing yet.
The function hasn't been created.
So you get an error

---
## --> Arrow functions
There’s another very simple and concise syntax for creating functions, that’s often better than Function Expressions.
It’s called arrow functions, because it looks like this:
```javascript
let func = (arg1, arg2, ..., argN) => expression;
```
`(a, b) => a + b` means a function that accepts two arguments named `a` and `b`. Upon the execution, it evaluates the expression `a + b` and returns the result.
```javascript
let sum = (a, b) => a + b;
/* This arrow function is a shorter form of:
let sum = function(a, b) {
  return a + b;
};
*/
alert( sum(1, 2) ); // 3

let sayHi = () => alert("Hello!");  // no arguments
sayHi();
```
 
 ```javascript
 let sum = (a, b) => {  // the curly brace opens a multiline function
    let result = a + b;
    return result; // if we use curly braces, then we need an explicit "return"
};
alert( sum(1, 2) ); // 3
 ```

---
## -->`this`

The `this` keyword (and the real arrow-function difference)
`this` is a special keyword that refers to the **context** the function is running in — "who is calling me / what object do I belong to." `this` is like that: its value depends on _how and where_ the function is called.
```js
function greet() {
  console.log(this.name);
}
const user1 = { name: "Aryan", greet };
const user2 = { name: "Ashu", greet };

user1.greet(); // "Aryan"
user2.greet(); // "Ashu"
```
Arrow functions **don't have their own `this` at all**. They don't participate in that whole "who called me" mechanism. Instead, they just grab `this` from the surrounding code where they were written (lexical scope) — like a variable lookup, not a function-call decision.
```js
const user = {
  name: "Aryan",
  regularGreet: function() {
    console.log(this.name); // "Aryan" — this = user (caller)
  },
  arrowGreet: () => {
    console.log(this.name); // undefined — this = outer scope, NOT user
  }
};

user.regularGreet(); // "Aryan"
user.arrowGreet();   // undefined (this = whatever `this` is outside the object, e.g. mo
```
The arrow function ignores the who called me question entirely and just says "I'll use whatever `this` was already in scope when I was defined."

---
## Higher-order functions
A **higher-order function** is simply: _a function that takes a function as an argument, or returns a function_ (or both).

```js
function makeTea(typeOfTea) {
  return `make tea ${typeOfTea}`;
}

function processTeaOrder(teaFunction) {     // takes a FUNCTION as a parameter
  return teaFunction("Earl Grey");          // calls it with an argument
}

let order = processTeaOrder(makeTea);       // pass makeTea itself (NO                                                                 parentheses!)
console.log(order);                         // "make tea Earl Grey"



function createTeaMaker() {
  return function (teaType) {        // returns a brand-new function
    return `making ${teaType}`;
  };
}

const teaMaker = createTeaMaker();   // teaMaker now HOLDS a function
let result = teaMaker("green tea");  // call the returned function
console.log(result);                 // "making green tea"
```

### Execution context — every function call gets its own workspace
**when you _call_ a function, JavaScript builds a temporary private room for that call to work in.** That room is the execution context. When the function finishes (returns), the room is torn down and thrown away.
Why does it need a private room? Because a function has its own variables and parameters, and they shouldn't leak out or collide with anyone else's. The room keeps them separate.
```js
function makeTea(type) {
  let message = "making " + type;
  return message;
}
makeTea("chai");
makeTea("green tea");
```
When `makeTea("chai")` is called:
1. JS builds a room for _this call_.
2. Inside the room: `type` = `"chai"`, and `message` = `"making chai"`.
3. It returns `"making chai"`.
4. The room is **destroyed** — `type` and `message` are gone.

Then `makeTea("green tea")` is called and JS builds a **brand-new, separate room** — `type` = `"green tea"` this time. The two calls never interfere because each got its own room. That's the whole point.
**The key takeaway:** _call = open a room; return = destroy the room.
Variables inside a function live only inside that room, only for the duration of that call. This is just block scope  applied to function calls.

## -->Closure — but sometimes a piece of the room survives
**Closure is the one situation where part of a room is kept alive.**
It happens when a function _returns another function_ that was using the outer room's variables.

```js
function createCounter() {
  let count = 0;                  // a variable in the OUTER room

  return function () {            // returns an INNER function
    count++;
    return count;
  };
}
const counter = createCounter();  // outer function runs and FINISHES here
```
- `createCounter()` is called → a room opens, `count = 0` is created inside it.
- It **returns the inner function** and finishes.
- By our Part 1 rule, the room should now be destroyed, so `count` should be gone…
- **But it isn't.** Watch:
```js
console.log(counter());  // 1
console.log(counter());  // 2
console.log(counter());  // 3
```

```js
function createTeaMaker(name) {     // `name` lives in this room
  return function (teaType) {
    return `${name} is making ${teaType}`;   // inner function uses `name`
  };
}
const aryanMaker = createTeaMaker("Aryan");  // outer runs, finishes
console.log(aryanMaker("chai"));   // "Aryan is making chai"
console.log(aryanMaker("green tea")); // "Aryan is making green tea"
```
`createTeaMaker` finished after the first line, but `aryanMaker` _still knows_ `name` is `"Aryan"` — because the inner function closed over `name`, kept it in its backpack, and uses it on every call. That's a closure.

---
---
### Decorators and methods

We already know `this` is decided by "what's before the dot." Sometimes we want to **override** that — force a function to run with a specific `this`, regardless of how it's normally called. `call`, `apply`, and `bind` are three tools for exactly that.

#### `call` — run immediately, args listed one by one
```js
function greet(greeting) {
  alert(greeting + ", " + this.name);
}

let user = { name: "Ashu" };
greet.call(user, "Hello"); // "Hello, Ashu"
```
`func.call(context, arg1, arg2, ...)` — runs `func` **right now**, with `this` forced to `context`, and the rest of the arguments passed normally.

#### `apply` —  args passed as an array instead
```js
greet.apply(user, ["Hello"]); // same result: "Hello, Ashu"
```
`func.apply(context, argsArray)` — identical to `call`, except arguments come bundled as an array/array-like.
```js
func.call(context, ...argsArray);  // these two lines
func.apply(context, argsArray);    // do the same thing
```

#### `bind` — doesn't run anything, gives you back a NEW function with `this` locked forever
```js
let boundGreet = greet.bind(user);
boundGreet("Hi"); // "Hi, Ashu" — this is PERMANENTLY user now

let f = boundGreet;
f("Hey"); // still "Hey, Ashu" — can't be un-bound, no matter how you call it later
```


```js
class Button {
  constructor() {
    this.handleClick = this.handleClick.bind(this); // lock "this" before it can get lost
  }
  handleClick() {
    console.log(this);
  }
}
```
This is literally solving the exact losing `this` problem you already fixed with arrow-function class fields — `bind` is the **older** way of solving the same problem. You already know the modern fix (`click = () => {...}`); `bind` is just the pre-arrow-function-class-fields tool for the same job.


**Method borrowing / forwarding**: if you write a generic wrapper function around someone else's function (like a caching layer, a logging layer, a timing layer), the wrapper needs to pass along the original `this` correctly:
```js
function cachingWrapper(func) {
  let cache = new Map();
  return function(x) {
    if (cache.has(x)) return cache.get(x);
    let result = func.call(this, x); // forward "this" correctly to the real function
    cache.set(x, result);
    return result;
  };
}
```
Without `call` here, `func(x)` would run with `this = undefined`, breaking anything inside `func` that relies on `this`

#### "Decorators" — just the name for this wrapping pattern
```js
function cachingDecorator(func) {
  let cache = new Map();
  return function(x) {
    if (cache.has(x)) return cache.get(x);
    let result = func(x);
    cache.set(x, result);
    return result;
  };
}

slow = cachingDecorator(slow); // slow now automatically caches its results
```
Decorator here just means: a function that wraps another function to add behavior (caching, logging, timing) without changing its original code. It's genuinely just a higher-order function

---
---
## -->Scheduling: setTimeout and setInterval

#### The core idea — "do this later, not right now"

Normally, JS runs code **immediately**, line by line
```js
console.log("first");
console.log("second");
```
This prints first then second, instantly, one after another.
But sometimes you want to say: don't run this right now — wait a bit, then run it." That's what `setTimeout` is for.

### -->`setTimeout`
```js
setTimeout(functionToRun, delayInMilliseconds);
```
1. **A function** — the thing you want to eventually run
2. **A number** — how many **milliseconds** to wait before running it (1000ms = 1 second)

```js
function sayHi() {
  alert("Hello");
}

setTimeout(sayHi, 1000); // waits 1 second, THEN runs sayHi
```
This line runs _immediately_ — but all it does is **schedule** `sayHi` to run 1 second from now. 
```js
console.log("A");
setTimeout(() => console.log("B"), 1000);
console.log("C");
// Output: A, C, (wait 1 second), B
```

```js
setTimeout(sayHi, 1000);   // ✓ correct
setTimeout(sayHi(), 1000); // ❌ wrong
```
Because `sayHi()` **with parentheses** means run `sayHi` **right now**.

#### Passing extra info to the function
```js
function sayHi(phrase, who) {
  alert(phrase + ', ' + who);
}
setTimeout(sayHi, 1000, "Hello", "John"); // after 1s: "Hello, John"
```
Anything **after** the delay number gets passed as arguments to your function, once it actually runs.

### -->`clearTimeout`
```js
let timerId = setTimeout(() => alert("never happens"), 1000);
clearTimeout(timerId);
```
`setTimeout` gives you back an ID (just a number). If you change your mind before the time is up, pass that ID to `clearTimeout()`, and the scheduled call is cancelled — it'll never run.

### -->`setInterval`
```js
setInterval(() => alert("tick"), 2000); // alerts every 2 seconds, forever
```
Identical syntax to `setTimeout`, except it keeps firing again and again, every `delay` milliseconds, instead of just once.

```js
let timerId = setInterval(() => alert("tick"), 2000);
setTimeout(() => clearInterval(timerId), 5000); // after 5 sec, stop the repeating
```
Start a repeating "tick" every 2 seconds, but also schedule a **one-time** call after 5 seconds that stops the repetition. 


---
## -->Generators 

Every function you've written so far runs start-to-finish in one go, the moment you call it. A **generator function** is fundamentally different: it can **pause partway through**, hand control back to you, and then **resume exactly where it left off** the next time you ask it to continue.


```js
function* numberGenerator() {
  yield 1;
  yield 2;
  yield 3;
}
```
- **`function*`** (note the asterisk) — marks this as a generator function, not a regular one.
- **`yield`** — like `return`, but instead of ending the function, it **pauses** it right there, handing back a value, and waits to be resumed.


```js
let gen = numberGenerator();
console.log(gen); // NOT 1, 2, 3 — it's a generator object, nothing has run yet!
```
This is the first surprising part: calling `numberGenerator()` does **not** execute any code inside it yet. Instead, it gives you back a special **generator object** — think of it as a **remote control** for the function, not the function's output.

#### `.next()` 
```js
gen.next(); // { value: 1, done: false }
gen.next(); // { value: 2, done: false }
gen.next(); // { value: 3, done: false }
gen.next(); // { value: undefined, done: true }
```
Each call to `.next()` resumes execution from **exactly where it last paused**, runs until it hits the next `yield` (or the function ends), and returns an object with two things:
- **`value`** — whatever was yielded
- **`done`** — `false` if there's more to come, `true` once the function has fully finished

```js
let gen1 = numberGenerator();
let gen2 = numberGenerator();

gen1.next(); // {value: 1, done: false}
gen1.next(); // {value: 2, done: false}

gen2.next(); // {value: 1, done: false} — gen2 is COMPLETELY independent, starts fresh
```
Each call to `numberGenerator()` creates a **new, separate** generator object, each tracking its own progress independently — exactly like how each call to a regular constructor function creates a fresh, independent object. `gen1` being two steps in doesn't affect `gen2` at all.

**Regular functions can only give you ONE result, all at once.** Generators can give you a **sequence of results, one at a time, on demand** . This connects directly back to  **Iterables**.

Generators are automatically iterable — you can even use `for...of` directly on them:
```js
for (let num of numberGenerator()) {
  console.log(num); // 1, 2, 3
}
```
This is genuinely the payoff: **generators are a much shorter way to write custom iterables**, compared to manually implementing `Symbol.iterator` + a `next()` method yourself.

