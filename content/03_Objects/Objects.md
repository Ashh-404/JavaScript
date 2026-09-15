Includes objects basics , Object references and copying , Garbage collection , Object Methods

---
An object can be created with curly braces `{…}` with an optional list of _properties_. A property is a key: value pair, where `key` is a string , and `value` can be anything.
We can imagine an object as a cabinet with signed files. Every piece of data is stored in its file by the key. It’s easy to find a file by its name or add/remove a file.

An empty object  can be created using one of two syntaxes:
```javascript
let user = new Object(); // "object constructor" syntax
let user = {};  // "object literal" syntax
```
Usually, the curly braces `{...}` are used.

### -->Literals and properties
A **property** is a key-value pair that belongs to an object. Once you've created the object, each of its key-value entries is called a property.
```javascript
let user = {     // an object
  name: "John",  // by key "name" store value "John"
  age: 30        // by key "age" store value 30
};
```
A property has a key  before the colon  and a value to the right of it.

Property values are accessible using the dot notation:
```javascript
// get property values of the object:
alert( user.name ); // John
alert( user.age ); // 30
```

The value can be of any type. Let’s add a boolean one:
```javascript
user.isAdmin = true;
```

To remove a property, we can use the `delete` operator:
```javascript
delete user.age;
```

We can also use multiword property names, but then they must be quoted:
The last property in the list may end with a comma:
That is called a trailing or hanging comma. Makes it easier to add/remove/move around properties, because all lines become alike.

### Square brackets

For multiword properties, the dot access doesn’t work:
```javascript
// this would give a syntax error
user.likes birds = true
```
JavaScript doesn’t understand that. It thinks that we address `user.likes`, and then gives a syntax error when comes across unexpected `birds`.

- There’s an alternative square bracket notation that works with any string..
- Square brackets also provide a way to obtain the property name as the result of any expression – as opposed to a literal string – like from a variable..
```javascript
let user = {};
// set
user["likes birds"] = true;
// get
alert(user["likes birds"]); // true
// delete
delete user["likes birds"];

// other method
let key = "likes birds";
// same as user["likes birds"] = true;
user[key] = true;
```


### -->Computed properties

We can use square brackets in an object literal, when creating an object. That’s called _computed properties_.
```javascript
let fruit = prompt("Which fruit to buy?", "apple");

let bag = {
  [fruit]: 5, // the name of the property is taken from the variable fruit
};

alert( bag.apple ); // 5 if fruit="apple"
```
The meaning of a computed property is simple: `[fruit]` means that the property name should be taken from `fruit`.
So, if a visitor enters apple, `bag` will become `{apple: 5}`.

Essentially, that works the same as:
```javascript
let fruit = prompt("Which fruit to buy?", "apple");
let bag = {};
// take property name from the fruit variable
bag[fruit] = 5;
```
…But looks nicer.

We can use more complex expressions inside square brackets:
```javascript
let fruit = 'apple';
let bag = {
  [fruit + 'Computers']: 5 // bag.appleComputers = 5
};
```
Square brackets are much more powerful than dot notation. They allow any property names and variables. But they are also more cumbersome to write.
### Property value shorthand
In real code, we often use existing variables as values for property names.
```javascript
function makeUser(name, age) {
  return {
    name: name,
    age: age,
    // ...other properties
  };
}
let user = makeUser("John", 30);
alert(user.name); // John
```

In the example above, properties have the same names as variables. The use-case of making a property from a variable is so common, that there’s a special _property value shorthand_ to make it shorter.
Instead of `name:name` we can just write `name`..
We can use both normal properties and shorthands in the same object:

### -->Property names limitations
As we already know, a variable cannot have a name equal to one of the language-reserved words like for, let, return etc.
But for an object property, there’s no such restriction:
```javascript
// these properties are all right
let obj = {
  for: 1,
  let: 2,
  return: 3
};
alert( obj.for + obj.let + obj.return );  // 6
```

```javascript
let obj = {
  0: "test" // same as "0": "test"
};
// both alerts access the same property (the number 0 is converted to string "0")
alert( obj["0"] ); // test
alert( obj[0] ); // test (same property)
```

There’s a minor gotcha with a special property named `__proto__`. We can’t set it to a non-object value:
```javascript
let obj = {};
obj.__proto__ = 5; // assign a number
alert(obj.__proto__); // [object Object] - the value is an object, didn't work as intended
```


### Property existence test, “in” operator

A notable feature of objects in JavaScript is that it’s possible to access any property. There will be no error if the property doesn’t exist!
Reading a non-existing property just returns `undefined`. So we can easily test whether the property exists:
```javascript
let user = {};

alert( user.noSuchProperty === undefined ); // true means "no such property"
```

There’s also a special operator `"in"` for that..
```javascript
let user = { name: "John", age: 30 };
alert( "age" in user ); // true, user.age exists
alert( "blabla" in user ); // false, user.blabla doesn't exist
```
Why does the `in` operator exist? Isn’t it enough to compare against `undefined`?
```javascript
let obj = {
  test: undefined
};
alert( obj.test ); // it's undefined, so - no such property?
alert( "test" in obj ); // true, the property does exist!
```

---
### -->The "for..in" loop
To walk over all keys of an object, there exists a special form of the loop: `for..in`. This is a completely different thing from the `for(;;)`
```javascript
for (key in object) {
  // executes the body for each key among object properties
}

let user = {
  name: "John",
  age: 30,
  isAdmin: true
};
for (let key in user) {
  alert( key );  // name, age, isAdmin
  alert( user[key] ); // John, 30, true
}
```

```javascript
let codes = {
  "49": "Germany",
  "41": "Switzerland",
  "44": "Great Britain",
  // ..,
  "1": "USA"
};
for (let code in codes) {
  alert(code); // 1, 41, 44, 49
}
```
The phone codes go in the ascending sorted order, because they are integers. So we see `1, 41, 44, 49`.
The “integer property” term here means a string that can be converted to-and-from an integer without a change.
```javascript
// Number(...) explicitly converts to a number
// Math.trunc is a built-in function that removes the decimal part
alert( String(Math.trunc(Number("49"))) ); // "49", same, integer property
alert( String(Math.trunc(Number("+49"))) ); // "49", not same "+49" ⇒ not integer property
alert( String(Math.trunc(Number("1.2"))) ); // "1", not same "1.2" ⇒ not integer property
```
…On the other hand, if the keys are non-integer, then they are listed in the creation order..
So, to fix the issue with the phone codes, we can “cheat” by making the codes non-integer. Adding a plus `"+"` sign before each code is enough.
```javascript
let codes = {
  "+49": "Germany",
  "+41": "Switzerland",
  "+44": "Great Britain",
  // ..,
  "+1": "USA"
};
for (let code in codes) {
  alert( +code ); // 49, 41, 44, 1
}
```

---
## Object references and copying

Here we put a copy of `message` into `phrase`:
```javascript
let message = "Hello!";
let phrase = message;
```
As a result we have two independent variables, each one storing the string `"Hello!"`.
When an object variable is copied, the reference is copied, but the object itself is not duplicated.
```javascript
let user = { name: "John" };
let admin = user; // copy the reference

admin.name = 'Pete'; // changed by the "admin" reference
alert(user.name); // 'Pete', changes are seen from the "user" reference
```
Now we have two variables, each storing a reference to the same object:
We can use either variable to access the object and modify its contents


### -->Comparison by reference

Two objects are equal only if they are the same object.
```javascript
let a = {};
let b = a; // copy the reference

alert( a == b ); // true, both variables reference the same object
alert( a === b ); // true
```

And here two independent objects are not equal, even though they look alike (both are empty)
```javascript
let a = {};
let b = {}; // two independent objects
alert( a == b ); // false
```
For comparisons like `obj1 > obj2` or for a comparison against a primitive `obj == 5`, objects are converted to primitives. 

Const objects can be modified(very imp)
```javascript
const user = {
  name: "John"
};
user.name = "Pete"; // (*)
alert(user.name); // Pete
```
In other words, the `const user` gives an error only if we try to set `user=...` as a whole.

### -->Cloning and merging, Object.assign

So, copying an object variable creates one more reference to the same object.
But what if we need to duplicate an object?
We can create a new object and replicate the structure of the existing one, by iterating over its properties and copying them on the primitive level.
```javascript
let user = {
  name: "John",
  age: 30
};

let clone = {}; // the new empty object
// let's copy all user properties into it
for (let key in user) {
  clone[key] = user[key];
}
// now clone is a fully independent object with the same content
clone.name = "Pete"; // changed the data in it
alert( user.name ); // still John in the original object
```

We can also use the method Object.assign..
```javascript
Object.assign(dest, ...sources)
```
- The first argument `dest` is a target object.
- Further arguments is a list of source objects.

```javascript
let user = { name: "John" };

let permissions1 = { canView: true };
let permissions2 = { canEdit: true };
// copies all properties from permissions1 and permissions2 into user
Object.assign(user, permissions1, permissions2);
// now user = { name: "John", canView: true, canEdit: true }
alert(user.name); // John
alert(user.canView); // true
alert(user.canEdit); // true
```

If the copied property name already exists, it gets overwritten:
```javascript
let user = { name: "John" };
Object.assign(user, { name: "Pete" });
alert(user.name); // now user = { name: "Pete" }
```
### -->Nested cloning

Until now we assumed that all properties of `user` are primitive. But properties can be references to other objects.
```javascript
let user = {
  name: "John",
  sizes: {
    height: 182,
    width: 50
  }
};
alert( user.sizes.height ); // 182
```
Now it’s not enough to copy `clone.sizes = user.sizes`, because `user.sizes` is an object, and will be copied by reference, so `clone` and `user` will share the same sizes:
```javascript
let clone = Object.assign({}, user);
alert( user.sizes === clone.sizes ); // true, same object
// user and clone share sizes
user.sizes.width = 60;    // change a property from one place
alert(clone.sizes.width); // 60, get the result from the other one
```
`Object.assign` only copies one level deep so for nested , it doesn't clone the inner object — it just copies the **reference**

### -->structuredClone

The call `structuredClone(object)` clones the `object` with all nested properties.
```javascript
let user = {
  name: "John",
  sizes: {
    height: 182,
    width: 50
  }
};
let clone = structuredClone(user);
alert( user.sizes === clone.sizes ); // false, different objects
// user and clone are totally unrelated now
user.sizes.width = 60;    // change a property from one place
alert(clone.sizes.width); // 50, not related
```
The `structuredClone` method can clone most data types, such as objects, arrays, primitive values.
It also supports circular references, when an object property references the object itself (directly or via a chain or references).
```javascript
let user = {};
// let's create a circular reference:
// user.me references the user itself
user.me = user;
let clone = structuredClone(user);
alert(clone.me === clone); // true
```
As you can see, `clone.me` references the `clone`, not the `user`! So the circular reference was cloned correctly as well.
Function properties aren’t supported.


---
## -->Garbage collection

Every time you make a variable, object, function — it takes up space in memory (RAM). If nothing ever cleaned that up, your program would eat more and more memory forever and crash. So something needs to delete stuff you're not using anymore.

In C, _you_ do this manually (`malloc`/`free`). In JS (like Python and Java), the language does it **automatically** for you — that automatic cleaner is called the **Garbage Collector (GC)**.

### -->Reachability
**If you can still get to a value by following references from your code, it stays. If you can't reach it anymore, it gets deleted.**
 Think of it like a family tree, or like folders on your computer. If a folder is linked from your Desktop, you can navigate to it — it exists to you. If you delete the shortcut and there's no other way to reach that folder, it's effectively lost — doesn't matter if the folder is technically still sitting on the hard drive somewhere, you can't get to it anymore. JS's GC eventually notices this and actually erases it for real.

### -->Roots — the starting points

Certain things are **always considered reachable**, no matter what. These are called **roots**:
- The function currently running, plus its local variables
- Any other functions currently in the call stack (functions that called this function, and so on)
- Global variables

Basically: anything your code can _currently, directly_ touch right now is a root. Everything else only counts as reachable if you can trace a path **from a root** to it, through references.

_Example_:
```js
let user = { name: "John" };
```
Here, `user` is a global variable (a root). It has a reference (an arrow) pointing to the object `{name: "John"}`. So that object is reachable — it stays in memory.

```js
user = null;
```
Now `user` doesn't point to that object anymore. Nothing else points to it either. So that `{name: "John"}` object is now **unreachable** — even though it technically still exists somewhere in memory for a split second — and the GC will eventually clean it up and free that memory.

##### -->Example 2 — two references to the same thing
```js
let user = { name: "John" };
let admin = user;
```
Now **two** variables point to the _same_ object. Not two copies — literally the same object in memory, two names for it.
```js
user = null;
```
Object is still reachable — through `admin`. Only when you also do `admin = null` does the object become truly unreachable and get cleaned up.
**This is the core lesson**: it's not about how many references _used_ to exist, it's about whether **any path from a root still exists right now**.

##### Example 3 — interlinked objects
```js
function marry(man, woman) {
  woman.husband = man;
  man.wife = woman;
  return { father: man, mother: woman };
}

let family = marry({ name: "John" }, { name: "Ann" });
```
Now we have 3 objects: `family`, John, and Ann. And they reference each other:
- `family.father` → John
- `family.mother` → Ann
- John.wife → Ann
- Ann.husband → John
Everything's reachable right now because `family` is a root-connected global, and everything traces back to it.
```js
delete family.father;
delete family.mother.husband;
```
- We cut **two** specific links: `family` no longer points to John directly, AND Ann no longer points to John .
- Now trace it: can you reach John from any root? . No path exists to John from anywhere. So John gets garbage collected — even though John _still has_ a reference going out (`John.wife → Ann`).
- **Outgoing references from an object don't keep that object alive. Only incoming references (from something reachable) keep it alive.** 

##### Example 4 
```js
family = null;
```
Now think about it: John and Ann still reference _each other_ . But neither of them is reachable from any root anymore, because `family` — the only thing connecting this group to the outside world — is now `null`.

->**Analogy**: imagine two people stuck on an island, holding hands. Doesn't matter that they're connected to each other — if there's no boat/bridge connecting the island to the mainland, the mainland can't reachthem. From the mainland's perspective, they're gone.

### -->Mark-and-Sweep

This is the actual algorithm, and it's genuinely simple:
1. **Mark step**: Start at the roots. Mark them as alive.
2. Follow every reference from marked objects, and mark whatever they point to, too.
3. Keep following references outward, marking everything you touch
4. **Sweep step**: Once you can't reach anything new, look at _everything_ in memory. Anything **not marked** = unreachable = garbage. Delete it.

That's it. It's literally just start from what I know is real, follow every connection, and anything paint never touched gets thrown away."

### -->The optimizations 
Real JS engines add tricks to make this faster so your program doesn't freeze while cleaning up:
- **Generational collection**: Most objects die young (created, used briefly, discarded ). So the engine checks new objects often, and checks old survivors less often..
- **Incremental collection**: Instead of stopping everything to scan _all_ memory at once (which would freeze your app), the engine breaks the job into small chunks, doing a little bit at a time.
- **Idle-time collection**: The engine tries to do cleanup when the CPU isn't busy doing your actual code, so you don't notice any lag.

---
## -->Object methods

 A **method** is just the fancy name for **a function that lives inside an object as a property**.
```js
let user = {
  name: "John",
  age: 30
};
user.sayHi = function() {
  alert("Hello!");
};
user.sayHi(); // "Hello!"
```
That's it. `sayHi` is a normal function, just sitting inside `user` as a property. 


```js
// long way
user = {
  sayHi: function() {
    alert("Hello");
  }
};

// shorthand — does the exact same thing
user = {
  sayHi() {
    alert("Hello");
  }
};
```
Just drop the word `function` and the colon. Same behavior. 

#### -->`this` inside methods 

When a method needs to use data from its _own_ object, it uses `this`:
```js
let user = {
  name: "John",
  age: 30,
  sayHi() {
    alert(this.name); // "this" = the object before the dot
  }
};
user.sayHi(); // "John"
```
 **`this` = whatever is written before the dot when you call it.** Here, `user.sayHi()` → `this` = `user`.

##### Why not just write `user.name` directly instead of `this.name`?
```js
let user = {
  name: "John",
  age: 30,
  sayHi() {
    alert(user.name); // hardcoded "user", not "this"
  }
};

let admin = user;   // admin now points to the same object
user = null;        // user variable reset to null

admin.sayHi(); // ❌ TypeError! Cannot read property 'name' of null
```
Even though `admin` still correctly holds the object, `sayHi()` was hardcoded to look up the _variable_ `user` — and that variable got reset to `null`. So it crashes.

If we'd used `this.name` instead:
```js
sayHi() {
  alert(this.name); // ✓ works no matter what variable calls it
}
```
`this` doesn't care what variable name was used to call the method — it just means "whatever object is currently before the dot, right now." So `admin.sayHi()` correctly gives `this = admin`, no crash.

```js
function sayHi() {
  alert(this.name);
}
let user = { name: "John" };
let admin = { name: "Admin" };

user.f = sayHi;
admin.f = sayHi;

user.f();  // "John"   (this = user, since user is before the dot)
admin.f(); // "Admin"  (this = admin, since admin is before the dot)
```

```js
let user = {
  firstName: "Ilya",
  sayHi() {
    let arrow = () => alert(this.firstName);
    arrow();
  }
};

user.sayHi(); // "Ilya"
```
The arrow function `arrow()` doesn't get its own `this`. It just looks outward to whatever `this` was in `sayHi()` — which is `user`, since `user.sayHi()` was called with `user` before the dot. 

