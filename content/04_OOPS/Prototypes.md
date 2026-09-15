

## -->Prototypal inheritance

Every JS object has a secret, hidden link to another object, called its **prototype**. 
> **When you try to read a property that doesn't exist directly on an object, JS automatically checks that object's prototype instead — before giving up.**


```js
let animal = { eats: true };
let rabbit = { jumps: true };

rabbit.__proto__ = animal; // link rabbit's prototype to animal

alert(rabbit.eats);  // true  — not on rabbit directly, found via prototype
alert(rabbit.jumps); // true  — found directly on rabbit
```
This is called **prototypal inheritance** — rabbit inherits from animal.

```js
let animal = {
  eats: true,
  walk() { alert("Animal walk"); }
};

let rabbit = { jumps: true, __proto__: animal };
rabbit.walk(); // "Animal walk" — found via prototype, not directly on rabbit
```

```js
let animal = { eats: true, walk() { alert("Animal walk"); } };
let rabbit = { jumps: true, __proto__: animal };
let longEar = { earLength: 10, __proto__: rabbit };

longEar.walk();       // "Animal walk" — found by walking UP the chain: longEar →                               rabbit → animal
alert(longEar.jumps); // true — found on rabbit
```
If a property isn't found directly, JS climbs the chain: `longEar` → `rabbit` → `animal` 

#### -->Two hard rules
1. **No circular chains** — you can't make `A`'s prototype be `B`, and `B`'s prototype be `A`
2. **`__proto__` can only be an object or `null`** — anything else (a number, string, etc.) is silently ignored.
3. Only ONE prototype per object — you can't inherit from two objects at once, unlike some other languages that support multiple inheritance.

#### -->`__proto__` vs `[[Prototype]]`

The actual internal mechanism is called `[[Prototype]]`. `__proto__` is just a **historical getter/setter** — a convenient handle that lets you read/write `[[Prototype]]` from regular code. Modern JS actually prefers `Object.getPrototypeOf()`/`Object.setPrototypeOf()` instead — `__proto__` still works everywhere in practice, just considered slightly old-fashioned syntax.

##### Critical rule: prototypes are ONLY used for reading, never writing
```js
let animal = {
  eats: true,
  walk() { /* this won't be used by rabbit */ }
};

let rabbit = { __proto__: animal };
rabbit.walk = function() {
  alert("Rabbit! Bounce-bounce!");
};
rabbit.walk(); // "Rabbit! Bounce-bounce!" — rabbit's OWN walk, not animal's
```
**Prototype chain lookup only kicks in when a property is missing** — writing always happens on the object you're directly writing to.

#### -->Exception: getters/setters DO use the prototype, because they're functions in disguise
```js
let user = {
  name: "John",
  surname: "Smith",
  set fullName(value) { [this.name, this.surname] = value.split(" "); },
  get fullName() { return `${this.name} ${this.surname}`; }
};

let admin = { __proto__: user, isAdmin: true };

alert(admin.fullName);        // "John Smith" — getter found via prototype, runs
admin.fullName = "Alice Cooper"; // setter found via prototype, runs
alert(admin.fullName);        // "Alice Cooper" — admin changed
alert(user.fullName);         // "John Smith" — user UNCHANGED
```
Why doesn't this contradict writing doesn't use the prototype? Because a setter isn't really writing a value— it's **calling a function** (the setter), which JS _does_ find via the prototype chain like any method. The function then decides what to actually do — here it sets `this.name`/`this.surname`, and since `this` = `admin` ), those changes land on `admin`, not `user`.

```js
let animal = {
  walk() { if (!this.isSleeping) alert(`I walk`); },
  sleep() { this.isSleeping = true; }
};

let rabbit = { name: "White Rabbit", __proto__: animal };
rabbit.sleep();
alert(rabbit.isSleeping); // true
alert(animal.isSleeping); // undefined — animal untouched!
```
**`this` = whatever is before the dot at the moment of the call.** Here, the call is `rabbit.sleep()`  Doesn't matter one bit that the function's _code_ physically lives on `animal` — `this` is decided by **how you called it**, not **where it was defined**.
So inside `sleep()`, `this` = `rabbit`.

**Why this matters practically**: this is what makes prototypal inheritance actually useful for real object-oriented code — you can have `animal`, `bird`, `snake` all share the _same_ `walk`/`sleep` methods (one copy in memory), while each individual object keeps its _own_ separate state (`isSleeping`, `name`, etc.) without them stepping on each other.


```js
let animal = { eats: true };
let rabbit = { jumps: true, __proto__: animal };

Object.keys(rabbit); // ["jumps"] — own properties only
for (let prop in rabbit) alert(prop); // "jumps", then "eats" — includes inherited!
```
`for...in` walks the **entire prototype chain**, while `Object.keys()`/`Object.values()`/`Object.entries()` only look at the object's **own** properties. 

#### -->Filtering with `hasOwnProperty`
```js
for (let prop in rabbit) {
  let isOwn = rabbit.hasOwnProperty(prop);
  if (isOwn) alert(`Our: ${prop}`);      // "Our: jumps"
  else alert(`Inherited: ${prop}`);       // "Inherited: eats"
}
```
`hasOwnProperty(key)` tells you: "does this object have this property **directly**, not via the prototype chain?"
**The fun detail buried here**: where does `rabbit.hasOwnProperty` itself even come from? You never defined it! Turns out — it's inherited too, from `Object.prototype`, which sits at the very top of essentially every plain object's prototype chain by default (`rabbit → animal → Object.prototype → null`).


---
## -->F.prototype

- **`[[Prototype]]`** — the hidden internal link every object has (what `__proto__` reads/writes). This is what actually matters for property lookup.
- **`F.prototype`** — just a **normal, regular property** that happens to be named `"prototype"`, sitting on **functions specifically**. 

#### The rule that ties them together

> **When you call `new F()`, JS takes whatever object is sitting in `F.prototype` and uses it to set the new object's `[[Prototype]]`.**

```js
let animal = { eats: true };
function Rabbit(name) {
  this.name = name;
}

Rabbit.prototype = animal; // "when 'new Rabbit()' runs, link new objects to animal"
let rabbit = new Rabbit("White Rabbit");
alert(rabbit.eats); // true — rabbit.__proto__ === animal now
```
So `Rabbit.prototype = animal` is basically saying: every future object built with `new Rabbit()` should have its `[[Prototype]]` set to `animal`. 
##### `F.prototype` is only checked AT the moment `new` runs
```js
function Rabbit() {}
Rabbit.prototype = { eats: true };
let rabbit1 = new Rabbit(); // gets THIS prototype
Rabbit.prototype = { jumps: true }; // change it AFTER
let rabbit2 = new Rabbit(); // gets the NEW prototype
```
`rabbit1` and `rabbit2` end up with **different** prototypes, because each `new Rabbit()` call snapshots whatever `Rabbit.prototype` happens to be **at that exact moment**. 

```js
function Rabbit() {}
// Rabbit.prototype = { constructor: Rabbit }  ← this exists automatically
```
Every function, the moment it's defined, automatically gets a `.prototype` property pointing to a plain object — and that object has exactly **one** property by default: `constructor`, which points **back to the function itself**.

```js
alert(Rabbit.prototype.constructor == Rabbit); // true
```
And since `new Rabbit()` links the new object's `[[Prototype]]` to this default object, **every instance inherits `.constructor`** automatically too:

```js
let rabbit = new Rabbit();
alert(rabbit.constructor == Rabbit); // true — found via the prototype chain
```

```js
function Rabbit(name) {
  this.name = name;
}
let rabbit = new Rabbit("White Rabbit");
let rabbit2 = new rabbit.constructor("Black Rabbit");
```
This is a genuinely handy pattern: if you have an object but **don't directly know** what constructor made it... `obj.constructor` lets you find and reuse the _same_ constructor to make another one of the same kind, without needing to know its name explicitly.

```js
function Rabbit() {}
Rabbit.prototype = { jumps: true }; // completely REPLACED the default prototype object

let rabbit = new Rabbit();
alert(rabbit.constructor === Rabbit); // false !!
```
Why? Because you didn't **add** to the default prototype — you **replaced it entirely** with a brand new plain object, `{ jumps: true }`, which never had a `constructor` property in the first place .

#### -->Two ways to avoid breaking it

**Option 1 — add to the prototype instead of replacing it wholesale:**
```js
function Rabbit() {}
Rabbit.prototype.jumps = true; // adds ONE property, leaves default constructor intact
```

**Option 2 — if you do replace it, manually re-add `constructor` yourself:**
```js
Rabbit.prototype = {
  jumps: true,
  constructor: Rabbit // manually restored
};
```


---
## -->Native prototypes

Remember way back, when a plain object mysteriously gave you `"[object Object]"` from `toString()`
```js
let obj = {};
alert(obj); // "[object Object]"
```
The trick: `{}` is secretly shorthand for `new Object()` 
```js
let obj = {};
alert(obj.__proto__ === Object.prototype); // true
alert(obj.toString === Object.prototype.toString); // true
```
So `obj.toString()` works via the **exact same mechanism** as `rabbit.walk()` did 

```js
alert(Object.prototype.__proto__); // null
```
`Object.prototype` sits at the very **top** of the chain — nothing above it. This is the `null` that ends every prototype chain you build.

```js
let arr = [1, 2, 3]; // secretly like new Array(1,2,3)

alert(arr.__proto__ === Array.prototype);          // true
alert(arr.__proto__.__proto__ === Object.prototype); // true
alert(arr.__proto__.__proto__.__proto__);           // null
```
This is huge for understanding something you use constantly: **`.map()`, `.filter()`, `.push()`, `.forEach()`** — none of these live on your individual array. They all live **once**, on `Array.prototype`, shared across every array that has ever existed in your program. 

**The chain**: `arr → Array.prototype → Object.prototype → null`. Every array inherits from `Array.prototype` first, and `Array.prototype` itself inherits from `Object.prototype` — this is literally where everything inherits from object comes from as a saying.

```js
let arr = [1, 2, 3];
alert(arr); // "1,2,3" — NOT "[object Object]"
```
Both `Array.prototype` **and** `Object.prototype` have their own `toString`. Since JS walks the chain from the object **outward/upward**, and stops at the **first match it finds**, `Array.prototype`'s version  wins, because it's **closer** in the chain than `Object.prototype`'s version. T

#### -->Functions are also objects, with their own prototype chain
```js
function f() {}
alert(f.__proto__ == Function.prototype);          // true
alert(f.__proto__.__proto__ == Object.prototype);  // true
```
functions themselves are objects in JS . `Function.prototype` is where methods like `.call()`, `.apply()`, `.bind()` actually live — every single function you've ever written inherits these automatically through this exact chain.

#### -->Primitives — the weird, clever trick

This is genuinely one of the more mind-bending JS details. You know strings/numbers/booleans are **primitives**, not objects. So how does `"hello".toUpperCase()` work?
**The trick**: the instant you try to access a property/method on a primitive, JS **secretly, temporarily** wraps it in an object (using `String`/`Number`/`Boolean` constructors), lets you use the method, then **immediately discards** that temporary wrapper object.
```js
"BOOM".toUpperCase(); 
// secretly, momentarily: (new String("BOOM")).toUpperCase(), then wrapper is thrown away
```
This is invisible to you 
**The one true exception**: `null` and `undefined` have **no** wrapper objects, **no** prototypes, nothing. T


---
## -->Prototype methods, objects without __proto__

##### -->The modern replacements for `__proto__`
```js
Object.getPrototypeOf(obj);       // read the prototype
Object.setPrototypeOf(obj, proto); // set the prototype
```
Same job as `obj.__proto__` (read) and `obj.__proto__ = x` (write)

```js
let animal = { eats: true };
let rabbit = Object.create(animal); // same as {__proto__: animal}
alert(rabbit.eats); // true
```
This does the same thing as `{ __proto__: animal }`, just as a dedicated function. It also accepts an optional second argument for defining extra properties with fine-grained control


```js
let clone = Object.create(
  Object.getPrototypeOf(obj),
  Object.getOwnPropertyDescriptors(obj)
);
```
This makes a _more complete_ copy than spread/`Object.assign` (shallow) — it preserves getters/setters and the prototype too. Niche, but good to know it exists if you ever need a truly faithful clone.


#####  the `__proto__` dictionary bug

This is the one section worth understanding properly, because it's a **real, subtle bug class** you could genuinely hit once you're handling user input (forms, API payloads ..)

```js
let obj = {};

let key = "__proto__"; // imagine this came from user input
obj[key] = "some value";

alert(obj[key]); // "[object Object]" — NOT "some value"!!
```
**Why this breaks**: `__proto__` isn't a normal property — remember, it's a **special getter/setter** that reads/writes `[[Prototype]]`. So `obj["__proto__"] = "some value"` doesn't create a regular key called `"__proto__"` 

**Why this is genuinely dangerous, not just quirky**: if you're building something like a dictionary/config object from **user-controlled keys** (e.g., a form where users can name their own fields, or JSON data from a client), and someone (accidentally or maliciously) uses the key `"__proto__"`, you could end up:
- Silently losing data (like above), or
- Worse — if the value they're setting genuinely IS an object, you could actually **overwrite the object's real prototype**, causing bizarre, hard-to-trace behavior elsewhere in your app. This is a known, real security concern called prototype pollution — you'll see this term in security contexts around Node/Express apps specifically.

#### -->The fix — `Object.create(null)`
```js
let obj = Object.create(null); // an object with NO prototype at all

let key = "__proto__";
obj[key] = "some value";

alert(obj[key]); // "some value" — works correctly now!
```
`Object.create(null)` builds an object whose `[[Prototype]]` is **`null`** — meaning there's no `Object.prototype` anywhere in its chain, which means there's no special `__proto__` getter/setter inherited at all. So `"__proto__"` behaves like any other completely ordinary key

 
 **if you genuinely need a safe, arbitrary user-key dictionary, Map is usually the better/simpler tool**, since it never had this `__proto__` problem to begin with ..
```js
let map = new Map();
map.set("__proto__", "some value");
map.get("__proto__"); // "some value" — totally fine, no special-casing
```



