##### Contains Intro , JS(browser) , JS fundamentals , Variables , Data Types , Operators , Conditionals and Loops

---
Initially, **JavaScript was created only to run inside web browsers**.
_JavaScript_ was initially created to “make web pages alive”.
The programs in this language are called _scripts_.
### Before JavaScript (Early 1990s)
- Websites were mostly **static** (just HTML and images).
- Browsers could display information, but users couldn't interact much with pages.
### Creation of JavaScript (1995)
Brendan Eich created JavaScript in **1995** while working at Netscape Communications
- The browser was called **Netscape Navigator**.
- He developed the first version of JavaScript in **just 10 days**.
- The language was originally named:
    1. **Mocha**
    2. **LiveScript**
    3. Finally renamed **JavaScript**

- At that time, Java was very popular. Netscape renamed LiveScript to **JavaScript** mainly for marketing purposes.

Initially , It ran **only inside browsers** and was used for:
- Form validation , Showing alerts , Image rollovers, , Simple animations , Responding to button clicks

### Standardization (1997)
Different browsers started implementing JavaScript differently.
To solve this, the organization Ecma International standardized the language as **ECMAScript** in **1997**.
- JavaScript = Implementation
- ECMAScript = Specification (the official standard)
### AJAX Era (2005)
Developers discovered techniques using JavaScript and **AJAX**.
AJAX allowed web pages to:
- Update data without refreshing the entire page.
- Make web applications more interactive.
Examples:
- Gmail
- Google Maps

### Major Improvement – ES5 (2009)
ECMAScript 5 introduced features such as:
- Strict mode
- JSON support
- Array methods (`forEach`, `map`, `filter`, etc.)
### JavaScript Outside Browsers – Node.js (2009)
Node.js was created by Ryan Dahl.
This was revolutionary because JavaScript could now run **outside the browser**.
Developers could build:
- Web servers
- APIs
- Command-line tools
- Desktop applications
### -->Modern JavaScript
Today JavaScript is used for:
#### Frontend Development
- Manipulating web pages
- Building interactive UIs

Frameworks: React , Angular , Vue.js
#### -->Backend Development
Using Node.js.
#### Mobile Apps
- React Native
#### Desktop Apps
- Electron
#### Game Development
- Browser games using Canvas and WebGL
---
### JavaScript (Browser)

1. You write JavaScript code.
2. The browser's JS engine  **parses** the code.
3. It creates an **Abstract Syntax Tree (AST)**.
4. The engine generates **bytecode/intermediate representation**.
5. The engine's **JIT compiler** optimizes frequently used code into **machine code**.
6. The CPU executes it.

JavaScript Source Code
          ↓
     JS Engine (V8)
          ↓
        Parser
          ↓
         AST
          ↓
       Bytecode
          ↓
 JIT → Machine Code
          ↓
         CPU


#### An engine is the software that takes your source code and makes the computer actually run it.

- **Engine** = executes the language.
- **Runtime** = the engine **plus extra features** provided by the environment.
For example:
- **V8** is the **JavaScript engine**. (Chrome, Opera and Edge)
- **Node.js** is a **runtime** that includes **V8 + file system access + networking + timers**, etc.

--> JavaScript’s capabilities greatly depend on the environment it’s running in.
-> Node.js supports functions that allow JavaScript to read/write arbitrary files, perform network requests, etc.
--> In-browser JavaScript can do everything related to webpage manipulation, interaction with the user, and the webserver. JavaScript’s abilities in the browser are limited to protect the user’s safety. The aim is to prevent an evil webpage from accessing private information or harming the user’s data. If that website's JavaScript could do **anything**, it could:
- Read all your files
- Delete your photos
- Install malware
- Read saved passwords
- Spy on other websites you're logged into
That would be a disaster.
So browsers put JavaScript inside a **sandbox**. The sandbox protects your computer.

#### Limitation 1: No direct file system access
#### Limitation 2: Cannot execute system commands
#### Limitation 3: Limited access to hardware
#### Limitation 4: Same-Origin Policy : JS on one site cannot freely access another site's data.

Such limitations do not exist if JavaScript is used outside of the browser, for example on a server. Modern browsers also allow plugins/extensions which may ask for extended permissions.
### Code editors
The main difference between a “lightweight editor” and an “IDE” is that an IDE works on a project-level, so it loads much more data on start, analyzes the project structure if needed and so on. A lightweight editor is much faster if we need only one file.

---
## JavaScript Fundamentals

JavaScript programs can be inserted almost anywhere into an HTML document using the `<script>` tag.
If we have a lot of JavaScript code, we can put it into a separate file.
Script files are attached to HTML with the `src` attribute:
```javascript
<script src="/path/to/script.js"></script>
```
Here, `/path/to/script.js` is an absolute path to the script from the site root. One can also provide a relative path from the current page. For instance, `src="script.js"`, just like `src="./script.js"`, would mean a file `"script.js"` in the current folder.
We can give a full URL as well.

--> As a rule, only the simplest scripts are put into HTML. More complex ones reside in separate files.
The benefit of a separate file is that the browser will download it and store it in its cache.
Other pages that reference the same script will take it from the cache instead of downloading it, so the file is actually downloaded only once.
That reduces traffic and makes pages faster.

-> A single `<script>` tag can’t have both the `src` attribute and code inside.
- Statements can be separated with a semicolon.
- A semicolon may be omitted in most cases when a line break exists.
- JavaScript interprets the line break as an “implicit” semicolon. This is called an automatic semicolon insertion
- In most cases, a newline implies a semicolon. But in most cases does not mean always!
- But there are situations where JavaScript fails to assume a semicolon where it is really needed. Errors which occur in such cases are quite hard to find and fix.
- We recommend putting semicolons between statements even if they are separated by newlines.
- **One-line comments start with two forward slash characters `//`.**  (Ctrl+/)
- For multiline `/* … */`,    (Ctrl+Shift+/)

## -->“use strict”
The directive looks like a string: `"use strict"` or `'use strict'`. When it is located at the top of a script, the whole script works the “modern” way.
- Please make sure that `"use strict"` is at the top of your scripts, otherwise strict mode may not be enabled.
- Once we enter strict mode, there’s no going back.
- Modern JavaScript supports classes and modules , that enable `use strict` automatically.
---
# Variables

A variable is a “named storage” for data. We can use variables to store goodies, visitors, and other data.
To create a variable in JavaScript, use the `let` keyword.
- When the value is changed, the old data is removed from the variable:
- We can also declare two variables and copy data from one into the other.
- A variable should be declared only once.

```javascript
let message = 'Hello!'; // define the variable and assign the value
alert(message); // Hello!

let user = 'John', age = 25 , message = 'Hello';
```

In older scripts, you may also find another keyword: `var` instead of `let`...It also declares a variable but in a slightly different, old-school way.

### Variable naming
There are two limitations on variable names in JavaScript:
1. The name must contain only letters, digits, or the symbols `$` and `_`.
2. The first character must not be a digit.
3. Case matters
4. `let`, `class`, `return`, and `function` are reserved.

--> Normally, we need to define a variable before using it. But in the old times, it was technically possible to create a variable by a mere assignment of the value without using `let`. This still works now if we don’t put `use strict` in our scripts to maintain compatibility with old scripts.

### Constants
To declare a constant (unchanging) variable, use `const` instead of `let`..
They cannot be reassigned. An attempt to do so would cause an error...

```javascript
const myBirthday = '18.04.1982';
myBirthday = '01.01.2001'; // error, can't reassign the constant!
```

---
# Data types

There are eight basic data types in JavaScript. We can put any type in a variable.
Programming languages that allow such things, such as JavaScript, are called “dynamically typed”, meaning that there exist data types, but variables are not bound to any of them.

| Type      | Description                                   |
| --------- | --------------------------------------------- |
| String    | A text of characters enclosed in quotes       |
| Number    | A number representing a mathematical value    |
| Bigint    | A number representing a large integer         |
| Boolean   | A data type representing true or false        |
| Object    | A collection of key-value pairs of data       |
| Undefined | A primitive variable with no assigned value   |
| Null      | A primitive value representing object absence |
| Symbol    | A unique and primitive identifier             |
 
### -->Number
 Represents both integer and floating point numbers. Besides regular numbers, there are so-called special numeric values which also belong to this data type: `Infinity`, `-Infinity` and `NaN`.
 - `Infinity` represents the mathematical Infinity..
 - `NaN` represents a computational error.. an incorrect or an undefined mathematical operation..

```javascript
alert( 1 / 0 ); // Infinity
alert( Infinity ); // Infinity

alert( "not a number" / 2 ); // NaN, such division is erroneous
alert( NaN + 1 ); // NaN
alert( 3 * NaN ); // NaN
alert( "not a number" / 2 - 1 ); // NaN
```

### -->BigInt
In JavaScript, the number type cannot safely represent integer values larger than `(253-1)` (that’s `9007199254740991`), or less than `-(253-1)` for negatives.

### -->String
A string in JavaScript must be surrounded by quotes.
Double and single quotes are “simple” quotes , practically no difference between them in JS
Backticks are extended functionality quotes. They allow us to embed variables and expressions into a string by wrapping them in `${…}`

```javascript
let name = "John";
alert( `Hello, ${name}!` ); // Hello, John!
alert( `the result is ${1 + 2}` ); // the result is 3
```
--> In JavaScript, `null` is not a reference to a non-existing object ..
It’s just a special value which represents “nothing”, “empty” or “value unknown”.

--> The meaning of `undefined` is “value is not assigned”.
	If a variable is declared, but not assigned, then its value is `undefined`

## Objects and Symbols

The `object` type is special.
All other types are called “primitive” because their values can contain only a single thing (be it a string or a number or whatever). In contrast, objects are used to store collections of data and more complex entities.
-->  The `symbol` type is used to create unique identifiers for objects. We have to mention it here for the sake of completeness, but also postpone the details till we know objects.

--> The *`typeof`* operator returns the type of the operand.

```javascript
typeof undefined;      // "undefined"
typeof 0;              // "number"
typeof 10n;            // "bigint"
typeof true;           // "boolean"
typeof "foo";          // "string"
typeof Symbol("id");   // "symbol"
typeof Math;           // "object"
typeof null;           // "object"
typeof alert;          // "function"
```

Last three lines may need additional explanation:
1. `Math` is a built-in object that provides mathematical operations.
2. The result of `typeof null` is object. That’s an officially recognized error in `typeof`, coming from very early days of JavaScript and kept for compatibility. 
3. The result of `typeof alert` is `"function"`, because `alert` is a function. Functions belong to the object type. But `typeof` treats them differently, returning `"function"`. That also comes from the early days of JavaScript.
==> `typeof` is an operator, not a function.  Some people prefer `typeof(x)`, although the `typeof x` syntax is much more common.

##### -->Of those 8, seven are primitive types: string, number, boolean, bigint, undefined, null, symbol. A primitive is a single, simple value.
##### --> The eighth, Object, is different — it's a container that groups many values together (arrays, objects, dates, functions are all objects). These are sometimes called reference types.
Why care? Because primitives and objects behave differently when you copy them

---
```javascript
process.stdout.write("boom");
process.stdout.write("boom");
// Output:  chaichai   ← stuck together on ONE line

console.log("boom");
console.log("boom");
// Output:
// chai
// chai              ← each on its OWN line
```
`console.log` automatically adds a newline (a line break) at the end. `process.stdout.write` does not.  Stick with `console.log`.
#### The console family
When you type `console.` and pause, your editor suggests many methods. A few useful ones:
```javascript
console.log("normal message");
console.info("informational message");
console.warn("a warning");      // shows in yellow in browsers
console.error("an error");      // shows in red in browsers
console.clear();                // wipes the console clean
console.table(someData);        // prints data as a neat table
console.table({ city: "Jaipur" });
```
`console.table` is genuinely cool with structured data..

### Borrowing (copying) values between variables
- For **primitives** (number, string, boolean…), borrowing copies the actual value. The two variables become independent
- For **objects**, "borrowing" copies the _reference_ (the address), so both names point to the _same_ underlying object — change one, the other sees it too
```javascript
let a = 10;
let b = a;   // b gets a copy: 10
a = 99;      // change a...
console.log(b);  // still 10 — b is independent

let user1 = { name: "A" };
let user2 = user1;       // copies the reference, not the object
user2.name = "B";
console.log(user1.name); // "B" — same object!
```

# Interaction: alert, prompt, confirm

## alert
This one we’ve seen already. It shows a message and waits for the user to press “OK”.
The mini-window with the message is called a _modal window_.
### prompt
The function `prompt` accepts two arguments:
It shows a modal window with a text message, an input field for the visitor, and the buttons OK/Cancel.
`title`
The text to show the visitor.
`default`
An optional second parameter, the initial value for the input field.

```Javascript
result = prompt(title, [default]);
```
## confirm
The syntax:
```javascript
result = confirm(question);
```

The function `confirm` shows a modal window with a `question` and two buttons: OK and Cancel.
The result is `true` if OK is pressed and `false` otherwise.

--> All these methods are modal: they pause script execution and don’t allow the visitor to interact with the rest of the page until the window has been dismissed.

```javascript
alert("Hello");

let age = prompt('How old are you?', 100);
alert(`You are ${age} years old!`); // You are 100 years old!

let isBoss = confirm("Are you the boss?");
alert( isBoss ); // true if OK is pressed
```

---
## -->Type Conversions
Most of the time, operators and functions automatically convert the values given to them to the right type. For example, `alert` automatically converts any value to a string to show it. Mathematical operations convert values to numbers..

### String Conversion
String conversion happens when we need the string form of a value.
`alert(value)` does it to show the value.
We can also call the `String(value)` function to convert a value to a string..

### Numeric Conversion
Numeric conversion in mathematical functions and expressions happens automatically.
We can use the `Number(value)` function to explicitly convert a `value` to a number..
Explicit conversion is usually required when we read a value from a string-based source like a text form but expect a number to be entered.
If the string is not a valid number, the result of such a conversion is `NaN`.

### Boolean Conversion
It happens in logical operations  but can also be performed explicitly with a call to `Boolean(value)`.

The conversion rule:
- Values that are intuitively empty, like `0`, an empty string, `null`, `undefined`, and `NaN`, become `false`.
- Other values become `true`.

```javascript
let value = true;
alert(typeof value); // boolean
value = String(value); // now value is a string "true"
alert(typeof value); // string

alert( "6" / "2" ); // 3, strings are converted to numbers
let str = "123";
let num = Number(str); // becomes a number 123
let age = Number("an arbitrary string instead of a number");
alert(age); // NaN, conversion failed

alert( Number("   123   ") ); // 123
alert( Number("123z") );      // NaN (error reading a number at "z")
alert( Number(true) );        // 1
alert( Number(false) );       // 0

alert( Boolean(1) ); // true
alert( Boolean(0) ); // false
alert( Boolean("hello") ); // true
alert( Boolean("") ); // false
```

---
# Basic operators

- An operator is _unary_ if it has a single operand.
- An operator is _binary_ if it has two operands.

The following math operations are supported:
- Addition `+`,
- Subtraction `-`,
- Multiplication `*`,
- Division `/`,
- Remainder `%`,
- Exponentiation `**`.
```javascript
let s = "my" + "string";
alert(s); // mystring

alert( '1' + 2 ); // "12"
alert( 2 + '1' ); // "21
alert(2 + 2 + '1' ); // "41" and not "221"
alert( 6 - '2' ); // 4, converts '2' to a number
alert( '6' / '2' ); // 3, converts both operands to numbers

alert( +true ); // 1        unary example
alert( +"" );   // 0
```
The binary `+` is the only operator that supports strings in such a way. Other arithmetic operators work only with numbers and always convert their operands to numbers.

--> JavaScript Operator Precedence (Highest → Lowest)

1. ()                          Parentheses
2. ++  --  !  typeof  +x  -x   Unary Operators
3. **                          Exponentiation
4. *  /  %                     Multiplication, Division, Modulus
5. +  -                        Addition, Subtraction
6. <  <=  >  >=                Comparison
7. ==  !=  ===  !==            Equality
8. &&                          Logical AND
9. ||                          Logical OR
10. ?:                         Ternary Operator
11. =  +=  -=  *=  /=  %=      Assignment

### Assignment = returns a value
- **Increment** `++` increases a variable by 1.
- **Decrement** `--` decreases a variable by 1.
- Increment/decrement can only be applied to variables. Trying to use it on a value like `5++` will give an error.
-  When the operator goes after the variable, it is in postfix form : `counter++`.
- The prefix form is when the operator goes before the variable : `++counter`.
- The prefix form returns the new value while the postfix form returns the old value (prior to increment/decrement).

```javascript
let counter = 1;
let a = ++counter; // (*)
alert(a); // 2

let counter = 1;
let a = counter++; // (*) changed ++counter to counter++
alert(a); // 1

let counter = 1;
alert( 2 * ++counter ); // 4

let counter = 1;
alert( 2 * counter++ ); // 2, because counter++ returns the "old" value
```
If we’d like to increase a value _and_ immediately use the result of the operator, we need the prefix form..
If we’d like to increment a value but use its previous value, we need the postfix form.

### Comma
The comma operator `,` is one of the rarest and most unusual operators.
The comma operator allows us to evaluate several expressions, dividing them with a comma `,`. Each of them is evaluated but only the result of the last one is returned.
```javascript
let a = (1 + 2, 3 + 4);
alert( a ); // 7 (the result of 3 + 4)
```
Comma has a very low precedence
Without parentheses: `a = 1 + 2, 3 + 4` evaluates `+` first, summing the numbers into `a = 3, 7`, then the assignment operator `=` assigns `a = 3`, and the rest is ignored. It’s like `(a = 1 + 2), 3 + 4`.

---
## Comparisons

In JavaScript they are written like this:
- Greater/less than: `a > b`, `a < b`.
- Greater/less than or equals: `a >= b`, `a <= b`.
- Equals: `a == b`, please note the double equality sign `==` means the equality test, while a single one `a = b` means an assignment.
- Not equals: In maths the notation is `≠`, but in JavaScript it’s written as `a != b`.
- - `true` – means “yes”, “correct” or “the truth”.
- `false` – means “no”, “wrong” or “not the truth”.

--> String comparison:  strings are compared letter-by-letter. dictionary or lexicographical order.
Not a real dictionary, but Unicode order
Case matters. A capital letter `"A"` is not equal to the lowercase `"a"`. Which one is greater? The lowercase `"a"`. Why? Because the lowercase character has a greater index in the internal encoding table JavaScript uses (Unicode).
```javascript
alert( 'Z' > 'A' ); // true
alert( 'Glow' > 'Glee' ); // true
alert( 'Bee' > 'Be' ); // true
```

```javascript
alert( '2' > 1 ); // true, string '2' becomes a number 2
alert( '01' == 1 ); // true, string '01' becomes a number 1

alert( true == 1 ); // true
alert( false == 0 ); // true
```

## Strict equality
A regular equality check `==` has a problem. It cannot differentiate `0` from `false`.
This happens because operands of different types are converted to numbers by the equality operator `==`
**A strict equality operator `===` checks the equality without type conversion.**
For maths and other comparisons `< > <= >=`
`null/undefined` are converted to numbers: `null` becomes `0`, while `undefined` becomes `NaN`.
The value `undefined` shouldn’t be compared to other values..it is false always

```javascript
alert( 0 == false ); // true
alert( 0 === false ); // false, because the types are different

alert( null === undefined ); // false
alert( null == undefined ); // true

alert( null > 0 );  // (1) false
alert( null == 0 ); // (2) false
alert( null >= 0 ); // (3) true
```

---
# Conditional branching: if, '?'
The `if(...)` statement evaluates a condition in parentheses and, if the result is `true`, executes a block of code.
The `if` statement may contain an optional `else` block. It executes when the condition is falsy.
Sometimes, we’d like to test several variants of a condition. The `else if` clause lets us do that.
```javascript
if (year == 2015) {
  alert( "That's correct!" );
  alert( "You're so smart!" );
}

let year = prompt('In which year was the ECMAScript-2015 specification published?', '');
if (year < 2015) {
	alert( 'Too early...' );
} else if (year > 2015) {
	alert( 'Too late' );
} else {
	alert( 'Exactly!' );
}
```

## Conditional operator ‘?’
Sometimes it’s called ternary, because the operator has three operands. It is actually the one and only operator in JavaScript which has that many.
```javascript
let result = condition ? value1 : value2;

let accessAllowed = (age > 18) ? true : false;

let age = prompt('age?', 18);
let message = (age < 3) ? 'Hi, baby!' :
	(age < 18) ? 'Hello!' :
	(age < 100) ? 'Greetings!' :
	'What an unusual age!';
alert( message );
```

Here’s how this looks using `if..else`:
```javascript
if (age < 3) {
  message = 'Hi, baby!';
} else if (age < 18) {
  message = 'Hello!';
} else if (age < 100) {
  message = 'Greetings!';
} else {
  message = 'What an unusual age!';
}
```

---
# Logical operators
There are four logical operators in JavaScript: `||` (OR), `&&` (AND), `!` (NOT), `??` (Nullish Coalescing).

## || (OR)
The “OR” operator is represented with two vertical line symbols.
In classical programming,If any of its arguments are `true`, it returns `true`, otherwise it returns `false`.
## OR "||" finds the first truthy value
- Evaluates operands from left to right.
- For each operand, converts it to boolean. If the result is `true`, stops and returns the original value of that operand.
- If all operands have been evaluated (i.e. all were `false`), returns the last operand.

```javascript
result = value1 || value2 || value3;

let firstName = "";
let lastName = "";
let nickName = "SuperCoder";
alert( firstName || lastName || nickName || "Anonymous"); // SuperCoder
```
**Short-circuit evaluation.**
->Another feature of OR `||` operator is the so-called “short-circuit” evaluation.
->It means that `||` processes its arguments until the first truthy value is reached, and then the value is returned immediately, without even touching the other argument.
->The importance of this feature becomes obvious if an operand isn’t just a value, but an expression with a side effect, such as a variable assignment or a function call.

In the example below, only the second message is printed
In the first line, the OR `||` operator stops the evaluation immediately upon seeing `true`, so the `alert` isn’t run.
```javascript
true || alert("not printed");
false || alert("printed");
```

## && (AND)
## AND “&&” finds the first falsy value
Precedence of AND `&&` is higher than OR `||`
So the code `a && b || c && d` is essentially the same as if the `&&` expressions were in parentheses: `(a && b) || (c && d)`.


```javascript
alert( 1 && 2 && null && 3 ); // null
alert( 1 && 2 && 3 ); // 3, the last one
```

## ! (NOT)
A double NOT `!!` is sometimes used for converting a value to boolean type:
```javascript
alert( !true ); // false
alert( !0 ); // true

alert( !!"non-empty string" ); // true
alert( !!null ); // false

```

# Nullish coalescing operator '??'
The result of `a ?? b` is:
- if `a` is defined, then `a`,
- if `a` isn’t defined, then `b`.
In other words, `??` returns the first argument if it’s not `null/undefined`. Otherwise, the second one.
The OR `||` operator can be used in the same way as `??`
The important difference between them is that:
- `||` returns the first _truthy_ value.
- `??` returns the first _defined_ value.
The precedence of the `??` operator is the same as `||`

```javascript
let user;
alert(user ?? "Anonymous"); // Anonymous (user is undefined)

let firstName = null;
let lastName = null;
let nickName = "Supercoder";
// shows the first defined value:
alert(firstName ?? lastName ?? nickName ?? "Anonymous"); // Supercoder
```

---
## Loops: while and for
_Loops_ are a way to repeat the same code multiple times.

### The while loop
a shorter way to write `while (i != 0)` is `while (i)`
```javascript
let i = 0;
while (i < 3) { // shows 0, then 1, then 2
  alert( i );
  i++;
}

let i = 3;
while (i) { // when i becomes 0, the condition becomes falsy, and the loop stops
   alert( i );
   i--;
}
```

### The do…while loop
The loop will first execute the body, then check the condition, and, while it’s truthy, execute it again and again.
This form of syntax should only be used when you want the body of the loop to execute **at least once** regardless of the condition being truthy. Usually, the other form is preferred: `while(…) {…}`.

```javascript
let i = 0;
do {
  alert( i );
  i++;
} while (i < 3);
```

### The for loop
The `for` loop is more complex, but it’s also the most commonly used loop.
```javascript
for (begin; condition; step) {
  // ... loop body ...
}

for (let i = 0; i < 3; i++) { // shows 0, then 1, then 2
	alert(i);
}

let i = 0; // we have i already declared and assigned
for (; i < 3; i++) { // no need for "begin"
	alert( i ); // 0, 1, 2
}
```
Here, the counter variable `i` is declared right in the loop. This is called an inline variable declaration. Such variables are visible only inside the loop.
Instead of defining a variable, we could use an existing one

### Breaking the loop

Normally, a loop exits when its condition becomes falsy.
But we can force the exit at any time using the special `break` directive.

-->>The `continue` directive is a lighter version of `break`. It doesn’t stop the whole loop. Instead, it stops the current iteration and forces the loop to start a new one (if the condition allows).
For even values of `i`, the `continue` directive stops executing the body and passes control to the next iteration of `for` (with the next number). So the `alert` is only called for odd values.
```javascript
for (let i = 1; i <= 5; i++) {
    if (i === 3) {                      // output is 1,2 only
        break;
    }
	console.log(i);
}

for (let i = 0; i < 10; i++) {
// if true, skip the remaining part of the body
	if (i % 2 == 0) continue;
	alert(i); // 1, then 3, 5, 7, 9
}
```

---
### The switch statement
A `switch` statement can replace multiple `if` checks.
It gives a more descriptive way to compare a value with multiple variants
```javascript
let a = 2 + 2;

switch (a) {
  case 3:
    alert( 'Too small' );
    break;
  case 4:
    alert( 'Exactly!' );
    break;
  case 5:
    alert( 'Too big' );
    break;
  default:
    alert( "I don't know such values" );
```
**If there is no `break` then the execution continues with the next `case` without any checks.**
