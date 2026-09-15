
An **event** is simply something that happens in the browser.
Examples:
```js
click       // user clicks
keydown     // user presses a key
submit      // form is submitted
focus       // input becomes active
```

#### 2. Event Handler

An **event handler** is the function that runs when an event happens.
```js
button.addEventListener("click", function() {
    console.log("Button clicked!");
});
```

```js
click happens
     ↓
browser detects it
     ↓
your function runs
```
That's the fundamental idea of browser events.

#### 3. `addEventListener()` 

```js
element.addEventListener("event", function);
```

> "Whenever this button is clicked, run this function."


You can add multiple listeners
```js
button.addEventListener("click", function() {
    console.log("Hello");
});

button.addEventListener("click", function() {
    console.log("Bye");
});
```
Both can run.
```js
button.onclick = function() {
    console.log("Hello");
};

button.onclick = function() {
    console.log("Bye");
};
```

> **`onclick` → one handler**  
> **`addEventListener` → multiple handlers**


#### 4. Event Object
When an event happens, the browser gives your function an **event object** containing information about what happened.
```js
button.addEventListener("click", function(event) {
    console.log(event);
});
```

You can use things like:
```js
event.type   // what event happened
event.currentTarget   //  which element is handling the event
```

##### `event.target` --> The element that was actually clicked.##### `event.currentTarget` -- > The element whose event handler is currently running.


--> Some browser actions happen automatically.
```js
<a href="https://google.com">Google</a>
```
Clicking it normally navigates to Google.
You can stop that:
```js
link.addEventListener("click", function(event) {
    event.preventDefault();
});
```
Now the default action is prevented.

---
### 5. The Different Ways to Attach Events

##### 1. HTML
```html
<button onclick="sayHello()">Click</button>
```

##### 2. DOM property
```js
button.onclick = sayHello;
```

##### 3. `addEventListener()` 
```js
button.addEventListener("click", sayHello);
```

primarily use `addEventListener()`.


See the mistake
```js
button.addEventListener("click", sayHello);

button.addEventListener("click", sayHello());  // wrong here
```
sayHello means:  "Here is the function. Run it when the click happens."
while:
sayHello() means: "Run the function RIGHT NOW."


You can remove a listener:
```js
function sayHello() {
    console.log("Hello");
}

button.addEventListener("click", sayHello);
button.removeEventListener("click", sayHello);
```

---
---
#### 1. Bubbling

```js
<form>
  <div>
    <button>Click me</button>
  </div>
</form>
```
You click the button.
The event travels **upward**:
```
button
  ↑
div
  ↑
form
  ↑
body
  ↑
html
  ↑
document
```
This is called **bubbling**.
Why does the parent's handler run

Suppose:
```js
<div>
    <button>Click</button>
</div>
```
and:
```js
div.addEventListener("click", () => {
    console.log("DIV clicked");
});
```
You click the button.
But the click event bubbles from the button to its parent, so the div gets a chance to handle it too.


`event.target` tells you:
> **"Which element did the event actually start on?"**

```js
div.addEventListener("click", function(event) {
    console.log(event.target);
});
```
If you click the button:
```
event.target → button
```
Even though the handler is attached to the `div`.

```js
event.target
    ↓
BUTTON

event.currentTarget
    ↓
DIV
```

##### --> `this`
```js
div.addEventListener("click", function(event) {
    console.log(this);
});
```
`this` refers to the element whose handler is running.
So:
```
this === event.currentTarget
```

Stop bubbling
```js
event.stopPropagation();
```

> **"Stop the event from travelling further."**

So the body handler doesn't run.

```js
preventDefault()
    ↓
"Browser, don't do your normal thing."

stopPropagation()
    ↓
"Event, don't travel to the parents."
```


`stopImmediatePropagation()`:

> stops the event from going to parents **AND** stops other handlers on the same element.


#### -->Capturing 

Now comes the opposite direction.
But events actually travel **down first**, then back up.
When you click the button, the event travels:

```js
        CAPTURING
            ↓
FORM → DIV → BUTTON
              ↑
              │
           TARGET
              │
              ↓
FORM ← DIV ← BUTTON
        BUBBLING
```


```js
element.addEventListener("click", handler);
```
runs during **bubbling**.
To listen during capturing:
```js
element.addEventListener("click", handler, true);

// OR
element.addEventListener("click", handler, {
    capture: true
```


##### **Events → `addEventListener()` → event object → bubbling → `target` vs `currentTarget` → event delegation**

---
---
### -->Event Delegation

```js
<div id="menu">
    <button>Save</button>
    <button>Load</button>
    <button>Search</button>
</div>
```

You _could_ do:
```js
saveButton.addEventListener("click", save);
loadButton.addEventListener("click", load);
searchButton.addEventListener("click", search);
```
But imagine you have 100 buttons.
That's annoying.
Instead, because clicks **bubble upward**, we can put ONE listener on the parent:
```js
menu.addEventListener("click", function(event) {
    // figure out which button was clicked
});
```
So:
```js
Button 1 ─┐
Button 2 ─┤
Button 3 ─┤→ MENU's one event listener
Button 4 ─┤
Button 5 ─┘
```
That's **event delegation**.

```js
menu.addEventListener("click", function(event) {

    if (event.target.tagName !== "BUTTON") {
        return;
    }

    console.log("A button was clicked");
});
```

Meaning:
```js
Something inside menu was clicked
            ↓
     Who was clicked?
            ↓
      event.target
            ↓
     Is it a BUTTON?
       ↙       ↘
     No         Yes
     ↓           ↓
   ignore    handle it
```
That's the **core concept**.


```js
<button>
    <strong>Save</strong>
</button>
```

You click directly on **Save**.
What is: event.target
Because the `<strong>` is the element you actually clicked.
So this:
```js
if (event.target.tagName !== "BUTTON") return;
```
would incorrectly ignore the click.

##### `closest()` solves this

> "Starting from whatever was clicked, find the nearest `<button>` above it."
```js
const button = event.target.closest("button");
```

#### -->`data-*` attributes

```js
<button data-action="save">Save</button>
<button data-action="load">Load</button>
<button data-action="search">Search</button>
```
These are **custom data attributes**.

Anything beginning with `data-*` attribute.

```js
event.target.dataset.action   // "save"
```



```js
<button data-counter>1</button>
<button data-counter>5</button>
<button data-counter>10</button>
```

Then one document-level listener:
```js
document.addEventListener("click", function(event) {

    if (event.target.dataset.counter !== undefined) {
        event.target.value++;
    }

});
```
Now **any element with `data-counter` automatically gets that behavior**.

---
### Browser Default Actions 

Some events automatically cause the browser to do something.
Examples:
- Clicking `<a>` → browser navigates
- Submitting `<form>` → browser submits/reloads
- Right-click → context menu appears

To stop that:
```js
event.preventDefault();
```


