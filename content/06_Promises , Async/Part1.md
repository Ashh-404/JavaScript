
## -->Callbacks
#### The core idea — async means "started now, finishes later"
Async exists because some operations take real time (network requests, timers, file reads), and JS can't just freeze the whole program while waiting — `async`/`await` is syntax specifically designed to let you write wait for this, then do that in a way that reads like normal code, instead of nesting everything inside callback functions.

```js
loadScript('/my/script.js');
newFunction(); //  Error — script probably hasn't loaded yet
```
`loadScript` **starts** the download and immediately moves on — it doesn't wait. Code after it runs **before** the script has actually finished loading. This is what asynchronous means: you kick something off, but its result arrives later, not immediately in sequence.

#### The fix — a callback: call this function when you're actually done
```js
function loadScript(src, callback) {
  let script = document.createElement('script');
  script.src = src;
  script.onload = () => callback(script); // runs only once loading finishes
  document.head.append(script);
}
loadScript('/my/script.js', function() {
  newFunction(); // ✓ works now — this only runs after loading completes
});
```
You pass in a function, and the async operation **calls it back** once it's actually done. That's the entire idea behind the name callback.


```js
function loadScript(src, callback) {
  script.onload = () => callback(null, script);        // success: error = null
  script.onerror = () => callback(new Error("..."));    // failure: error passed                                                              first
}
loadScript('/my/script.js', function(error, script) {
  if (error) {
    // handle it
  } else {
    // use script
  }
});
```
Convention: **first argument = error (or `null` if none), second argument = the actual result.** You'll recognize this pattern if you ever see older Node.js code

#### -->callback hell / pyramid of doom
```js
loadScript('1.js', function(error, script) {
  if (error) { handleError(error); }
  else {
    loadScript('2.js', function(error, script) {
      if (error) { handleError(error); }
      else {
        loadScript('3.js', function(error, script) {
          // ...deeper and deeper
        });
      }
    });
  }
});
```
Every sequential async step nests **one level deeper** than the last. A few steps in, this becomes genuinely unreadable — code drifting rightward, error handling duplicated at every level

Callbacks work, but chaining many async steps sequentially makes code deeply nested and hard to follow — entire reason Promises, and later `async/await`, were introduced.

---
## -->Promise

It's an object that represents "a result that doesn't exist yet, but will exist eventually."
```js
let promise = new Promise(function(resolve, reject) {
  // the actual work happens here
});
```

#### -->The executor — runs immediately, does the real work
```js
let promise = new Promise(function(resolve, reject) {
  setTimeout(() => resolve("done"), 1000);
});
```
The function you pass to `new Promise(...)` is called the **executor** — and it runs **immediately**, right when you create the Promise . Its job: do the actual slow work (network request, timer, whatever), and when it's finally done, call one of two special functions it's given:
- **`resolve(value)`** — I succeeded, here's the result
- **`reject(error)`** — I failed, here's the error

`resolve` and `reject` aren't functions you write — JS automatically hands them to you as the two parameters of the executor. You just call whichever one applies once your work finishes.

#### -->The three states —the core mechanic

Every Promise starts as:
- **`"pending"`** — still waiting, no result yet
Then moves to exactly one of:
- **`"fulfilled"`** — `resolve(value)` was called, the value is now available
- **`"rejected"`** — `reject(error)` was called, the error is now available

Once it moves from pending to either of the other two, **that's final — forever.** A Promise can never go back to pending, and can never switch between fulfilled/rejected. 
```js
let promise = new Promise(function(resolve, reject) {
  resolve("done");
  reject(new Error("...")); //  ignored — already settled
  setTimeout(() => resolve("...")); //  also ignored
});
```
Only the first call counts. Everything after is silently ignored. 

#### -->`.then()`
The executor produces the result, but how does your other code actually **use** it? This is the direct fix for the callback problem — instead of passing a callback _into_ the function upfront, you get an object back, and **attach** what to do with it afterward:
```js
let promise = new Promise(function(resolve, reject) {
  setTimeout(() => resolve("done!"), 1000);
});
promise.then(
  result => alert(result),  // runs if resolve() was called
  error => alert(error)     // runs if reject() was called
);
```
`.then()` takes **two optional functions**: first runs on success, second runs on failure. If you only care about success:
```js
promise.then(result => alert(result)); // second arg just omitted
```

#### -->`.catch()`
```js
promise.catch(error => alert(error));
```
This is **exactly the same** as `promise.then(null, error => alert(error))` — just cleaner syntax when you only want to react to failures. You'll use `.catch()` far more often than `.then()`'s second argument in real code.

#### -->Why Promises are genuinely better than callbacks

**1. Natural reading order**
```js
// callback — you had to know what to do BEFORE calling the function
loadScript(src, function(error, script) { ... });

// promise — you call the function FIRST, decide what to do AFTER
let promise = loadScript(src);
promise.then(script => { ... });
```
This directly solves the pyramid of doom ordering problem 

**2. Multiple subscribers — impossible with plain callbacks**
```js
promise.then(script => alert('First handler'));
promise.then(script => alert('Another handler...'));
```
With callbacks, a function can only take **one** callback. With Promises, you can call `.then()` as many times as you want on the same Promise 

#### -->Practical --`loadScript`, Promise version
```js
function loadScript(src) {
  return new Promise(function(resolve, reject) {
    let script = document.createElement('script');
    script.src = src;
    script.onload = () => resolve(script);
    script.onerror = () => reject(new Error(`Script load error for ${src}`));
    document.head.append(script);
  });
}
loadScript("https://example.com/script.js")
  .then(script => alert(`${script.src} is loaded!`))
  .catch(error => alert(`Error: ${error.message}`));
```
Compare directly to the old callback version . `loadScript` just **returns a Promise**, and the caller decides what to do with it afterward, cleanly separated.

#### -->`.finally()` 

```js
new Promise((resolve, reject) => {
  setTimeout(() => resolve("value"), 2000);
})
  .finally(() => alert("Promise ready")) // always runs
  .then(result => alert(result));         // "value" — passed through
```
finally — runs no matter whether the Promise succeeded or failed, good for cleanup (stopping a loading spinner, etc.).
Two things worth knowing: it gets **no arguments** (doesn't know/care about success vs failure) whatever the Promise's actual result was **passes through** untouched to the next `.then()`/`.catch()` after it.


```js
let promise = new Promise(resolve => resolve("done!")); // resolves IMMEDIATELY
promise.then(alert); // still works — runs right away
```
If a Promise is already settled by the time you call `.then()` on it, the handler just runs **immediately**. 

---
## -->Promises chaining

#### -->`.then()` returns a NEW promise
```js
new Promise((resolve) => {
  setTimeout(() => resolve(1), 1000);
})
.then(result => {
  alert(result); // 1
  return result * 2;
})
.then(result => {
  alert(result); // 2
  return result * 2;
})
.then(result => {
  alert(result); // 4
  return result * 2;
});
```
The mechanic: whatever value you `return` from inside a `.then()` handler automatically becomes the result that the **next** `.then()` in the chain receives. Trace it:
1. Original promise resolves with `1`
2. First `.then` gets `1`, alerts it, returns `1 * 2 = 2`
3. Second `.then` gets `2` (because the first `.then` returned it), alerts it, returns `2 * 2 = 4`
4. Third `.then` gets `4`

This is genuinely the entire mechanism. Each `.then()` call **itself returns a new Promise** — that's what lets you keep chaining 

```js
let promise = new Promise(resolve => setTimeout(() => resolve(1), 1000));

promise.then(result => { alert(result); return result * 2; }); // 1
promise.then(result => { alert(result); return result * 2; }); // 1 — NOT 2!
promise.then(result => { alert(result); return result * 2; }); // 1 — NOT 4!
```
All three alert `1`  Because these are three **independent** subscriptions to the **same original promise** — each one gets the _original_ result (`1`), not each other's return values. 

#### -->Returning a PROMISE from `.then()`
```js
new Promise(resolve => setTimeout(() => resolve(1), 1000))
.then(result => {
  alert(result); // 1
  return new Promise(resolve => setTimeout(() => resolve(result * 2), 1000));
})
.then(result => {
  alert(result); // 2 — after ANOTHER 1 second wait
  return new Promise(resolve => setTimeout(() => resolve(result * 2), 1000));
})
.then(result => {
  alert(result); // 4
});
```
If you `return` a **Promise** (instead of a plain value) from inside `.then()`, the **next** `.then()` automatically **waits** for that returned Promise to settle, then receives _its_ resolved value. 

```js
loadScript("one.js")
  .then(script => loadScript("two.js"))
  .then(script => loadScript("three.js"))
  .then(script => {
    one(); two(); three();
  });
```
Each `loadScript(...)` call returns a Promise; the next `.then()` automatically waits for it, then moves on to the next step. This is the concrete resolution of the pyramid of doom problem 

```js
loadScript("one.js").then(script1 => {
  loadScript("two.js").then(script2 => {
    loadScript("three.js").then(script3 => {
      // works, but grows RIGHTWARD — same pyramid problem as callbacks!
    });
  });
});
```
This still technically works but it recreates the exact pyramid-of-doom structure you're trying to escape. **Chaining (flat, `.then().then().then()`) is the preferred style almost always.**

#### -->Thenables 

Some objects that aren't real Promises but have a `.then()` method (called thenables) get treated like Promises automatically by the chain. This exists so third-party libraries can make Promise-compatible objects. 
#### -->`fetch()`
```js
fetch('/user.json')
  .then(response => response.json())
  .then(user => alert(user.name));
```
`fetch(url)` returns a Promise that resolves once the server responds (headers received). `response.json()` **also returns a Promise** (parsing the body takes a moment too) — which is exactly why you need a **second** `.then()` to actually get the parsed data. This two-step fetch, then parse pattern is the standard shape of every API call you'll write.


```js
fetch('/user.json')
  .then(response => response.json())
  .then(user => fetch(`https://api.github.com/users/${user.name}`)) // use result                                                 of step 1 to make step 2's request
  .then(response => response.json())
  .then(githubUser => {                  // now use the github data
  });
```
This is a completely realistic pattern: fetch user data → use part of that data to make a **second**, dependent request → process that result. 

```js
.then(githubUser => new Promise((resolve, reject) => {
  let img = document.createElement('img');
  img.src = githubUser.avatar_url;
  document.body.append(img);

  setTimeout(() => {
    img.remove();
    resolve(githubUser);       // only now does the chain continue
  }, 3000);
}))
.then(githubUser => alert(`Finished showing ${githubUser.name}`));
```
 If you want something to happen after this step is truly done (like after an image finishes displaying for 3 seconds), and that step isn't naturally Promise-based (like `setTimeout` isn't), you **wrap it in a `new Promise(...)`** yourself, and call `resolve()` only when it's actually finished. This lets you extend the chain even further afterward. 


```js
function loadJson(url) {
  return fetch(url).then(response => response.json());
}

function loadGithubUser(name) {
  return loadJson(`https://api.github.com/users/${name}`);
}

loadJson('/user.json')
  .then(user => loadGithubUser(user.name))
  .then(githubUser => alert(githubUser.name));
```
This is genuinely how real code looks — small, named, reusable Promise-returning functions, composed together via chaining. Clean, readable, testable.

---
## -->Error handling with promises

Promise chains are great at error handling. When a promise rejects, the control jumps to the closest rejection handler. That’s very convenient in practice.
#### -->`.catch()`
```js
fetch('https://no-such-server.blabla')
  .then(response => response.json())
  .catch(err => alert(err));
```
`.catch()` doesn't need to sit immediately after the step that might fail — it can be at the **very end** of a long chain, and it'll still catch a failure from **any** step before it. This directly mirrors `try { many lines } catch (err) { ... }` — one `catch` covering a whole block of `try`, not one per line.

##### -->Errors auto-convert to rejections — invisible try/catch
```js
new Promise((resolve, reject) => {
  throw new Error("Whoops!");
}).catch(alert); // "Error: Whoops!"
```
This behaves **identically** to:
```js
new Promise((resolve, reject) => {
  reject(new Error("Whoops!"));
}).catch(alert);
```
JS wraps the executor (and every `.then()` handler) in an invisible `try/catch`. If something throws inside, it's automatically treated as `reject(thatError)`
```js
new Promise((resolve) => resolve("ok"))
  .then(result => { throw new Error("Whoops!"); }) // this REJECTS the chain
  .catch(alert); // "Error: Whoops!"
```
And it catches genuine programming mistakes too, not just deliberate `throw`s:
```js
.then(result => { blabla(); }) // blabla doesn't exist — ReferenceError
.catch(alert); // "ReferenceError: blabla is not defined"
```

 A regular synchronous `try/catch` does NOT catch errors inside `setTimeout`. But Promise chains **do** correctly propagate errors from anywhere inside the chain..


```js
new Promise((resolve, reject) => { throw new Error("Whoops!"); })
  .catch(function(error) {
    if (error instanceof URIError) {
      // handle it
    } else {
      alert("Can't handle such error");
      throw error; // rethrow — jumps to the NEXT .catch down the chain
    }
  })
  .then(function() { /* skipped */ })
  .catch(error => {
    alert(`Unknown error: ${error}`); // this one catches it
  });
```
Exact same idea as rethrowing in regular `try/catch`: only handle what you recognize, `throw` again for what you don't — control then jumps to the **next** `.catch()` further down the chain 

**The flip side, worth noting**: if a `.catch()` **successfully** handles an error (doesn't rethrow), execution continues normally to the **next `.then()`** afterward — the chain "recovers" and proceeds as if nothing went wrong.
```js
new Promise((resolve, reject) => { throw new Error("Whoops!"); })
  .catch(error => alert("handled, continuing normally"))
  .then(() => alert("this runs next!")); // yes, this DOES run
```

##### Unhandled rejections 
```js
new Promise(() => {
  noSuchFunction(); // errors
})
  .then(() => { /* ... */ }); // no .catch anywhere!
```
If nothing ever catches the error, it goes unhandled . The browser logs a global error to the console 

--> **Same mental model as regular error handling, just async-flavored**: `.catch()` = `catch`, thrown errors inside `.then()`/executor automatically become rejections , rethrow what you don't recognize, and a single `.catch()` at the end of a chain is a totally normal, common pattern 


---
## Promise API

##### -->`Promise.all`
Sometimes you need several independent async things to happen **at the same time** (not one-after-another like chaining), and you want to wait until **all** of them finish before continuing.
```js
Promise.all([
  new Promise(resolve => setTimeout(() => resolve(1), 3000)),
  new Promise(resolve => setTimeout(() => resolve(2), 2000)),
  new Promise(resolve => setTimeout(() => resolve(3), 1000))
]).then(alert); // [1, 2, 3] — after 3 seconds (the LONGEST one)
```
Takes an **array of promises**, runs them all **in parallel** , and gives you back an array of results **once every single one has resolved**. 
**Order is preserved** — even though promise 3 finishes fastest, its result still lands at index 2 in the output array, matching the order you passed them in, not the order they finished.

```js
let urls = ['url1', 'url2', 'url3'];
let requests = urls.map(url => fetch(url)); // array of promises, all starting                                                      immediately

Promise.all(requests)
  .then(responses => { /* all done, work with all of them */ });
```
**This exact pattern — `array.map(x => someAsyncCall(x))` then `Promise.all(...)`** — is what you'll reach for whenever you need to fetch/process multiple independent things at once instead of one at a time. Genuinely common, worth remembering this shape.


```js
Promise.all([p1, p2_that_rejects, p3]).catch(alert);
```
If **any one** promise in the list rejects, `Promise.all` **immediately** rejects as a whole — even if the others would've eventually succeeded. The other promises don't get cancelled , they just keep running in the background, but their results are **ignored** by `Promise.all`. 

#### The other 5 methods
- **`Promise.allSettled`** — like `Promise.all`, but never rejects as a whole — gives you a status (`fulfilled`/`rejected`) for _every_ promise individually, even if some failed. 
- **`Promise.race`** — resolves/rejects as soon as the **first** promise settles (whichever is fastest), ignores the rest.
- **`Promise.any`** — like `race`, but specifically waits for the first **success**, ignoring rejections unless _everything_ fails.
- **`Promise.resolve(value)`** — makes an already-resolved promise instantly. Mostly obsolete once you know `async/await` 
- **`Promise.reject(error)`** — makes an already-rejected promise instantly. Rarely used.

---
---
## Async/await

There’s a special syntax to work with promises in a more comfortable fashion, called async/await. It’s surprisingly easy to understand and use.

### -->`async`
```js
async function f() {
  return 1;
}
f().then(alert); // 1
```
`async` before a function does one thing: **it always makes the function return a Promise**, automatically. `return 1` inside an async function doesn't actually return the raw number `1` — it returns `Promise.resolve(1)`, wrapped automatically. 

### -->`await`
```js
async function f() {
  let promise = new Promise((resolve) => {
    setTimeout(() => resolve("done!"), 1000);
  });

  let result = await promise; // pauses HERE until the promise settles
  alert(result); // "done!"
}
```
`await` literally means: pause this function right here until the promise settles, then give me the resolved value directly." Compare to what you'd write with raw Promises:
```js
// old way
promise.then(result => alert(result));

// async/await way
let result = await promise;
alert(result);
```
Same effect, but `await` lets you write it as a normal variable assignment, on its own line, reading top-to-bottom — instead of nesting your next step inside a `.then()` callback. **This is the entire point of the syntax** — it makes async code _look_ synchronous, even though it's still fully non-blocking underneath 
**Rule**: `await` only works **inside** an `async` function.


```js
async function showAvatar() {
  let response = await fetch('/user.json');
  let user = await response.json();

  let githubResponse = await fetch(`https://api.github.com/users/${user.name}`);
  let githubUser = await githubResponse.json();

  let img = document.createElement('img');
  img.src = githubUser.avatar_url;
  document.body.append(img);

  await new Promise(resolve => setTimeout(resolve, 3000));
  img.remove();

  return githubUser;
}
```
Compare this to the `.then().then().then()` chain version — **this reads like a normal step-by-step recipe**, no nesting, no chaining syntax. Every `await` just pauses until that step's result is ready, then moves to the next line. This is genuinely what you'll write for basically every API interaction from here forward.


```js
async function f() {
  try {
    let response = await fetch('http://no-such-url');
    let user = await response.json();
  } catch (err) {
    alert(err); // catches errors from EITHER await line
  }
}
```
If a promise **rejects**, `await` **throws** that rejection as a regular error . That means normal `try/catch` works perfectly to catch it. This is a genuine structural win
**If you skip try/catch entirely**, the async function itself just returns a rejected Promise, which you'd need to `.catch()` from outside:

```js
async function f() {
  let response = await fetch('http://no-such-url');
}
f().catch(alert); // catches it from outside
```

##### Async methods in classes — just add `async`
```js
class Waiter {
  async wait() {
    return await Promise.resolve(1);
  }
}
new Waiter().wait().then(alert); // 1
```
Same rule, just inside a class , `async` works on class methods exactly like standalone functions.


```js
let results = await Promise.all([
  fetch(url1),
  fetch(url2),
  fetch(url3)
]);
```
You get: run these three fetches simultaneously, and pause this function until **all** of them are done. 

**You will write `async function`/`await`/`try-catch` constantly. You will almost never write raw `.then()` chains for your own code anymore** — except at the very top level, outside any async function, where `await` isn't syntactically allowed..

---
## -->Microtasks 

Microtasks is pure **event-loop internals**: the exact order in which the JS engine schedules and runs `.then()` callbacks versus `setTimeout` callbacks versus regular code, down to the level of a microtask queue vs macrotask queue. It's real, it's technically accurate, and it explains some genuinely subtle timing quirks ..

 Promise callbacks (`.then()`) run before `setTimeout` callbacks, even if the timeout is `0ms`, because Promises use a higher-priority microtask queue that the engine always empties before moving to the next `setTimeout`-style macrotask. 

> `.then/catch/finally` handlers are always called **after the current code is finished** — even if the promise is already resolved.

```js
console.log("1");
Promise.resolve().then(() => console.log("2"));
console.log("3");

// output: 1, 3, 2 — NOT 1, 2, 3
```
Even though the promise resolves **instantly**, the `.then()` callback still waits until all the currently-running synchronous code finishes first. 

- Promise handling is always async (queued, never truly instant) 
- The `unhandledrejection` timing detail — irrelevant, you already skipped that whole event listener topic.

### -->Macrotasks 

Things like `setTimeout`, `setInterval`, UI rendering, and user events (clicks) go into the **macrotask queue**. Each macrotask runs **one at a time**, and — importantly — after **every single macrotask**, the engine fully empties the **microtask queue** (all pending `.then()`s) before moving to the next macrotask.

```js
setTimeout(() => console.log("macrotask"), 0);
Promise.resolve().then(() => console.log("microtask"));
// output: "microtask" then "macrotask" — even though setTimeout was 0ms!
```
Microtasks (Promises) always run before the next macrotask (`setTimeout`), even at `0ms` delay.






