
**`window`** = the global object in browsers, representing the browser window itself.
**`document`** = your entry point for reading/changing the actual page content (we'll use constantly). **DOM** = the page's content, represented as objects you can manipulate.
**BOM** = everything else browser-related (`location`, `navigator`, `alert`) — not part of the page itself.
```js
document.body.style.background = "red"; // DOM — changes the page
alert(location.href); // BOM — browser stuff, not page content
```

---
---
## -->Dom Tree

**HTML becomes a tree of objects when the browser loads it.** Every tag is an element node, text inside tags becomes text nodes, and JS can read/modify any of it.
```js
document.body.style.background = 'red'; // document.body = the <body> tag, as an                                                  object
```

- **`document`** is your entry point — the whole page, as one object you navigate from.
- Tags nest inside each other exactly like the HTML does — `<html>` → `<head>`/`<body>` → their contents, etc. This is why you'll sometimes hear DOM tree — it's literally tree-shaped, matching  HTML's nesting.
- The browser **auto-corrects** malformed HTML (missing tags get added, unclosed tags get closed) — so the DOM might not exactly match what you typed if your HTML has mistakes.
- **Comments become DOM nodes too** — technically true, practically irrelevant

### -->Selecting elements
```js
document.querySelector('.my-class');  // first matching element
document.querySelector('#my-id');
document.querySelectorAll('.item');   // ALL matching elements (a list)
```
`querySelector` uses **CSS selector syntax** — `.class`, `#id`, `tag`, `[attribute]` .
The older `getElementById`/`getElementsByClassName` still exist but `querySelector` covers everything more flexibly.

```js
let items = document.querySelectorAll('.item');
items.forEach(item => console.log(item)); // loop over all matches
```

### -->Reading/changing content
```js
element.textContent = "Hello";     // plain text, safe, most common
element.innerHTML = "<b>Hello</b>"; // parses as HTML — use carefully, can be a security risk with user input
```
**Rule of thumb**: use `textContent` unless you specifically need to insert actual HTML tags.

### -->Changing styles/classes
```js
element.style.color = "red";           // inline style, one property at a time
element.classList.add("active");        // add a CSS class
element.classList.remove("active");
element.classList.toggle("active");      // add if missing, remove if present
element.classList.contains("active");    // check — returns true/false
```
**Prefer `classList` over `.style`** — you define the actual styling in CSS, and just toggle class names from JS. Cleaner separation.

### -->Creating and adding elements
```js
let div = document.createElement('div'); //creates a NEW element, not yet on page
div.textContent = "New item";
document.body.append(div);              // now it's actually visible on the page

element.remove(); // removes it from the page
```
**Pattern you'll use constantly**: create → set content/attributes → append to a parent.

### -->Attributes
```js
element.getAttribute('data-id');
element.setAttribute('data-id', '5');
element.id;      // for standard attributes, you can also just use dot access
element.value;   // e.g. for reading an <input>'s current value
```

### -->Events
```js
button.addEventListener('click', function(event) {
  console.log("clicked!");
});
```
`event` (the parameter) gives you info about what happened — most commonly:
```js
button.addEventListener('click', (event) => {
  console.log(event.target); // the exact element that was clicked
});
```

For forms specifically:
```js
form.addEventListener('submit', (event) => {
  event.preventDefault(); // stops the page from reloading (default form behavior)
  // ...your logic
});

input.addEventListener('input', (event) => {
  console.log(event.target.value); // current text as user types
});
```
`event.preventDefault()` is genuinely important — without it, submitting a form reloads the whole page, which you almost never want in a JS-driven app.

---
#### The core syntax
```js
document.querySelector(cssSelector);
document.querySelectorAll(cssSelector);
```
Both take a **CSS selector string**
#### `querySelector` — returns the FIRST match, or `null`
```js
document.querySelector('.item');    // first element with class="item"
document.querySelector('#header');  // element with id="header"
document.querySelector('div');      // first <div> on the page
document.querySelector('input[type="text"]'); // first text input
```
If nothing matches, you get `null` back — not an error, not `undefined`. 

```js
let el = document.querySelector('.does-not-exist');
el.textContent = "hi"; //  Error: Cannot set property of null
```

#### `querySelectorAll` — returns ALL matches, as a NodeList
```js
let items = document.querySelectorAll('.item');
console.log(items.length); // how many matched
```
This is not a real array — it's a NodeList (array-like). It supports `for...of` and `.forEach()`, but not `.map()`/`.filter()` directly:
```js
items.forEach(item => console.log(item)); // ✓ works
items.map(item => item.textContent);       //  doesn't exist on NodeList

Array.from(items).map(item => item.textContent); // ✓ convert first if you need array methods
```

```js
'.classname'        // by class
'#id'                // by id
'tagname'            // by tag, e.g. 'div', 'button', 'input'
'.parent .child'     // descendant (child.class inside something with parent.class)
'.parent > .child'   // DIRECT child only
'[data-id="5"]'      // by attribute value
'input[type="text"]' // tag + attribute combo
'.item.active'       // element with BOTH classes (no space = combined, not descendant)
'li:first-child'     // pseudo-selectors work too
```
Anything valid in CSS works here — this is genuinely one selector language for both your stylesheet and your JS.


You're not limited to `document.querySelector` — you can call it on **any** element, to search only inside that element:
```js
let container = document.querySelector('.todo-list');
let firstItem = container.querySelector('.item'); // only searches INSIDE container
```
Genuinely useful pattern: grab a container first, then search within it, rather than risking grabbing an unrelated `.item` elsewhere on the page.

#### The realistic pattern 
```js
// Grab references once, at the top
let input = document.querySelector('#todo-input');
let addBtn = document.querySelector('#add-btn');
let list = document.querySelector('#todo-list');

// Use them throughout your code
addBtn.addEventListener('click', () => {
  let newItem = document.createElement('li');
  newItem.textContent = input.value;
  list.append(newItem);
});
```
**Common convention**: grab all your key elements at the top of your script, once, store them in clearly-named variables (`input`, `list`, `addBtn`), then reference those variables everywhere else — instead of calling `querySelector` repeatedly for the same element.

#### Older alternatives 
```js
document.getElementById('header');           // faster, but only works for IDs
document.getElementsByClassName('item');      // returns a live HTMLCollection
document.getElementsByTagName('div');
```
These predate `querySelector` and are marginally faster for simple cases, but `querySelector`/`querySelectorAll` is more flexible (full CSS selector power) 

---
### -->`matches()`
```js
if (element.matches('.active')) { ... }
```
Checks does this element match this selector? — `true`/`false`, doesn't search anything. Useful when  already looping over elements and want to filter by a condition.

### -->`closest()`
```js
button.addEventListener('click', (e) => {
  let item = e.target.closest('.todo-item'); // finds nearest ANCESTOR matching                                                     selector
  item.remove();
});
```
Searches **upward** from an element through its parents until it finds one matching your selector. Real practical use: click a delete button _inside_ a todo item, but you want to remove the whole item, not just the button — `closest('.todo-item')` gets you there without manually walking `.parentElement` repeatedly.

### -->The DOM class hierarchy 

Pure background on _why_ DOM objects have the properties they have. 
DOM elements are objects built through inheritance
**Every single DOM element you've ever touched is built using the exact prototype/class inheritance system 
```js
document.body instanceof HTMLBodyElement; // true
document.body instanceof HTMLElement;     // true
document.body instanceof Element;         // true
document.body instanceof Node;            // true
document.body instanceof EventTarget;     // true
```
`document.body` isn't just an object with some DOM stuff on it — it's an instance sitting at the bottom of a real inheritance chain, five levels deep, each level `extends`-ing the one above it 

```
EventTarget  →  Node  →  Element  →  HTMLElement  →  HTMLBodyElement
```
**`EventTarget`** (top of the chain, most general) — gives **every** DOM node the ability to have event listeners attached (`.addEventListener`). This is why `document.body.addEventListener(...)` and `button.addEventListener(...)` both work identically — that capability isn't defined separately on `body` and `button`, it's defined **once**, on `EventTarget`, and everything inherits it. 

**`Node`** — adds the tree-navigation stuff: `.parentNode`, `.childNodes`, `.nextSibling`. This is why _any_ node — element, text, comment — can navigate the tree the same way, because they all inherit from `Node`.

**`Element`** — adds element-specific navigation and searching: `.children`, `.querySelector`, `.getElementsByTagName`. Text nodes and comments **don't** get these, because they don't inherit from `Element` — only actual tag-based nodes do. This is exactly why `textNode.querySelector(...)` doesn't exist but `divElement.querySelector(...)` does 

**`HTMLElement`** — adds properties common to all **HTML** tags specifically (as opposed to XML/SVG elements, which the browser also supports): things like `.hidden`, `.style`, `.innerHTML`.

**`HTMLBodyElement`** (or `HTMLInputElement`, `HTMLAnchorElement`, etc.) — the most specific level, adding properties unique to that particular tag. `<input>` gets `.value`, `<a>` gets `.href` — these only exist on their specific classes, not shared with unrelated tags.

```js
input.value;   // works — HTMLInputElement provides this
div.value;     // undefined — div doesn't inherit from HTMLInputElement
```
: `.value` is defined specifically on `HTMLInputElement` (and a few similar classes like `HTMLSelectElement`), not on the shared `HTMLElement`/`Element`/`Node` levels 

#### `console.dir()`
```js
console.dir(document.body);
```
In browser DevTools, this shows you the **actual prototype chain** — `HTMLBodyElement.prototype → HTMLElement.prototype → Element.prototype → ...` — literally visible, exactly like `__proto__` chains 


#### -->`innerHTML` vs `textContent`
```js
element.innerHTML = "<b>bold</b>"; // parses as HTML — tags become real tags
element.textContent = "<b>bold</b>"; // literal text — shows the tags as text, not rendered
```
If you're inserting anything that came from **user input**, use `textContent`, never `innerHTML`. If a user types `<script>...</script>` into a form and you shove it into `innerHTML`, that's a real vulnerability (called XSS). `textContent` treats everything as plain text, safely, no matter what's in it.

#### -->The `outerHTML` trap 
```js
let div = document.querySelector('div');
div.outerHTML = '<p>New</p>'; // replaces div IN THE PAGE...
div.outerHTML; // ...but 'div' variable still points to the OLD div object!
```
Assigning to `outerHTML` replaces the element in the actual page, but your JS variable still references the **old**, now-removed element. 

```js
input.value    // current text in an <input>
link.href      // the URL in an <a href="...">
element.id     // the id attribute
```
Most standard HTML attributes have a matching JS property you access with dot notation 

#### `hidden`
```js
element.hidden = true; // same as style.display = 'none', just shorter
```

---
---
### Attributes and properties 
```html
<div data-user-id="123" data-order-state="pending"></div>
```

```js
element.dataset.userId;     // "123"
element.dataset.orderState; // "pending" — multi-word becomes camelCase
```
The standard, safe way to attach custom data to HTML elements from your JS, without polluting them with random non-standard attributes. You'll use this pattern in real projects (e.g., tagging each todo `<li>` with `data-id` so you know which one to delete when clicked).


### -->Modifying the document 
```js
let div = document.createElement('div');
div.className = "alert";
div.textContent = "Hello"; // or innerHTML if you need actual tags
document.body.append(div); // adds it to the page
```

##### The insertion methods
```js
node.append(x)      // add at the end, inside node
node.prepend(x)      // add at the beginning, inside node
node.before(x)       // add right before node (outside it)
node.after(x)         // add right after node (outside it)
node.replaceWith(x)   // swap node out entirely
```

### Styles and classes
```js
element.classList.add('active');
element.classList.remove('active');
element.classList.toggle('active'); // add if missing, remove if present
element.classList.contains('active'); // true/false
```

