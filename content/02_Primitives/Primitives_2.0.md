Includes Map, WeakMap , Set, WeakSet , 

---
## -->Map and Set

A **Map stores data as key-value pairs**.
```js
let map = new Map();

map.set('1', 'str1');   // string key
map.set(1, 'num1');     // number key
map.set(true, 'bool1'); // boolean key

alert(map.get(1));   // 'num1'
alert(map.get('1')); // 'str1'
```
Notice `1`  and `'1'`  are treated as **completely different keys**
this is literally just a Python dict
```python
d = {1: "num1", "1": "str1"}  # normal in Python — 1 and "1" are different keys
```

#### -->Core methods — small, memorize these

|Method|What it does|
|---|---|
|`map.set(key, value)`|add/update an entry|
|`map.get(key)`|read a value (returns `undefined` if key missing)|
|`map.has(key)`|check existence → `true`/`false`|
|`map.delete(key)`|remove one entry|
|`map.clear()`|remove everything|
|`map.size`|count of entries (note: **property**, not method — no `()`)|

```js
map[key] = 2; //  technically "works" but wrong
```
This bypasses Map entirely and just adds a **regular object property** onto the map object . It won't show up in map.size, map.get(), or iteration — it's silently broken. Always use `.set()`/`.get()`


This is where Map genuinely does something a plain object structurally **cannot**:
```js
let john = { name: "John" };
let visitsCountMap = new Map();

visitsCountMap.set(john, 123);
alert(visitsCountMap.get(john)); // 123
```


--> Map uses an algorithm called **SameValueZero** — basically identical to `===`, with **one exception**: `NaN` is considered equal to itself in a Map ..

##### -->Chaining — `.set()` returns the map itself
```js
map.set('1', 'str1')
   .set(1, 'num1')
   .set(true, 'bool1');
```
Since `.set()` returns the map, you can chain calls 

##### -->Iterating a Map 
```js
let recipeMap = new Map([
  ['cucumber', 500],
  ['tomatoes', 350],
  ['onion', 50]
]);

for (let veg of recipeMap.keys())   { alert(veg); }    // cucumber, tomatoes, onion
for (let amt of recipeMap.values()) { alert(amt); }    // 500, 350, 50
for (let entry of recipeMap)        { alert(entry); }  // [cucumber,500], etc —                                                            default iteration
```
Map guarantees iteration happens in **insertion order** — the order you actually called `.set()` in.
```js
recipeMap.forEach((value, key, map) => {
  alert(`${key}: ${value}`);
});
```
Same pattern as array's `forEach`, just with `(value, key, map)` args in that order.

##### -->Converting between Object and Map

**Object → Map**, using `Object.entries()`:
```js
let obj = { name: "John", age: 30 };
let map = new Map(Object.entries(obj));
alert(map.get('name')); // "John"
```
`Object.entries(obj)` turns `{name: "John", age: 30}` into `[["name","John"], ["age",30]]`

**Map → Object**, using `Object.fromEntries()`:
```js
let map = new Map();
map.set('banana', 1);
map.set('orange', 2);

let obj = Object.fromEntries(map); // { banana: 1, orange: 2 }
```


---
### -->Set 
A **Set stores unique values**. It does **not allow duplicates**.
```js
let set = new Set();

let john = { name: "John" };
let pete = { name: "Pete" };

set.add(john);
set.add(pete);
set.add(john); // adding again — does nothing, already exists

alert(set.size); // 2, not 3
```

##### Why not just use an array and manually check for duplicates?
```js
if (!arr.includes(item)) arr.push(item); // works, but SLOW for large arrays
```

`arr.includes()` has to check **every single element**, one by one, every time 

##### -->Set methods

|Method|What it does|
|---|---|
|`set.add(value)`|add a value (no-op if already present)|
|`set.delete(value)`|remove a value|
|`set.has(value)`|check existence|
|`set.clear()`|empty the set|
|`set.size`|count|

##### Iterating a Set
```js
let set = new Set(["oranges", "apples", "bananas"]);
for (let value of set) { alert(value); }
set.forEach((value, valueAgain, set) => { alert(value); });
```
Odd detail worth knowing: Set's `forEach` callback gets **the same value twice** (`value, valueAgain`) before the set itself. This looks redundant, but it's purely so Set's `forEach` signature _matches_ Map's `forEach(value, key, map)` shape — meaning code can be written generically to work with either Map or Set with minimal changes. 

---
## -->WeakMap and WeakSet

```js
let john = { name: "John" };
let map = new Map();
map.set(john, "...");

john = null; // we're done with john... right?
```
You'd think `john` is now garbage, ready for cleanup. But it's **not** — because `map` still holds a reference to that object as a key.  You'd have to manually `map.delete(john)` to free it

#### -->WeakMap — the fix
```js
let john = { name: "John" };
let weakMap = new WeakMap();
weakMap.set(john, "...");

john = null;
// john is now GONE — removed from memory AND from the weakMap automatically
```
A WeakMap holds what's called a weak reference to its keys. A weak reference means: _"I'm pointing at this object, but that pointer alone doesn't count as a reason to keep it alive."_ 

- **Regular Map** = a strong reference (keeps things alive as long as the Map exists).  
- **WeakMap** = a weak reference (doesn't keep things alive on its own).

##### Why WeakMap has restrictions 

**Restriction 1: keys must be objects, never primitives**
```js
weakMap.set("test", "value"); //  Error
```
Why? Because weak reference, gets garbage collected when unreachable only makes sense for objects. 

**Restriction 2: no `.size`, no `.keys()`, no `.values()`, no iteration at all**
```js
// Only these 4 methods exist:
weakMap.set(key, value)
weakMap.get(key)
weakMap.delete(key)
weakMap.has(key)
```

##### -->The real use case: "extra data that should die when the main object dies"
**Example — visit counting:**
```js
// with regular Map — PROBLEM
let visitsCountMap = new Map();

function countUser(user) {
  let count = visitsCountMap.get(user) || 0;
  visitsCountMap.set(user, count + 1);
}
```

```js
let john = { name: "John" };
countUser(john);
john = null; // john should be gone, but visitsCountMap still holds him — MEMORY LEAK
```
Every user who ever visited stays in memory forever unless you manually remember to clean `visitsCountMap`


```js
// with WeakMap — FIXED
let visitsCountMap = new WeakMap();

function countUser(user) {
  let count = visitsCountMap.get(user) || 0;
  visitsCountMap.set(user, count + 1);
}
```

```js
let john = { name: "John" };
countUser(john);
john = null; // john AND his visit count both vanish automatically. No leak, no manual cleanup.
```
You get to just... not think about cleanup. The moment nothing else references `john`, both `john` and his associated data in the WeakMap disappear together, automatically.

##### -->Second use case — caching

Same exact pattern, different purpose: cache a computed result _tied to_ an object, but don't keep that cached result around forever if the object itself is gone.

```js
let cache = new WeakMap();

function process(obj) {
  if (!cache.has(obj)) {
    let result = /* expensive calculation */;
    cache.set(obj, result);
    return result;
  }
  return cache.get(obj);
}
```

```js
let obj = {/* ... */};
let result1 = process(obj); // computed, cached
let result2 = process(obj); // reused from cache — fast

obj = null;
// cached result for obj is automatically freed once obj is garbage collected
```

With a regular `Map` here, `cache.size` would stay `1` forever even after `obj = null` — a silent, growing memory leak if `process()` gets called on thousands of temporary objects over the app's lifetime. WeakMap fixes this with zero extra code.

### WeakSet — same idea, simpler (no values, just membership)

```js
let visitedSet = new WeakSet();

let john = { name: "John" };
let pete = { name: "Pete" };

visitedSet.add(john);
visitedSet.add(pete);

alert(visitedSet.has(john)); // true

john = null;
// john automatically removed from visitedSet once garbage collected
```

WeakSet is for **yes/no facts** about an object , rather than storing an actual associated value like WeakMap does. Same weak-reference behavior, same restrictions (no iteration, no `.size`), same only objects, never primitives.

---
## -->Destructuring assignment

```js
let arr = ["John", "Smith"];
let firstName = arr[0];
let surname = arr[1];
```
That's two lines of repetitive `arr[...]` access just to get two values into variables. Destructuring lets you do this in **one line**, matching shape-to-shape:
```js
let [firstName, surname] = arr;
```
Think of the left side as a **template that mirrors the right side's shape** 

##### -->Array destructuring — position-based matching
```js
let [firstName, surname] = ["John", "Smith"];
alert(firstName); // John
alert(surname);   // Smith
```
**Critical thing to understand**: this matches by **position**, not by name. 
Destructuring" is not destructive— the original array `arr` is completely untouched. You're just _copying_ values out into new variables.

```js
let [firstName, , title] = ["Julius", "Caesar", "Consul", "of the Roman Republic"];
alert(title); // Consul
```
An empty comma slot just says skip this position, I don't want it.

##### Works on ANY iterable, not just arrays
```js
let [a, b, c] = "abc";           // strings are iterable
let [one, two, three] = new Set([1, 2, 3]); // Sets are iterable
```
 destructuring is really just sugar for calling `for...of` under the hood and grabbing values one at a time. Anything with `Symbol.iterator` works here.

##### Left side can be more than just plain variables
```js
let user = {};
[user.name, user.surname] = "John Smith".split(' ');
alert(user.name); // John
```
You can destructure directly into existing object properties, not just fresh `let`/`const` variables.

```js
let user = { name: "John", age: 30 };
for (let [key, value] of Object.entries(user)) {
  alert(`${key}:${value}`); // name:John, then age:30
}
```
`Object.entries(user)` gives you `[["name","John"], ["age",30]]` — an array of 2-item arrays. The `for...of` loop pulls out each pair, and `[key, value]` **destructures each pair** on the fly. This is _the_ standard, idiomatic way to loop over an object's key-value pairs in modern JS..

##### The classic swap trick
```js
let guest = "Jane", admin = "Pete";
[guest, admin] = [admin, guest];
alert(`${guest} ${admin}`); // Pete Jane
```

```js
let [name1, name2, ...rest] = ["Julius", "Caesar", "Consul", "of the Roman Republic"];

alert(rest[0]);     // Consul
alert(rest[1]);     // of the Roman Republic
alert(rest.length); // 2
```
`...rest` scoops up **everything not already claimed** into a new array. Must come **last** in the pattern. 

##### -->Default values — fill in gaps safely
```js
let [name = "Guest", surname = "Anonymous"] = ["Julius"];
alert(name);    // Julius (was provided)
alert(surname); // Anonymous (default used, since array was too short)
```
If the array is too short, missing spots become `undefined` — unless you give a default with `=`. Defaults are only used when the value is genuinely missing (`undefined`)
```js
let [name = prompt('name?'), surname = prompt('surname?')] = ["Julius"];
// prompt only runs for surname, since name was already provided
```

### -->Object destructuring — name-based matching 

```js
let options = { title: "Menu", width: 100, height: 200 };
let { title, width, height } = options;
```
**Key difference from arrays**: this matches by **property name**, not position. Order on the left side doesn't matter at all:
```js
let { height, width, title } = options; // still works identically
```
Variable names on the left must **match the property names** on the right

#### -->Renaming with `:`
```js
let { width: w, height: h, title } = options;
// width -> w, height -> h, title -> title (unchanged)
```

```js
let { width: w = 100, height: h = 200, title } = options;
```
Both pieces stack: rename `width`→`w`, AND give it a default of `100` if missing.


```js
let options = { title: "Menu", height: 200, width: 100 };
let { title, ...rest } = options;
// title = "Menu", rest = { height: 200, width: 100 }
```
Grabs everything not explicitly named into a new object.

#### -->The `({...} = {...})` gotcha 
```js
let title, width, height;
{title, width, height} = {...}; //  Syntax error!
```
JS sees a `{` at the start of a statement and assumes it's a **code block**, not destructuring.
Fix: wrap in parentheses so JS knows it's an expression:
```js
({title, width, height} = {...}); // ✓ works
```

#### -->Nested destructuring
```js
let options = {
  size: { width: 100, height: 200 },
  items: ["Cake", "Donut"]
};

let {
  size: { width, height },
  items: [item1, item2]
} = options;

alert(width);  // 100
alert(item1);  // Cake
```
The left-side pattern **literally mirrors the structure** of the right side — nested object → nested `{}` pattern, nested array → nested `[]` pattern. You're not creating a `size` or `items` variable at all 

####  -->smart function parameters

This is what you'll actually use every single day in React. The problem:
```js
function showMenu(title = "Untitled", width = 200, height = 100, items = []) { ... }

showMenu("My Menu", undefined, undefined, ["Item1", "Item2"]); // ugly, error-prone
```
With many optional params, you have to remember exact **order**, and pass `undefined` as filler for anything you want to skip — genuinely bad API design.

**Fix**: pass a single object, and destructure it right in the function signature:
```js
function showMenu({ title = "Untitled", width = 200, height = 100, items = [] }) {
  alert(`${title} ${width} ${height}`);
  alert(items);
}

showMenu({ title: "My menu", items: ["Item1", "Item2"] }); // order doesn't matter, skip anything!
```
This is **exactly** the pattern you'll see in every React component:


```js
function showMenu({ title = "Menu", width = 100, height = 200 } = {}) { ... }
showMenu(); // now works fine — Menu 100 200
```
That trailing `= {}` means "if no argument at all was passed, pretend an empty object was passed" — giving the inner destructuring something safe to work with.


---
## -->Date and time

#### -->Creating a Date — 4 ways

**1. No arguments — current date/time**
```js
let now = new Date();
alert(now); // shows current date/time
```

**2. Milliseconds since Jan 1, 1970 UTC — the timestamp
```js
let Jan01_1970 = new Date(0);
let Jan02_1970 = new Date(24 * 3600 * 1000); // add 24hrs worth of ms
```
This number — **milliseconds since Jan 1, 1970 UTC** — is called a **timestamp**. It's the universal, lightweight way computers represent a specific moment in time as just one number. Extremely common across programming languages 

Negative timestamps = dates before 1970:
```js
let Dec31_1969 = new Date(-24 * 3600 * 1000);
```

**3. From a string**
```js
let date = new Date("2017-01-26");
```
JS auto-parses common date-string formats. Worth noting: if no time is specified, midnight UTC is assumed, then displayed adjusted to _your_ local timezone — so the exact hour shown can vary depending on where you're running the code.

**4. From individual components**
```js
new Date(year, month, date, hours, minutes, seconds, ms)

new Date(2011, 0, 1, 0, 0, 0, 0); // Jan 1, 2011, 00:00:00
new Date(2011, 0, 1);             // same — trailing args default to 0
```

- **Month is 0-indexed**: `0` = January, `11` = December. This trips up literally everyone at least once.
- Only year + month are required; everything else defaults sensibly (day → 1, time → 0).

#### Reading date components — the get methods

| Method                                                            | Returns                                    |
| ----------------------------------------------------------------- | ------------------------------------------ |
| `getFullYear()`                                                   | 4-digit year                               |
| `getMonth()`                                                      | 0–11                                       |
| `getDate()`                                                       | day of month, 1–31                         |
| `getHours()`, `getMinutes()`, `getSeconds()`, `getMilliseconds()` | time components                            |
| `getDay()`                                                        | day of **week**, 0 (Sunday) – 6 (Saturday) |
never use `getYear()` — it's deprecated and returns weird 2-digit values sometimes. Always `getFullYear()`.
Also notice: `getDate()` gets the **day of month** ( you'd expect it to return the whole date), while `getDay()` gets the **day of week**. 


```js
date.getHours();    // local timezone
date.getUTCHours();  // UTC+0 (London time, no daylight savings)
```
Every get method above has a `getUTC*` twin — same data, but relative to UTC+0 instead of your local timezone. Matters when your app has users across timezones and you need a consistent reference point 
#### -->Two special methods with no UTC twin
```js
date.getTime();           // the timestamp — ms since Jan 1 1970 UTC+0
date.getTimezoneOffset(); // difference between UTC and your local time, in minutes
```
`getTimezoneOffset()` example: if you're in UTC-1, this returns `60`; if UTC+3, returns `-180`. Niche, but useful when doing timezone math manually.

##### Setting date components — the set methods
```js
setFullYear(year, [month], [date])
setMonth(month, [date])
setDate(date)
setHours(hour, [min], [sec], [ms])
setMinutes(min, [sec], [ms])
setSeconds(sec, [ms])
setMilliseconds(ms)
setTime(milliseconds)
```
Each has a UTC-variant too (`setUTCHours()`, etc.) except `setTime()`.

```js
let today = new Date();
today.setHours(0);       // only the hour changes — date, minutes stay same
today.setHours(0,0,0,0); // sets hour, min, sec, ms all to 0 → midnight exactly
```

```js
let date = new Date(2013, 0, 32); // "32 Jan"?! doesn't exist
alert(date); // → 1st Feb 2013 (auto-corrected!)
```
**Why this matters practically**: it means you never have to manually handle month-length or leap-year edge cases yourself:

j
```js
let date = new Date(2016, 1, 28); // 28 Feb 2016 (leap year)
date.setDate(date.getDate() + 2);
alert(date); // 1 Mar 2016 — correctly handled the leap year Feb 29th automatically
```
You just say add 2 days via `getDate() + 2`, and Date figures out the rollover into March itself 
```js
let date = new Date();
date.setSeconds(date.getSeconds() + 70); // correctly rolls into next minute if needed
```
```js
let date = new Date(2016, 0, 2); // 2 Jan 2016
date.setDate(0); // day "0" = the day BEFORE day 1 = last day of PREVIOUS month
alert(date); // 31 Dec 2015
```

#### -->Date → Number conversion, and date subtraction

```js
let date = new Date();
alert(+date); // same as date.getTime() — the timestamp
```
Converting a Date to a number gives its timestamp. And since subtraction always forces number conversion:

```js
let start = new Date();
// ...do some work...
let end = new Date();

alert(`The loop took ${end - start} ms`); // subtracting dates = ms difference
```
This is genuinely useful — the standard way to measure elapsed time in JS.

#### -->`Date.now()` 
```js
let start = Date.now(); // just the timestamp number directly
// ...work...
let end = Date.now();
alert(`${end - start} ms`);
```
Same result as `new Date().getTime()`, but skips creating an actual `Date` object — faster, and doesn't create extra garbage for the GC to clean up later 

#### -->Benchmarking section

comparing two ways of getting the difference between two dates.
```js
function diffSubtract(date1, date2) {
  return date2 - date1; // relies on auto object-to-primitive conversion
}

function diffGetTime(date1, date2) {
  return date2.getTime() - date1.getTime(); // explicit, no conversion needed
}
```
Both produce the same result. Question: which is faster?

**First  attempt** — just run each 100,000 times and time it:
```js
function bench(f) {
  let date1 = new Date(0);
  let date2 = new Date();
  let start = Date.now();
  for (let i = 0; i < 100000; i++) f(date1, date2);
  return Date.now() - start;
}

alert('Time of diffSubtract: ' + bench(diffSubtract) + 'ms');
alert('Time of diffGetTime: ' + bench(diffGetTime) + 'ms');
```
Result: `diffGetTime` (explicit `.getTime()`) is noticeably faster — because it skips the implicit object-to-primitive conversion step entirely, which is extra work for the engine.

```js
let time1 = 0, time2 = 0;

for (let i = 0; i < 10; i++) {
  time1 += bench(diffSubtract);
  time2 += bench(diffGetTime);
}
```

Why? Modern JS engines (like V8, which runs Node and Chrome) apply optimizations only to code that's run enough times to be considered worth optimizing (called "hot code"). The very first few runs of any function are slower because the engine hasn't optimized them yet. Running each function once beforehand "warms it up" so your actual measurement reflects steady-state performance, not artificially slow first-run behavior.

Microbenchmarking (measuring tiny operations) is genuinely tricky — engines optimize artificial test code differently than real-world usage patterns, so tiny benchmark results can be misleading. If performance really matters to you, understanding _how the engine works_ generally beats obsessing over tiny benchmark numbers.


#### -->`Date.parse(str)` 

```js
Date.parse('2012-01-26T13:51:50.417-07:00'); // returns a timestamp number
```
Expected format: `YYYY-MM-DDTHH:mm:ss.sssZ`

- `YYYY-MM-DD` = year-month-day
- `T` = literal delimiter character separating date from time
- `HH:mm:ss.sss` = hours:minutes:seconds.milliseconds
- `Z` (or `+hh:mm`) = timezone; a bare `Z` means UTC+0

Shorter forms also valid: just `YYYY-MM-DD`, or even just `YYYY`.
Invalid format → returns `NaN`, not an error.

```js
let date = new Date(Date.parse('2012-01-26T13:51:50.417-07:00'));
```
`Date.parse` combined directly with `new Date(...)` like this to build a proper Date object from a string.


---
## -->JSON methods


```js
let user = {
  name: "John",
  age: 30,
  toString() {
    return `{name: "${this.name}", age: ${this.age}}`;
  }
};
```
Works, but it's a maintenance headache — every time you add/rename/remove a property, you have to go update this string manually. 

#### -->What JSON actually is

**JSON (JavaScript Object Notation)** is a **universal, language-independent** text format for representing data. It was born from JS syntax, but literally every major language (Python, Java, Ruby, PHP) has libraries to read/write it — which is exactly why it's the standard format for **client ↔ server communication**. 

#### -->`JSON.stringify` — object → string
```js
let student = {
  name: 'John',
  age: 30,
  isAdmin: false,
  courses: ['html', 'css', 'js'],
  spouse: null
};

let json = JSON.stringify(student);

alert(typeof json); // "string"
alert(json);
/*
{
  "name": "John",
  "age": 30,
  "isAdmin": false,
  "courses": ["html", "css", "js"],
  "spouse": null
}
*/
```
1. Strings **must** use double quotes — `'John'` becomes `"John"`, no single quotes, no backticks, ever.
2. Property **names/keys** must also be double-quoted — `age` becomes `"age"`. 

JSON actually supports
- Objects `{...}`
- Arrays `[...]`
- Primitives: strings, numbers, booleans, `null`

```js
JSON.stringify(1);      // "1"
JSON.stringify('test'); // "\"test\""  (a quoted string)
JSON.stringify(true);   // "true"
JSON.stringify([1,2,3]); // "[1,2,3]"
```

```js
let user = {
  sayHi() { alert("Hello"); },  // function — DROPPED
  [Symbol("id")]: 123,           // symbol key — DROPPED
  something: undefined           // undefined value — DROPPED
};
alert(JSON.stringify(user)); // {} — completely empty!
```

**Three things JSON.stringify ignores entirely**:
1. **Functions/methods** — JSON is a _data-only_ format, it has no concept of code, so any method just vanishes.
2. **Symbol keys and values** 
3. **Properties whose value is `undefined`** — because JSON has no `undefined` type at all (only `null` represents nothing in JSON).

```js
let meetup = {
  title: "Conference",
  room: { number: 23, participants: ["john", "ann"] }
};
JSON.stringify(meetup);
// {"title":"Conference","room":{"number":23,"participants":["john","ann"]}}
```
This is genuinely the whole point — unlike  manual `toString()` approach, you don't have to write any special logic for nesting. `JSON.stringify` recursively walks the entire structure, however deep, automatically.

##### -->The hard limitation — circular references
```js
let room = { number: 23 };
let meetup = { title: "Conference", participants: ["john", "ann"] };

meetup.place = room;        // meetup → room
room.occupiedBy = meetup;   // room → meetup (circular!)

JSON.stringify(meetup); //  Error: Converting circular structure to JSON
```
 `JSON.stringify` tries to walk through the entire object recursively, and it would literally **never finish** — `meetup → room → meetup → room → ...` forever. 

#### -->The `replacer` argument 

Full signature: `JSON.stringify(value, replacer, space)`.
Form 1: `replacer` as an ARRAY — a simple whitelist
```js
let user = { name: "John", age: 30, password: "secret123" };
JSON.stringify(user, ['name', 'age']);
// {"name":"John","age":30}
```
`password` isn't in the array, so it's excluded. Simple.

###### The trap: it applies to EVERY level, not just the top
```js
let meetup = {
  title: "Conference",
  participants: [{name: "John"}, {name: "Alice"}]
};
JSON.stringify(meetup, ['title', 'participants']);
// {"title":"Conference","participants":[{},{}]}
```
Why did `participants` become `[{},{}]` — empty objects?? Because the whitelist `['title', 'participants']` only allows keys named `title` or `participants` **anywhere in the structure**. When JS gets down to the objects _inside_ `participants` (`{name: "John"}`), it checks: "is `name` in my whitelist?" — No. So `name` gets dropped too, leaving empty `{}`.

##### Form 2: `replacer` as a FUNCTION

This is the powerful, flexible version. You write a function, and JSON.stringify calls it **once for every single key-value pair it encounters** — including nested ones — right before deciding what to actually write into the output string.

```js
function replacer(key, value) {
  // you decide what happens to THIS pair
  return value; // "keep it as-is"
}
```

**The contract**: whatever you `return` from this function becomes the actual value used in the output JSON.
- Return the value unchanged → keep it normally
- Return a _different_ value → substitute it
- Return `undefined` → **delete this property entirely** from the output

```js
let room = { number: 23 };
let meetup = {
  title: "Conference",
  participants: [{name: "John"}, {name: "Alice"}],
  place: room
};
room.occupiedBy = meetup; // circular!

JSON.stringify(meetup, function replacer(key, value) {
  return (key == 'occupiedBy') ? undefined : value;
});
```
JSON.stringify walks the whole object tree, and for **every** key it finds, it calls `replacer(key, value)` first, before deciding whether to include that key in the output:

##### -->The `space` argument
```js
JSON.stringify(user, null, 2);
/*
{
  "name": "John",
  "age": 25,
  ...
}
*/
```
`space` controls indentation for human-readable output  


```js
let room = {
  number: 23,
  toJSON() {
    return this.number;
  }
};
JSON.stringify(room); // "23" — not {"number":23}, just the plain value!
```
Works both standalone AND when nested inside another object being stringified — the nested `room` still gets simplified down to just `23` wherever it appears. 

#### -->`JSON.parse` — the reverse: string → object

```js
let numbers = "[0, 1, 2, 3]";
numbers = JSON.parse(numbers);
alert(numbers[1]); // 1 — now a real array, not a string
```

```js
let userData = '{ "name": "John", "age": 35, "friends": [0,1,2,3] }';
let user = JSON.parse(userData);
alert(user.friends[1]); // 1
```
This is what you'll use constantly when receiving data from a server — API responses typically come back as JSON text, and `JSON.parse` turns that text into an actual usable JS object/array.

common mistakes...
```js
let json = `{
  name: "John",                     // ❌ key not quoted
  "surname": 'Smith',               // ❌ single quotes in value
  'isAdmin': false                  // ❌ single quotes in key
  "birthday": new Date(2000, 2, 3), // ❌ "new Date(...)" not allowed — JSON only allows bare/literal values
  "friends": [0,1,2,3]              // ✓ fine
}`;
```
Also: **JSON does not support comments at all** — adding a `//` comment makes the whole thing invalid JSON. (There's an informal extended format called JSON5 that allows this, but it's a separate library, not the actual JSON standard.)
##### -->`reviver` — the `JSON.parse` equivalent of `replacer`

Real problem: dates come back from JSON as **plain strings**, not actual `Date` objects — since JSON has no native date type (this is exactly why `Date.toJSON()` exists, to turn a Date into a string in the first place).
```js
let str = '{"title":"Conference","date":"2017-11-30T12:00:00.000Z"}';
let meetup = JSON.parse(str);

meetup.date.getDate(); //  Error! meetup.date is just a string, not a real Date
```

Fix with a **reviver function**, second argument to `JSON.parse`:
```js
let meetup = JSON.parse(str, function(key, value) {
  if (key == 'date') return new Date(value);
  return value;
});
meetup.date.getDate(); // ✓ works now — real Date object
```
Same idea as `replacer`, mirrored for parsing



