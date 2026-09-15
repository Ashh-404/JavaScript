
```js
class User {
  constructor(name) {
    this.name = name;
  }

 sayHi() {
    alert(this.name);
  }
}
let user = new User("John");
user.sayHi(); // "John"
```
`constructor()` runs automatically when `new User(...)` is called 

```js
alert(typeof User); // "function"
```
A `class` isn't a new kind of thing in JS — it's **literally still a function under the hood**.:
1. Creates a function named `User` — its body = whatever's inside `constructor()`
2. Puts all the other methods (`sayHi`, etc.) onto `User.prototype`

```js
// class version
class User {
  constructor(name) { this.name = name; }
  sayHi() { alert(this.name); }
}

// manual equivalent — you could write this yourself right now
function User(name) {
  this.name = name;
}
User.prototype.sayHi = function() {
  alert(this.name);
};
```
Both produce nearly identical results.  called class "syntactic sugar" — same underlying mechanism (constructor function + shared prototype methods), just cleaner to write and read.

1. Classes must be called with `new`
```js
class User { constructor() {} }
User(); //  Error: Class constructor User cannot be invoked without 'new'
```

**2. Class methods are non-enumerable**
```js
for (let key in user) { ... } // sayHi does NOT show up here
```
Remember `for...in` walks the whole prototype chain, including inherited stuff? Class methods are automatically flagged so they're **excluded** from `for...in` . 

**3. Classes are automatically in strict mode**

#### -->Class Expressions 
```js
let User = class {
  sayHi() { alert("Hello"); }
};
```
Same idea as function expressions (`let f = function() {...}`) 


#### -->Getters/setters — same syntax as object literals, now inside a class
```js
class User {
  constructor(name) {
    this.name = name; // triggers the setter below
  }

 get name() {
    return this._name;
  }

  set name(value) {
    if (value.length < 4) {
      alert("Name is too short.");
      return;
    }
    this._name = value;
  }
}
```
class getters/setters end up on `User.prototype` too, exactly like regular methods do.
**Why `_name` instead of `name` inside**: if the setter were named `set name` but tried to do `this.name = value` inside itself, it'd call the setter again — infinite loop. 


#### -->Class fields
```js
class User {
  name = "John"; // class field

  sayHi() {
    alert(`Hello, ${this.name}!`);
  }
}
```
This is a relatively modern addition. The **critical difference** from methods: class fields are set **per-object** (directly on each instance), **not** on the shared prototype:
```js
let user = new User();
alert(user.name);           // "John" — on the instance
alert(User.prototype.name); // undefined — NOT on the prototype!
```
Class fields are **NOT shared** — each object gets its own independent copy..

```js
class Button {
  constructor(value) { this.value = value; }
  click() { alert(this.value); }
}
let button = new Button("hello");
setTimeout(button.click, 1000); // undefined — "this" got lost!
```
**Why this breaks**: remember the core `this` rule — it's decided by **what's before the dot at call time**. `setTimeout(button.click, ...)` extracts the `click` function itself, detached from `button` — it's no longer being called as `button.click()`, so by the time `setTimeout` actually invokes it, there's no dot at all, and `this` ends up as `undefined`  instead of `button`.

**The class-field fix**:
```js
class Button {
  constructor(value) { this.value = value; }
  click = () => {              // arrow function class field!
    alert(this.value);
  }
}
let button = new Button("hello");
setTimeout(button.click, 1000); // "hello" — works!
```
1. Class fields are created **per-object** (not shared on the prototype) — so `click` here is a genuinely separate function for each `Button` instance.
2. It's an **arrow function** —  arrow functions don't have their own `this`, they borrow it from their surrounding scope. 

---
## -->Class inheritance

Class inheritance is a way for one class to extend another class.

#### -->`extends`
```js
class Animal {
  constructor(name) {
    this.speed = 0;
    this.name = name;
  }
  run(speed) {
    this.speed = speed;
    alert(`${this.name} runs with speed ${this.speed}.`);
  }
}
 class Rabbit extends Animal {
  hide() {
    alert(`${this.name} hides!`);
  }
}

let rabbit = new Rabbit("White Rabbit");
rabbit.run(5);  // works — inherited from Animal
rabbit.hide();  // works — Rabbit's own method
```
`Rabbit` gets everything `Animal` has, plus whatever new methods you add. Under the hood, this is exactly the prototype-chaining  — `Rabbit.prototype`'s prototype becomes `Animal.prototype`.
#### Overriding a method 
```js
class Rabbit extends Animal {
  stop() {
    // this replaces Animal's stop() completely for Rabbit
  }
}
```
If `Rabbit` defines its own version of a method `Animal` already has, `Rabbit`'s version wins. Simple override.

#### -->`super.method()`
```js
class Rabbit extends Animal {
  hide() {
    alert(`${this.name} hides!`);
  }

  stop() {
    super.stop(); // run Animal's stop first
    this.hide();  // then do rabbit-specific stuff
  }
}
```
`super.stop()` = "go run the parent class's version of this method." This is the pattern you'll actually use: parent handles the general logic, child adds its own extra behavior on top.

```js
class Rabbit extends Animal {
  constructor(name, earLength) {
    super(name);       // MUST call this first
    this.earLength = earLength; // only AFTER super()
  }
}
```
**The rule**: if your child class has its own `constructor`, you **must** call `super(...)` before you can use `this` at all. Skip it, and you get an error.

In a class that `extends` something, `this` doesn't exist yet until the parent constructor runs and creates it. `super()` is literally what creates `this` for you — calling the parent's constructor. So trying to use `this` before calling `super()` is like trying to use a variable before it's declared — nothing's there yet.
**Practical rule of thumb**: first line of any child constructor = `super(...)`. Always.

```js
class Parent {
  constructor(x) { this.x = x; }
  greet() { alert("Hi from parent"); }
}

class Child extends Parent {
  constructor(x, y) {
    super(x);        // 1. always call super() first
    this.y = y;       // 2. then set your own stuff
  }

  greet() {
    super.greet();    // 3. optionally call parent's version
    alert("Hi from child"); // 4. add your own behavior
  }
}
```

---
## -->Static properties and methods

A regular method belongs to **each instance** (`rabbit.run()`). 
A `static` method belongs to the **class itself** (`Article.compare()`) — you call it directly on the class, never on an object made from it.

```js
class Article {
  constructor(title, date) {
    this.title = title;
    this.date = date;
  }

  static compare(articleA, articleB) {
    return articleA.date - articleB.date;
  }
}

articles.sort(Article.compare); // called on the CLASS, not an article
```

```js
let article = new Article("HTML", new Date());
article.compare(...); //  Error — static methods don't exist on instances
```

#### When you'd actually use this — two real patterns

**1. Utility functions related to the class, but not tied to one specific object** — like the `compare` example above, used for sorting a list of articles. It doesn't make sense as "a property of one article" — it's about comparing two of them.

**2. Factory methods — alternate ways to create an object**
```js
class Article {
  constructor(title, date) {
    this.title = title;
    this.date = date;
  }

  static createTodays() {
    return new this("Today's digest", new Date());
  }
}
let article = Article.createTodays();
```
`Article.createTodays()` is basically a **named alternative constructor** — useful when you have multiple sensible ways to build an object and want clear, readable names for each way, instead of overloading one constructor with confusing optional args.

#### -->Static properties 
```js
class Article {
  static publisher = "Ilya Kantor";
}

alert(Article.publisher); // shared, class-level data
```
Same as `Article.publisher = "..."` written directly. 

```js
class Animal {
  static planet = "Earth";
  static compare(a, b) { return a.speed - b.speed; }
}

class Rabbit extends Animal {}
alert(Rabbit.planet); // "Earth" — inherited
Rabbit.compare(...);   // also inherited
```
Works exactly like regular method inheritance 


---
## -->Private and protected properties and methods

Not everything on an object should be freely changeable from outside. Some stuff is internal machinery — you want to control how it's accessed, not let anyone set it to garbage values directly.

#### -->Protected (`_name`) — convention only, not enforced
```js
class CoffeeMachine {
  _waterAmount = 0;

  set waterAmount(value) {
    if (value < 0) value = 0;
    this._waterAmount = value;
  }

  get waterAmount() {
    return this._waterAmount;
  }
}
let machine = new CoffeeMachine();
machine.waterAmount = -10; // gets clamped to 0, not actually -10
```
The underscore prefix (`_waterAmount`) is just a **naming signal to other developers**: "don't touch this directly, use the getter/setter instead." 

#### -->Private (`#name`) — actually enforced by JS
```js
class CoffeeMachine {
  #waterLimit = 200;

  #fixWaterAmount(value) {
    if (value > this.#waterLimit) return this.#waterLimit;
    return value;
  }

  setWaterAmount(value) {
    this.#waterLimit = this.#fixWaterAmount(value);
  }
}
let machine = new CoffeeMachine();
machine.#waterLimit = 1000; //  real error — genuinely blocked by JS
```
`#` fields are **actually inaccessible** from outside the class — not a convention, a real language rule. This is the modern, proper way to do true privacy.

Private (`#`) fields are **not inherited-accessible** — if you `extends` a class, the child can't touch the parent's `#fields` directly, only through whatever public getters/setters the parent exposes. Protected (`_name`) fields **are** freely accessible in child classes, since it's just a naming convention with no real enforcement.

---
### **`instanceof`**

```js
class Animal {}
class Rabbit extends Animal {}

let rabbit = new Rabbit();

alert(rabbit instanceof Rabbit); // true
alert(rabbit instanceof Animal); // true — works through inheritance too
alert(rabbit instanceof Object); // true — everything's an object
```
It just checks is this object built from this class (or one it inherits from)?
You'll use this occasionally — e.g., checking what type of error you caught (`if (err instanceof ValidationError)`), or checking a value's type before processing it. 
