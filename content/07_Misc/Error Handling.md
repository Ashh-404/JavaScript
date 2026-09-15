
#### The basic syntax
```js
try {
  // code that might fail
} catch (err) {
  // runs ONLY if something in try{} threw an error
}
```
**How it flows**: `try` block runs first. If nothing goes wrong, `catch` is skipped entirely. If something **does** throw an error, execution **immediately stops** , and jumps straight into `catch` 
```js
try {
  alert("Start of try runs");   // (1) runs
  alert("End of try runs");     // (2) runs — no error happened
} catch (err) {
  alert("Catch is ignored");    // never runs
}
```

```js
try {
  alert("Start of try runs");   // (1) runs
  lalala;                        // error! execution stops HERE
  alert("End of try (never reached)"); // (2) never runs
} catch (err) {
  alert("Error has occurred!"); // (3) runs
}
```

##### Two important limitations

1. Only catches _runtime_ errors, not syntax errors
```js
try {
  {{{{{{ // this is just broken, unparseable code
} catch (err) {
  alert("won't even get here");
}
```
If your code is straight-up invalid JS , `try/catch` can't save it — the engine never even gets to the running stage. This is called a parse-time error, and it's unrecoverable from inside that same script. 

2. Only works synchronously — doesn't reach into scheduled/async code
```js
try {
  setTimeout(function() {
    noSuchVariable; // script dies here — NOT caught!
  }, 1000);
} catch (err) {
  alert("won't work");
}
```
Why: the `try` block finishes and exits immediately (since `setTimeout` just schedules the callback for later and moves on) — by the time the callback actually runs a second later, you're no longer inside the `try` block at all. The fix: put `try/catch` **inside** the callback itself:

```js
setTimeout(function() {
  try {
    noSuchVariable;
  } catch {
    alert("error is caught here!");
  }
}, 1000);
```
this exact limitation is one of the big reasons `async/await` is nicer than raw callbacks — `try/catch` **does** work properly around `await`, unlike around `setTimeout`/callbacks.

#### -->The error object — what you actually get in `catch`
```js
try {
  lalala;
} catch (err) {
  alert(err.name);    // "ReferenceError"
  alert(err.message); // "lalala is not defined"
  alert(err.stack);   // detailed call-stack trace, for debugging
}
```
Two properties you'll actually use:
**`name`** (what kind of error — `ReferenceError`, `TypeError`, `SyntaxError`, etc.) and
**`message`** (human-readable description). 
`stack` is mostly for debugging — a trace of how the code got to that error..

#####  -->parsing JSON safely
```js
let json = "{ bad json }";

try {
  let user = JSON.parse(json); // throws, since this isn't valid JSON
  alert(user.name);
} catch (err) {
  alert("Our apologies, the data has errors, we'll try again.");
  alert(err.name);    // "SyntaxError"
  alert(err.message); // something like "Unexpected token b in JSON..."
}
```
Without `try/catch`, malformed JSON from a server would just **crash your app silently** for the user . With it, you can show a proper message, retry, log it — genuinely better UX.

#### -->`throw` — creating your OWN errors
```js
let json = '{ "age": 30 }'; // valid JSON, but missing "name"

let user = JSON.parse(json); // no error here!
alert(user.name); // undefined — not what we want, but JS won't complain
```
`JSON.parse` succeeded — but for your app, a user without a `name` is still a problem. You can manually trigger an error using `throw`:
```js
try {
  let user = JSON.parse(json);

  if (!user.name) {
    throw new SyntaxError("Incomplete data: no name");
  }

  alert(user.name);
} catch (err) {
  alert("JSON Error: " + err.message); // "JSON Error: Incomplete data: no name"
}
```
`throw new SyntaxError(...)` manually creates and throws an error


Problem: `catch` catches **everything**, even errors you didn't anticipate and don't know how to handle correctly.
```js
try {
  user = JSON.parse(json); // forgot "let" — typo bug!
} catch (err) {
  alert("JSON Error: " + err); // WRONG — this isn't actually a JSON problem, it's                                      a ReferenceError from the typo!
}
```
If your `catch` block assumes every error is bad JSON and handles it that way, you'll **misdiagnose real bugs** — like here, where the actual problem is a missing `let`, not malformed data, but your error message lies about it.

**The fix — only handle what you recognize, rethrow the rest:**
```js
try {
  let user = JSON.parse(json);
  if (!user.name) throw new SyntaxError("Incomplete data: no name");
  blabla(); // some unrelated bug
  alert(user.name);
} catch (err) {
  if (err instanceof SyntaxError) {
    alert("JSON Error: " + err.message); // we know how to handle THIS
  } else {
    throw err; // don't recognize it — let it propagate/crash normally
  }
}
```
`instanceof`  checks the error's actual type. If it's the kind you expected, handle it nicely. If not, `throw err` again — this rethrows it, letting it either be caught by an **outer** `try/catch` , or crash the script normally with a proper, honest error message — instead of being silently mislabeled.


#### -->`finally`
```js
try {
  // ...
} catch (err) {
  // ...
} finally {
  // ALWAYS runs — whether try succeeded, or catch handled an error
}
```

```js
try {
  alert('try');
  if (confirm('Make an error?')) BAD_CODE();
} catch (err) {
  alert('catch');
} finally {
  alert('finally');
}
```
Two paths: error happens → `try → catch → finally`. No error → `try → finally`


```js
let start = Date.now();
let result, diff;

try {
  result = fib(num); // might throw if num is invalid
} catch (err) {
  result = 0;
} finally {
  diff = Date.now() - start; // ALWAYS measured correctly, error or not
}
```
Without `finally`, you'd have to duplicate the measure time logic in both the success path and the error path. `finally` guarantees it runs exactly once, either way.


---
## Custom errors, extending Error

#####  plain `Error` doesn't tell you what kind of problem happened
```js
function validateUser(user) {
  if (!user.name) {
    throw new Error("No name specified");
  }
  if (!user.age) {
    throw new Error("No age specified");
  }
}
```

Now imagine calling this and catching the error:
```js
try {
  validateUser({ age: 25 });
} catch (err) {
  // err is just... an Error. We don't actually know WHY it failed,
  // except by reading err.message as plain text
  alert(err.message);
}
```
This mostly works, but it's fragile.  We want to **react differently** depending on the type of problem Like — if it's a validation problem, show a friendly form error near the input field
but if it's a network problem, show a retry button.
With plain `Error`, your only way to tell them apart is checking the **text of the message string** 

#### The idea — make a new error type, specifically for validation problems
```js
class ValidationError extends Error {
  // ...
}
```
This says: I'm creating a new kind of error, called `ValidationError`, which is a _specialized version_ of the built-in `Error`. 

```js
class ValidationError extends Error {
  constructor(message) {
    super(message);
  }
}
```
 **when a child class has its own constructor, it must call `super(...)` before using `this`.** 
 Here, `super(message)` calls `Error`'s own constructor, passing along the message text 
```js
class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = "ValidationError";
  }
}
```
Here's the thing: if we skip this line, `err.name` would still just say `"Error"` — because that's what `Error`'s constructor sets by default, and we didn't override it.
But we want to be able to tell, later, "was this specifically a `ValidationError`?" — so we manually set `this.name` to match our new class's actual name. 

```js
function validateUser(user) {
  if (!user.name) {
    throw new ValidationError("No name specified"); // a SPECIFIC kind of error
  }
}

try {
  validateUser({ age: 25 });
} catch (err) {
  if (err instanceof ValidationError) {
    // we KNOW, for certain, this was a validation problem
    alert("Invalid data: " + err.message);
  } else {
    // some other, unexpected kind of error — don't pretend to understand it
    throw err;
  }
}
```
`err instanceof ValidationError` is a **reliable, structural check** —  "is this object genuinely an instance of the `ValidationError` class?" This connects directly to `instanceof`

This is exactly the same **rethrowing pattern** from the try/catch : handle what you recognize, `throw err` again for what you don't. Except now, instead of guessing based on message text


- User submitted bad data (missing email) → you want to send back "400 Bad Request"
- Database connection failed → you want to send back "500 Server Error"
```js
try {
  validateUser(req.body);
  await saveToDatabase(req.body);
} catch (err) {
  if (err instanceof ValidationError) {
    res.status(400).send(err.message); // client's fault
  } else {
    res.status(500).send("Something went wrong"); // server's fault
  }
}
```
Without custom error classes, you'd have no clean way to tell these two failure categories apart in your `catch` block — you'd be stuck parsing message strings or guessing. With them, `instanceof` gives you a clean, reliable branch.

