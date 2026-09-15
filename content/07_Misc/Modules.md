
As your app grows, you can't keep everything in one giant file. You want to split code across files. Modules are JS's system for that: **exporting** (making something available from a file) and **importing** (pulling it into another file).

#### ES6 Modules 

**Named exports** — export multiple things from one file, each with its own name:
```js
// math.js
export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}
```
You can export as many named things as you want from a single file — functions, variables, classes, whatever.

**Default export** — exactly ONE per file, meant to represent the main thing this file provides
```js
// math.js
export default function multiply(a, b) {
  return a * b;
}
```

**Importing named exports** — must use curly braces, and the names must match exactly what was exported:
```js
// app.js
import { add, subtract } from './math.js';
add(2, 3); // 5
```

**Importing a default export** — no curly braces, and you can name it whatever you want on the import side:
```js
// app.js
import multiply from './math.js'; //could also be: import anything from                                                     './math.js';

multiply(2, 3); // 6
```


```js
// math.js
export function add(a, b) { return a + b; }
export function subtract(a, b) { return a - b; }
export default function multiply(a, b) { return a * b; }
```
```js
// app.js
import multiply, { add, subtract } from './math.js';
```
Default import comes first (no braces), named imports follow (with braces) 


#### CommonJS — the older system, still relevant for Node/some npm packages

Before ES6 modules existed, Node.js used its own system called **CommonJS**. Different keywords, same underlying idea.

**Exporting:**
```js
// math.js
function add(a, b) { return a + b; }
function subtract(a, b) { return a - b; }

module.exports = { add, subtract };
```
`module.exports` is an object — you attach whatever you want to make available onto it. `{ add, subtract }` here is shorthand object syntax (same as `{ add: add, subtract: subtract }`) 

**Importing:**
```js
// app.js
const math = require('./math.js');

math.add(2, 3); // 5
```
`require(...)` returns whatever was assigned to `module.exports` — here, an object — and you access properties off it normally with dot notation.


**Mongoose** (the MongoDB library ) and a good chunk of the Node.js ecosystem, especially older packages, still use `require`/`module.exports`. You don't need to write CommonJS yourself often, but you WILL see it in tutorials, package source code, and possibly some backend boilerplate. 


