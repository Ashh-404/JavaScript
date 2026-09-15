Includes  Intro , Number , String , Array , Array methods and Iterables...

---
- There are 7 primitive types: `string`, `number`, `bigint`, `boolean`, `symbol`, `null` and `undefined`.

An object
- Is capable of storing multiple values as properties.
- Can be created with `{}`, for instance: `{name: "John", age: 30}`. There are other kinds of objects in JavaScript: functions..
- Objects are heavier than primitives. They require additional resources to support the internal machinery.

## A primitive as an object
Here’s the paradox faced by the creator of JavaScript:
- There are many things one would want to do with a primitive. It would be great to access them using methods.
- Primitives must be as fast and lightweight as possible.

The solution looks a little bit awkward, but here it is:
1. The language allows access to methods and properties of strings, numbers, booleans and symbols.
2. In order for that to work, a special object wrapper that provides the extra functionality is created, and then is destroyed.
The object wrappers are different for each primitive type and are called: `String`, `Number`, `Boolean`, `Symbol` and `BigInt`. Thus, they provide different sets of methods.

```javascript
let str = "Hello";
alert( str.toUpperCase() ); // HELLO

let n = 1.23456;
alert( n.toFixed(2) ); // 1.23
```
what actually happens in `str.toUpperCase()`:
1. The string `str` is a primitive. So in the moment of accessing its property, a special object is created that knows the value of the string, and has useful methods, like `toUpperCase()`.
2. That method runs and returns a new string (shown by `alert`).
3. The special object is destroyed, leaving the primitive `str` alone.
So primitives can provide methods, but they still remain lightweight.

-->>Constructors `String/Number/Boolean` are for internal use only
-->>Some languages like Java allow us to explicitly create wrapper objects for primitives using a syntax like `new Number(1)` or `new Boolean(false)`.
-->>In JavaScript, that’s also possible for historical reasons, but highly **unrecommended**. 
- Objects are always truthy in `if`
- null/undefined have no methods . they are “the most primitive"...

---
# Numbers
Regular numbers in JavaScript are stored in 64-bit format IEEE-754, also known as “double precision floating point numbers”.
In other words, `e` multiplies the number by `1` with the given zeroes count.
```javascript
let billion = 1000000000;
let billion = 1_000_000_000;
let billion = 1e9;  // 1 billion, literally: 1 and 9 zeroes
alert( 7.3e9 );  // 7.3 billions (same as 7300000000 or 7_300_000_000)
1e3 === 1 * 1000;           // e3 means *1000
1.23e6 === 1.23 * 1000000;  // e6 means *1000000

let mcs = 0.000001;
let mcs = 1e-6; // five zeroes to the left from 1
1e-3 === 1 / 1000; // 0.001
1.23e-6 === 1.23 / 1000000; // 0.00000123
1234e-2 === 1234 / 100; // 12.34, decimal point moves 2 times
```
## toString(base)
The method `num.toString(base)` returns a string representation of `num` in the numeral system with the given `base`.
The `base` can vary from `2` to `36`. By default, it’s `10`.
```javascript
let num = 255;
alert( num.toString(16) );  // ff
alert( num.toString(2) );   // 11111111

alert( 123456..toString(36) ); // 2n9c
```
Please note that two dots in `123456..toString(36)` is not a typo. If we want to call a method directly on a number, then we need to place two dots `..` after it.
Also could write `(123456).toString(36)`.

## Rounding
`Math.floor`
Rounds down: `3.1` becomes `3`, and `-1.1` becomes `-2`.

`Math.ceil`
Rounds up: `3.1` becomes `4`, and `-1.1` becomes `-1`.

`Math.round`
Rounds to the nearest integer: `3.1` becomes `3`, `3.6` becomes `4`. In the middle cases `3.5` rounds up to `4`, and `-3.5` rounds up to `-3`.

`Math.trunc` (not supported by Internet Explorer)
Removes anything after the decimal point without rounding: `3.1` becomes `3`, `-1.1` becomes `-1`

The method toFixed(n) rounds the number to `n` digits after the point and returns a string representation of the result.
```javascript
let num = 12.34;
alert( num.toFixed(1) ); // "12.3"

let num = 12.36;
alert( num.toFixed(1) ); // "12.4"

let num = 12.34;
alert( num.toFixed(5) ); // "12.34000", added zeroes to make exactly 5 digits
```

## Imprecise calculations
Internally, a number is represented in 64-bit format IEEE-754, so there are exactly 64 bits to store a number: 52 of them are used to store the digits, 11 of them store the position of the decimal point, and 1 bit is for the sign.

If a number is really huge, it may overflow the 64-bit storage and become a special numeric value `Infinity`:
-->> A number is stored in memory in its binary form. But fractions like `0.1`, `0.2` that look simple  are actually unending fractions in their binary form.
-->> There’s just no way to store _exactly 0.1_ or _exactly 0.2_ using the binary system, just like there is no way to store one-third as a decimal fraction.
-->> The same issue exists in many other programming languages.
PHP, Java, C, Perl, and Ruby give exactly the same result, because they are based on the same numeric format.
```javascript
alert( 1e500 ); // Infinity

alert(0.1 + 0.2 == 0.3); // false
alert(0.1 + 0.2); // 0.30000000000000004

alert(0.1.toFixed(20)); // 0.10000000000000000555

let sum = 0.1 + 0.2;
alert(sum.toFixed(2)); // "0.30"

// Hello! I'm a self-increasing number!
alert(9999999999999999); // shows 10000000000000000

```
 `toFixed` always returns a string. It ensures that it has 2 digits after the decimal point.
 **`Object.is(a, b)` is the same as `a === b`.** 

## parseInt and parseFloat
They read a number from a string until they can’t. In case of an error, the gathered number is returned. The function `parseInt` returns an integer, whilst `parseFloat` will return a floating-point number:
```javascript
alert( parseInt('100px') ); // 100
alert( parseFloat('12.5em') ); // 12.5

alert( parseInt('12.3') ); // 12, only the integer part is returned
alert( parseFloat('12.3.4') ); // 12.3, the second point stops the reading

alert( parseInt('a123') ); // NaN, the first symbol stops the process

alert( parseInt('0xff', 16) ); // 255
alert( parseInt('ff', 16) ); // 255, without 0x also works
alert( parseInt('2n9c', 36) ); // 123456
```
The second argument of `parseInt(str, radix)`
The `parseInt()` function has an optional second parameter. It specifies the base of the numeral system, so `parseInt` can also parse strings of hex numbers, binary numbers and so on:

## Other math functions
`Math.random()`
Returns a random number from 0 to 1 (not including 1).

`Math.max(a, b, c...)` and `Math.min(a, b, c...)`

`Math.pow(n, power)`
Returns `n` raised to the given power.

```javascript
alert( Math.random() ); // 0.1234567894322
alert( Math.random() ); // 0.5435252343232

alert(Math.max(3, 5, -10, 0, 1)); // 5
alert(Math.min(1, 2)); // 1
alert(Math.pow(2,10));// 2 in power 10 = 1024`
```

For regular number tests:
- `isNaN(value)` converts its argument to a number and then tests it for being `NaN`
- `Number.isNaN(value)` checks whether its argument belongs to the `number` type, and if so, tests it for being `NaN`
- `isFinite(value)` converts its argument to a number and then tests it for not being `NaN/Infinity/-Infinity`
- `Number.isFinite(value)` checks whether its argument belongs to the `number` type, and if so, tests it for not being `NaN/Infinity/-Infinity`

## QUESTIONS
```javascript
alert( 1.35.toFixed(1) ); // 1.4
alert( 6.35.toFixed(1) ); // 6.3
```
The precision loss can cause both increase and decrease of a number. In this particular case the number becomes a tiny bit less, that’s why it rounded down.
6.34999999999999964473 and 1.35000000000000008882

#### 2_This loop is infinite. It never ends. Why?

```javascript
let i = 0;
while (i != 10) {
  i += 0.2;
}
```
That’s because `i` would never equal `10`.

#### 3_A random number from min to max
```javascript
alert( random(1, 5) ); // 1.2345623452
alert( random(1, 5) ); // 3.7894332423
alert( random(1, 5) ); // 4.3435234525
```
we can do same with randomInteger...

---
# Strings
### Strings are immutable
```javascript
let str = 'Hi';
str[0] = 'h'; // error
alert( str[0] ); // doesn't work
```

For more than 1 line ..we can do these..
```javascript
let str1 = "Hello\nWorld"; // two lines using a "newline symbol"
// two lines using a normal newline and backticks
let str2 = `Hello
World`;
alert(str1 == str2); // true

alert( "I'm the Walrus!" ); // I'm the Walrus!
```

To get a character at position `pos`, use square brackets `[pos]` or call the method [str.at(pos)]. 
The square brackets always return `undefined` for negative indexes
```javascript
let str = `Hello`;
alert( str[0] ); // H
alert( str.at(0) ); // H
alert( str[str.length - 1] ); // o
alert( str.at(-1) );
alert( str[-2] ); // undefined
alert( str.at(-2) ); // l
```
Methods [toLowerCase()] and [toUpperCase()]change the case:
```javascript
alert( 'Interface'.toUpperCase() ); // INTERFACE
alert( 'Interface'.toLowerCase() ); // interface
alert( 'Interface'[0].toLowerCase() ); // 'i'
```
## Searching for a substring
The first method is [str.indexOf(substr, pos)].
It looks for the `substr` in `str`, starting from the given position `pos`, and returns the position where the match was found or `-1` if nothing can be found.
```javascript
let str = 'Widget with id';
alert( str.indexOf('Widget') ); // 0, because 'Widget' is found at the beginning
alert( str.indexOf('widget') ); // -1, not found, the search is case-sensitive
alert( str.indexOf("id") ); // 1, "id" is found at the position 1 (..idget with id)

alert( str.indexOf('id', 2) ) // 12
```
### includes, startsWith, endsWith
```javascript
alert( "Widget".includes("id") ); // true
alert( "Widget".includes("id", 3) ); // false, from position 3 there is no "id"

alert( "Widget".startsWith("Wid") ); // true, "Widget" starts with "Wid"
alert( "Widget".endsWith("get") ); // true, "Widget" ends with "get"
```
## Getting a substring
There are 3 methods in JavaScript to get a substring: `substring`, `substr` and `slice`.
Negative arguments are (unlike slice) not supported, they are treated as `0`.(substring)
Returns the part of the string from `start`, with the given `length`.(substr)
```javascript
let str = "stringify";
alert( str.slice(0, 5) ); // 'strin', the substring from 0 to 5 (not including 5)
alert( str.slice(0, 1) ); // 's', from 0 to 1,not including 1
alert(str.slice(2)); // 'ringify', from the 2nd position till the end
alert(str.slice(-4, -1)); // 'gif'

// substring()
alert(str.substring(2, 6)); // 'ring'
alert(str.substring(6, 2)); // 'ring'

// substr()
alert(str.substr(2, 4)); // 'ring', from the 2nd position get 4 characters
alert(str.substr(-4, 2)); // 'gi', from the 4th position from the end get 2 characters

```



---
# Arrays
A special data structure named `Array`, to store ordered collections.
There are two syntaxes for creating an empty array:
```javascript
let arr = new Array();
let arr = [];
```
Almost all the time, the second syntax is used. We can supply initial elements in the brackets:
We can replace an element , …Or add a new one to the array
The total count of the elements in the array is its `length`
We can also use `alert` to show the whole array.
```javascript
let fruits = ["Apple", "Orange", "Plum"];

alert( fruits[0] ); // Apple
alert( fruits[1] ); // Orange
alert( fruits[2] ); // Plum

fruits[2] = "Pear"; // now ["Apple", "Orange", "Pear"]
fruits[3] = "Lemon"; // now ["Apple", "Orange", "Pear", "Lemon"]
alert(fruits.length); // 3
alert(fruits); // Apple,Orange,Plum
```
An array can store elements of any type.
```javascript
let arr = [ 'Apple', { name: 'John' }, true, function() { alert('hello'); } ];
// get the object at index 1 and then show its name
alert( arr[1].name ); // John
// get the function at index 3 and run it
arr[3](); // hello
```
An array, just like an object, may end with a comma.. The trailing comma style makes it easier to insert/remove items, because all lines become alike.

## Methods pop/push, shift/unshift
A queue(FIFO) is one of the most common uses of an array.
- `push` appends an element to the end.
- `shift` get an element from the beginning, advancing the queue, so that the 2nd element becomes the 1st.
There’s another use case for arrays – the data structure named stack..(LIFO)
- `push` adds an element to the end.
- `pop` takes an element from the end.
Arrays in JavaScript can work both as a queue and as a stack. They allow you to add/remove elements, both to/from the beginning or the end.
In computer science, the data structure that allows this, is called deque..
Methods `push/pop` run fast, while `shift/unshift` are slow.

```javascript
let fruits = ["Apple", "Orange", "Pear"];
alert( fruits.pop() ); // remove "Pear" and alert it
alert( fruits ); // Apple, Orange
fruits.push("Pear");
alert( fruits ); // Apple, Orange, Pear
```
- `unshift`:  Add the element to the beginning of the array.
- Methods `push` and `unshift` can add multiple elements at once
```javascript
let fruits = ["Apple"];
fruits.push("Orange", "Peach");
fruits.unshift("Pineapple", "Lemon");
// ["Pineapple", "Lemon", "Apple", "Orange", "Peach"]
alert( fruits );
```

## Internals
An array is a special kind of object. The square brackets used to access a property `arr[0]` actually come from the object syntax. That’s essentially the same as `obj[key]`, where `arr` is the object, while numbers are used as keys.
Arrays are objects at their base. We can add any properties to them.
The ways to misuse an array:
- Add a non-numeric property like `arr.test = 5`.
- Make holes, like: add `arr[0]` and then `arr[1000]` (and nothing between them).
- Fill the array in the reverse order, like `arr[1000]`, `arr[999]` and so on.
## Loops
One of the oldest ways to cycle array items is the `for` loop over indexes:
But for arrays there is another form of loop, `for..of`:
```javascript
let arr = ["Apple", "Orange", "Pear"];
for (let i = 0; i < arr.length; i++) {
  alert( arr[i] );
}

let fruits = ["Apple", "Orange", "Plum"];
// iterates over array elements
for (let fruit of fruits) {
  alert( fruit );
}
```
The length property automatically updates when we modify the array. To be precise, it is actually not the count of values in the array, but the greatest numeric index plus one.
So, the simplest way to clear the array is: `arr.length = 0;`.
```javascript
let fruits = [];
fruits[123] = "Apple";
alert( fruits.length ); // 124

let arr = [1, 2, 3, 4, 5];
arr.length = 2; // truncate to 2 elements
alert( arr ); // [1, 2]
arr.length = 5; // return length back
alert( arr[3] ); // undefined: the values do not return
```
If `new Array` is called with a single argument which is a number, then it creates an array _without items, but with the given length_. (imp feature of new Array... `new` Array(2);)

## Multidimensional arrays
Arrays can have items that are also arrays. We can use it for multidimensional arrays, for example to store matrices
```javascript
let matrix = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
];
alert( matrix[0][1] ); // 2, the second value of the first inner array
```

## Don’t compare arrays with ==
So, if we compare arrays with `==`, they are never the same, unless we compare two variables that reference exactly the same array.
```javascript
alert( [] == [] ); // false
alert( [0] == [0] ); // false

alert( 0 == [] ); // true
alert('0' == [] ); // false
```
Here, in both cases, we compare a primitive with an array object. So the array `[]` gets converted to primitive for the purpose of comparison and becomes an empty string `''`.
Don’t use the `==` operator. Instead, compare them item-by-item in a loop or using iteration methods...

---
# Array methods
- `arr.push(...items)` – adds items to the end,
- `arr.pop()` – extracts an item from the end,
- `arr.shift()` – extracts an item from the beginning,
- `arr.unshift(...items)` – adds items to the beginning.
```javascript
let arr = ["I", "go", "home"];
delete arr[1]; // remove "go"
alert( arr[1] ); // undefined
// now arr = ["I",  , "home"];
alert( arr.length ); // 3
```
--> The element was removed, but the array still has 3 elements. That’s natural, because `delete obj.key` removes a value by the `key`. It’s all it does.
--> The [arr.splice] method is a Swiss army knife for arrays. It can do everything: insert, remove and replace elements. It modifies `arr` starting from the index `start`: removes `deleteCount` elements and then inserts `elem1, ..., elemN` at their place. Returns the array of removed elements.

```javascript
arr.splice(start[, deleteCount, elem1, ..., elemN])
```

```javascript
let arr = ["I", "study", "JavaScript"];
arr.splice(1, 1); // from index 1 remove 1 element
alert( arr ); // ["I", "JavaScript"]

let arr = ["I", "study", "JavaScript", "right", "now"];
// remove 3 first elements and replace them with another
arr.splice(0, 3, "Let's", "dance");
alert( arr ) // now ["Let's", "dance", "right", "now"]

// remove 2 first elements
let removed = arr.splice(0, 2);
alert( removed ); // "I", "study" <-- array of removed elements

let arr = [1, 2, 5];
// from index -1 (one step from the end)
// delete 0 elements,
// then insert 3 and 4
arr.splice(-1, 0, 3, 4);
alert( arr ); // 1,2,3,4,5
```

--> slice returns a new array copying to it all items from index `start` to `end` (not including `end`). Both `start` and `end` can be negative, in that case position from array end is assumed.
```javascript
let arr = ["t", "e", "s", "t"];
alert( arr.slice(1, 3) ); // e,s (copy from 1 to 3)
alert( arr.slice(-2) ); // s,t (copy from -2 till the end)
```
### concat
The method arr.concat creates a new array that includes values from other arrays and additional items.
```javascript
let arr = [1, 2];
// create an array from: arr and [3,4]
alert( arr.concat([3, 4]) ); // 1,2,3,4
// create an array from: arr and [3,4] and [5,6]
alert( arr.concat([3, 4], [5, 6]) ); // 1,2,3,4,5,6
// create an array from: arr and [3,4], then add values 5 and 6
alert( arr.concat([3, 4], 5, 6) ); // 1,2,3,4,5,6
```

## Iterate: forEach
This method allows to run a function for every element of the array.
The syntax:
```javascript
arr.forEach(function(item, index, array) {
  // ... do something with an item
});

// for each element call alert
["Bilbo", "Gandalf", "Nazgul"].forEach(alert);
```

## Searching in array
- `arr.indexOf(item, from)` – looks for `item` starting from index `from`, and returns the index where it was found, otherwise `-1`.
- `arr.includes(item, from)` – looks for `item` starting from index `from`, returns `true` if found.
- The method arr.lastIndex is the same as `indexOf`, but looks for from right to left.
```javascript
let arr = [1, 0, false];

alert( arr.indexOf(0) ); // 1
alert( arr.indexOf(false) ); // 2
alert( arr.indexOf(null) ); // -1

alert( arr.includes(1) ); // true
```
Please note that `indexOf` uses the strict equality `===` for comparison. So, if we look for `false`, it finds exactly `false` and not the zero.

### find and findIndex/findLastIndex

```javascript
let result = arr.find(function(item, index, array) {
  // if true is returned, item is returned and iteration is stopped
  // for falsy scenario returns undefined
});
```
The function is called for elements of the array, one after another:
- `item` is the element.
- `index` is its index.
- `array` is the array itself.
If it returns `true`, the search is stopped, the `item` is returned. If nothing is found, `undefined` is returned.
```javascript
let users = [
  {id: 1, name: "John"},
  {id: 2, name: "Pete"},
  {id: 3, name: "Mary"}
];
let user = users.find(item => item.id == 1);
alert(user.name); // John
```
To `find` the function `item => item.id == 1` with one argument. That’s typical, other arguments of this function are rarely used.

- The arr.findIndex method has the same syntax but returns the index where the element was found instead of the element itself. The value of `-1` is returned if nothing is found.
- The arr.findLastIndex method is like `findIndex`, but searches from right to left, similar to `lastIndexOf`.
```javascript
let users = [
  {id: 1, name: "John"},
  {id: 2, name: "Pete"},
  {id: 3, name: "Mary"},
  {id: 4, name: "John"}
];
// Find the index of the first John
alert(users.findIndex(user => user.name == 'John')); // 0
// Find the index of the last John
alert(users.findLastIndex(user => user.name == 'John')); // 3
```

```javascript
let users = [
  {id: 1, name: "John"},
  {id: 2, name: "Pete"},
  {id: 3, name: "Mary"}
];
// returns array of the first two users
let someUsers = users.filter(item => item.id < 3);
alert(someUsers.length); // 2
```
The arr.map calls the function for each element of the array and returns the array of results.
```javascript
let lengths = ["Bilbo", "Gandalf", "Nazgul"].map(item => item.length);
alert(lengths); // 5,7,6
```

```javascript
let arr = [ 1, 2, 15 ];
// the method reorders the content of arr
arr.sort();
alert( arr );  // 1, 15, 2
```
The order became `1, 15, 2`. Incorrect. But why?
**The items are sorted as strings by default.**
To use our own sorting order, we need to supply a function as the argument of `arr.sort()`.
```javascript
function compareNumeric(a, b) {
  if (a > b) return 1;
  if (a == b) return 0;
  if (a < b) return -1;
}
let arr = [ 1, 2, 15 ];
arr.sort(compareNumeric);
alert(arr);  // 1, 2, 15

let arr = [ 1, 2, 15 ];
arr.sort(function(a, b) { return a - b; });
alert(arr);  // 1, 2, 15

arr.sort( (a, b) => a - b );  //works same 
```

For many alphabets, it’s better to use `str.localeCompare` method to correctly sort letters, such as `Ö`.
```javascript
let countries = ['Österreich', 'Andorra', 'Vietnam'];
alert( countries.sort( (a, b) => a > b ? 1 : -1) ); // Andorra, Vietnam, Österreich (wrong)
alert( countries.sort( (a, b) => a.localeCompare(b) ) ); // Andorra,Österreich,Vietnam (correct!)
```
- The method arr.reverse reverses the order of elements in `arr`.
- The str.split(delim) method does exactly that. It splits the string into an array by the given delimiter `delim`.
- The `split` method has an optional second numeric argument – a limit on the array length. If it is provided, then the extra elements are ignored. In practice it is rarely used though:
```javascript
let arr = [1, 2, 3, 4, 5];
arr.reverse();
alert( arr ); // 5,4,3,2,1

let names = 'Bilbo, Gandalf, Nazgul';
let arr = names.split(', ');
for (let name of arr) {
   alert( `A message to ${name}.` ); // A message to Bilbo  (and other names)
}

let arr = 'Bilbo, Gandalf, Nazgul, Saruman'.split(', ', 2);
alert(arr); // Bilbo, Gandalf

let str = "test";
alert( str.split('') ); // t,e,s,t

let arr = ['Bilbo', 'Gandalf', 'Nazgul'];
let str = arr.join(';'); // glue the array into a string using ;
alert( str ); // Bilbo;Gandalf;Nazgul
```

## Array.isArray
Arrays do not form a separate language type. They are based on objects.
So `typeof` does not help to distinguish a plain object from an array:
```javascript
alert(typeof {}); // object
alert(typeof []); // object (same)

alert(Array.isArray({})); // false
alert(Array.isArray([])); // true
```

---
**With a primitive (number), copying truly copies:**
```js
let var1 = 5;
let var2 = var1;   // var2 gets its own copy of the value 5
var1 = 9;          // change var1...
console.log(var1, var2);  // 9 5  ← var2 is unaffected, fully independent
```
For primitives, `var2 = var1` snapshots the _value_. The two go their separate ways.

**With an array, the same syntax does NOT copy:**
```js
let popularTeas = ["green tea", "oolong tea", "chai"];
let softCopyTeas = popularTeas;   // looks like a copy...

popularTeas.pop();   // change ONLY popularTeas...
console.log(popularTeas);  // ["green tea", "oolong tea"]
console.log(softCopyTeas); // ["green tea", "oolong tea"]  ← it changed too?!
```
Primitive (independent):
  var1 ──► 5
  var2 ──► 5          (two separate boxes, two separate values)

Array (shared):
  popularTeas ──┐
                ├──►  [ "green tea", "oolong tea", "chai" ]   (ONE array in memory)
  softCopyTeas ──┘

For objects and arrays, a variable doesn't hold the array itself — it holds a **reference** (an address) pointing to where the array lives.
- `let b = a` → **reference assignment / aliasing** (one array, two names).
- making a genuinely separate array → a **copy** (next section), split into **shallow copy** and **deep copy**.

To get a _real, separate_ array, you use the **spread operator** `...`:
Two independent arrays now exist, so changing one leaves the other alone.
```js
let topCities = ["Berlin", "Singapore", "New York"];
let hardCopyCities = [...topCities];   // a brand-new array with the same items

topCities.pop();              // change the original...
console.log(topCities);       // ["Berlin", "Singapore"]
console.log(hardCopyCities);  // ["Berlin", "Singapore", "New York"]  ← untouched!
```
-> A second way, using a method:
```js
let hardCopyCities = topCities.slice();   // slice() with no arguments also copies
```
slice() normally extracts a _portion_ of an array, but called with no arguments it returns a copy of the whole thing. It works, but `[...arr]` is what you'll see far more often.

=> **The catch nobody warns beginners about — "shallow."** Both `[...arr]` and `slice()` make a **shallow copy**: they copy the top level only. If your array contains _nested_ arrays or objects, those inner items are still shared by reference:
```js
let original = [[1, 2], [3, 4]];
let copy = [...original];
copy[0].push(99);
console.log(original[0]);  // [1, 2, 99]  ← the inner array is STILL shared!
```
For flat arrays of strings/numbers , shallow copy is a perfect, true copy. For nested structures you'd need a **deep copy** (e.g. `structuredClone(original)`), which copies every level. 
The outer array got copied (two separate outer arrays now exist), but `shallow[0]` and `original[0]` are still the _same inner array_. Spread only went one layer deep.


```js
let original = [[1, 2], [3, 4], { name: "chai" }];
let deep = structuredClone(original);
deep[0].push(99);
deep[2].name = "green tea";
console.log(original[0]);     // [1, 2]        ← untouched
console.log(original[2].name);// "chai"        ← untouched
console.log(deep[0]);         // [1, 2, 99]
console.log(deep[2].name);    // "green tea"
```


---
## -->Iterables

```js
for (let num of [1, 2, 3]) { alert(num); }
```
But what makes an array loop-able with `for...of` while a plain object isn't?
```js
for (let key of { a: 1, b: 2 }) {} //  TypeError: {a:1,b:2} is not iterable
```
The answer: `for...of` doesn't have special hardcoded knowledge of arrays. Instead, it looks for a **specific method** on whatever you give it — and if that method exists, it works, no matter what kind of object it is. If you build that method yourself onto _any_ object, `for...of` will work on it too. That's the whole concept of iterable.

##### -->The magic method: `Symbol.iterator`

`Symbol.iterator` is a **predefined special symbol** that JS itself recognizes and looks for automatically whenever you write `for...of`.

**The contract, in plain terms**:
1. Your object needs a method named `[Symbol.iterator]`
2. That method must **return another object** — called the **iterator**
3. That iterator object must have a `next()` method
4. Every time `for...of` wants the next value, it calls `next()`
5. `next()` must return `{ done: false, value: someValue }` while there's more to give, or `{ done: true }` when finished

##### Building a custom iterable 
```js
let range = { from: 1, to: 5 };
```
Right now this is just a plain object — `for...of` doesn't know what to do with it. Let's teach it:
```js
range[Symbol.iterator] = function() {
  return {
    current: this.from,
    last: this.to,

    next() {
      if (this.current <= this.last) {
        return { done: false, value: this.current++ };
      } else {
        return { done: true };
      }
    }
  };
};

for (let num of range) {
  alert(num); // 1, 2, 3, 4, 5
}
```

**Trace exactly what happens when `for...of` runs:**
1. `for...of` sees `range`, and calls `range[Symbol.iterator]()` **once** — this returns a fresh object with a `current` counter and a `next()` method. Call this the **iterator**.
2. `for...of` calls `iterator.next()` → `current` (1) ≤ `last` (5) → returns `{ done: false, value: 1 }` → loop prints `1`, and `current` becomes 2 (because of `current++`, which increments _after_ returning the old value).
3. Calls `next()` again → `{ done: false, value: 2 }` → prints `2`, `current` becomes 3.
4. ...continues... eventually `current` becomes 6.
5. Calls `next()` again → `current` (6) is NOT ≤ `last` (5) → returns `{ done: true }` → loop stops, no more printing.

##### -->Why iterator is a separate object from iterable

 The range object itself doesn't have `next()`. Instead, calling `range[Symbol.iterator]()` **creates a brand new separate object** that has `next()` and tracks progress (`current`).
--> Why does this separation matter? Because it means you could theoretically run **two independent `for...of` loops** over the same `range` at the same time, and they wouldn't interfere with each other — each loop gets its **own fresh iterator object**, with its **own independent `current` counter**, since `Symbol.iterator` gets called fresh each time.

You can simplify by making `range` itself double as the iterator:
```js
let range = {
  from: 1,
  to: 5,

  [Symbol.iterator]() {
    this.current = this.from;
    return this; // return the range object itself, not a separate new one
  },

  next() {
    if (this.current <= this.to) {
      return { done: false, value: this.current++ };
    } else {
      return { done: true };
    }
  }
};
for (let num of range) { alert(num); } // 1, 2, 3, 4, 5
```


```js
range.to = Infinity;
```
Nothing forces `next()` to ever return `done: true`. You can build infinite sequences (random number generators, counters that never stop) — you'd just need to manually stop the loop with `break` when you're done, since it'll never naturally terminate on its own.

```js
for (let char of "test") {
  alert(char); // t, e, s, t
}
```
Strings secretly have their own built-in `Symbol.iterator` implementation, which is why `for...of` works on them directly, character by character.

-->**The surrogate pair detail** (a bit niche, but worth knowing): some characters (like emojis 😂 or certain math symbols 𝒳) are actually stored internally as **two** code units, not one — called a surrogate pair. String's built-in iterator is smart enough to treat these correctly as one single character
```js
let str = '𝒳😂';
for (let char of str) {
  alert(char); // 𝒳, then 😂 — correct, 2 iterations, not 4
}
```


You can bypass `for...of` entirely and do exactly what it does internally, by hand:
```js
let str = "Hello";
let iterator = str[Symbol.iterator](); // get the iterator directly

while (true) {
  let result = iterator.next();
  if (result.done) break;
  alert(result.value); // H, e, l, l, o
}
```
This is literally what `for...of` is doing under the hood, every single time.

#### -->Iterable vs Array-like 

**Iterable** = has a `Symbol.iterator` method → works with `for...of`.
**Array-like** = has numeric indexes (`0`, `1`, `2`...) and a `.length` property, just _looks_ like an array structurally..
```js
let arrayLike = {
  0: "Hello",
  1: "World",
  length: 2
};
```
This _looks_ array-ish (`arrayLike[0]`, `arrayLike.length` both work), but:
```js
for (let item of arrayLike) {} //  Error — no Symbol.iterator, so for..of has                                                 nothing to call
```

- Iterable but NOT array-like → `range` (no indexes/length, but has `Symbol.iterator`)
- Array-like but NOT iterable → arrayLike above (has indexes/length, but no `Symbol.iterator`)
- Both → arrays and strings (have indexes/length **and** `Symbol.iterator`)
- Neither → most plain objects


Since neither iterables nor array-likes have real array methods (`.push`, `.pop`, `.map`, etc.)
```js
let arrayLike = { 0: "Hello", 1: "World", length: 2 };
let arr = Array.from(arrayLike);
alert(arr.pop()); // "World" — now it's a real array with real methods!
```

```js
let arr = Array.from(range); // works on iterables too
alert(arr); // [1, 2, 3, 4, 5]
```
Under the hood, `Array.from` just checks: does this have Symbol.iterator? Or does it have indexes + length? — either way, it loops through and builds a genuine new `Array` from the values.

```js
let arr = Array.from(range, num => num * num);
alert(arr); // [1, 4, 9, 16, 25]
```
```js
let str = '𝒳😂';
let chars = Array.from(str);
alert(chars.length); // 2 — correct!
```
This is actually **better** than the older `str.split('')`, which can incorrectly split surrogate pairs into garbage. 

