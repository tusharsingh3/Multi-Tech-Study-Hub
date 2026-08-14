# JavaScript Notes

> Roadmap-aligned reference notes. Sections intentionally follow `roadmap.md` order.
>
> Examples are illustrative. Prefer understanding execution semantics and trade-offs over memorizing syntax.

## Table of Contents

1. [Basics](#1-basics)
2. [Core Concepts](#2-core-concepts)
3. [Data Structures & Built-ins](#3-data-structures--built-ins)
4. [ES6+ Features](#4-es6-features)
5. [Asynchronous JavaScript](#5-asynchronous-javascript)
6. [DOM & Browser](#6-dom--browser)
7. [Advanced Topics](#7-advanced-topics)
8. [Tooling & Ecosystem](#8-tooling--ecosystem)
9. [Performance & Best Practices](#9-performance--best-practices)
10. [Next Steps](#10-next-steps)

---

# 1. Basics

## 1.1 Syntax, Variables (`var`, `let`, `const`)

JavaScript is dynamically typed: variables hold references to values, and a variable may later reference a value of another type.

- `var` is function-scoped, can be redeclared, and is hoisted with an initial value of `undefined`.
- `let` is block-scoped, cannot be redeclared in the same scope, and exists in the temporal dead zone until initialization.
- `const` is also block-scoped and requires initialization. It prevents reassignment of the binding, not mutation of the referenced object.

```js
var legacy = 1;
let counter = 0;
const config = { retries: 3 };

counter += 1;
config.retries = 5; // valid
// config = {};     // TypeError
```

**Best practices**

- Default to `const`; use `let` only when reassignment is intentional.
- Avoid `var` in modern application code because function scope and hoisting make behavior easier to misread.
- Give variables semantic names; avoid encoding type into names unless the codebase standard requires it.

**Common mistake:** assuming `const` makes an object immutable. Use `Object.freeze()` for shallow runtime protection when appropriate.

**Interview angle:** explain binding immutability vs object immutability and temporal dead zone behavior.

## 1.2 Data Types and Type Coercion

JavaScript has seven primitive types: `string`, `number`, `bigint`, `boolean`, `undefined`, `symbol`, and `null`. Everything else is an object, including arrays and functions.

```js
const name = "Tushar";          // string
const score = 42;               // number
const huge = 9007199254740993n; // bigint
const active = true;            // boolean
const missing = undefined;
const id = Symbol("id");
const empty = null;
```

`typeof null` returns `"object"` because of a historical language bug.

```js
typeof null;       // "object"
typeof [];         // "object"
typeof function(){}; // "function"
Array.isArray([]); // true
```

### Coercion

Implicit coercion converts values automatically depending on the operator/context.

```js
"5" + 2; // "52" because + can concatenate
"5" - 2; // 3 because - converts operands to numbers
Boolean(0); // false
Boolean("0"); // true
```

Falsy values are: `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, and `NaN`.

Prefer explicit conversion at system boundaries:

```js
const quantity = Number(input.value);
const enabled = Boolean(config.enabled);
const label = String(id);
```

**Common mistake:** using `==` without understanding coercion. Prefer `===` unless loose equality is deliberately required.

## 1.3 Operators

Important categories include arithmetic, assignment, comparison, logical, bitwise, ternary, and type-related operators.

```js
const total = price * quantity;
const isAdult = age >= 18;
const canProceed = isLoggedIn && hasPermission;
const displayName = user.name || "Guest";
const status = score >= 60 ? "pass" : "fail";
```

`&&` and `||` return operands, not necessarily booleans.

```js
"hello" && 42; // 42
0 || 10;       // 10
```

Prefer nullish coalescing when `0`, `false`, or `""` are valid values:

```js
const retries = options.retries ?? 3;
```

**Interview angle:** know short-circuit evaluation, `===` vs `==`, and the difference between `||` and `??`.

## 1.4 Conditionals and Loops

Use conditionals to select behavior and loops to repeat behavior.

```js
if (temperature > 40) {
  console.log("Very hot");
} else if (temperature > 30) {
  console.log("Hot");
} else {
  console.log("Normal");
}
```

`switch` is useful when comparing one expression against multiple discrete values.

```js
switch (status) {
  case "pending":
    queueJob();
    break;
  case "complete":
    archiveJob();
    break;
  default:
    logUnknownStatus(status);
}
```

Common loops:

```js
for (let i = 0; i < items.length; i++) {
  console.log(items[i]);
}

for (const item of items) {
  console.log(item);
}

for (const key in object) {
  console.log(key, object[key]);
}

while (queue.length) {
  process(queue.shift());
}
```

Prefer array methods such as `map`, `filter`, and `reduce` when they clearly express data transformation. Use loops when control flow (`break`, `continue`, early exit) matters.

**Common mistake:** using `for...in` for arrays. It iterates enumerable property keys, not array values.

## 1.5 Functions: Declaration, Expression, Arrow

Functions are first-class values: they can be assigned, passed, returned, and stored in data structures.

### Declaration

```js
function add(a, b) {
  return a + b;
}
```

Function declarations are hoisted with their implementation.

### Expression

```js
const subtract = function (a, b) {
  return a - b;
};
```

### Arrow function

```js
const multiply = (a, b) => a * b;
```

Arrow functions do not create their own `this`, `arguments`, `super`, or `new.target`; they capture lexical `this`.

```js
const user = {
  name: "Ava",
  normal() {
    return this.name;
  },
  brokenArrow: () => this.name
};
```

Use normal methods when dynamic receiver-based `this` is required. Use arrows heavily for callbacks and lexical context.

## 1.6 Scope and Hoisting

JavaScript uses lexical scope: accessible variables are determined by where code is written, not where a function is called.

```js
const globalValue = "global";

function outer() {
  const functionValue = "function";

  if (true) {
    const blockValue = "block";
    console.log(globalValue, functionValue, blockValue);
  }
}
```

### Hoisting

Declarations are processed before execution, but initialization rules differ.

```js
console.log(a); // undefined
var a = 10;

// console.log(b); // ReferenceError: TDZ
let b = 20;

sayHi(); // works
function sayHi() {
  console.log("Hi");
}
```

Think of hoisting as creation of bindings during environment setup, not literal movement of source code.

## 1.7 Template Literals

Template literals support interpolation, multiline strings, and tagged templates.

```js
const user = "Ava";
const message = `Hello ${user},
welcome back.`;
```

Tagged templates allow custom processing.

```js
function upper(strings, value) {
  return `${strings[0]}${String(value).toUpperCase()}${strings[1]}`;
}

const result = upper`Hello ${"world"}!`;
```

Use template literals for readable interpolation; do not build untrusted HTML directly from user input.

---

# 2. Core Concepts

## 2.1 Closures

A closure is a function together with access to the lexical environment in which that function was created. The inner function can continue to access outer variables even after the outer function has returned.

```js
function createCounter() {
  let count = 0;

  return function increment() {
    count += 1;
    return count;
  };
}

const counter = createCounter();
counter(); // 1
counter(); // 2
```

Closures power data privacy, factories, memoization, callbacks, configuration capture, and module-like patterns.

```js
function createApiClient(baseUrl) {
  return async function request(path) {
    return fetch(`${baseUrl}${path}`);
  };
}

const usersApi = createApiClient("/api/users");
```

**Cost:** captured data may remain reachable longer than expected. Avoid retaining large object graphs unnecessarily.

**Interview angle:** explain why `count` survives after `createCounter()` returns and how closures relate to lexical scope.

## 2.2 `this` Keyword and Binding (`call`, `apply`, `bind`)

`this` is determined primarily by the call site for normal functions.

```js
const account = {
  owner: "Ava",
  getOwner() {
    return this.owner;
  }
};

account.getOwner(); // "Ava"
```

Rules, simplified:

1. `new Fn()` -> new instance.
2. `fn.call(obj)` / `fn.apply(obj)` / bound function -> explicit binding.
3. `obj.fn()` -> `obj` is the receiver.
4. Plain function call -> `undefined` in strict mode (or global object in legacy non-strict browser code).
5. Arrow function -> lexical `this` from surrounding scope.

```js
function greet(greeting, punctuation) {
  return `${greeting} ${this.name}${punctuation}`;
}

const person = { name: "Ava" };

greet.call(person, "Hi", "!");
greet.apply(person, ["Hello", "."]);

const boundGreet = greet.bind(person, "Welcome");
boundGreet("!");
```

`call` invokes immediately with comma-separated arguments; `apply` invokes immediately with an argument array; `bind` returns a new function.

## 2.3 Prototypes and Prototypal Inheritance

Objects can delegate property lookup to another object through their internal prototype (`[[Prototype]]`).

```js
const animal = {
  speak() {
    return "sound";
  }
};

const dog = Object.create(animal);
dog.bark = () => "woof";

dog.speak(); // inherited through prototype chain
```

Constructor functions expose a `.prototype` object used for instances created with `new`.

```js
function User(name) {
  this.name = name;
}

User.prototype.greet = function () {
  return `Hi ${this.name}`;
};

const user = new User("Ava");
```

ES6 classes are cleaner syntax over prototype-based semantics; they do not replace the prototype model.

**Best practice:** prefer composition when inheritance creates rigid hierarchies or implicit coupling.

## 2.4 Execution Context and Call Stack

Before executing code, JavaScript establishes an execution context containing lexical bindings, environment references, and `this` information. Function invocation creates a new function execution context.

The call stack tracks active execution contexts in LIFO order.

```js
function third() {
  console.log("done");
}

function second() {
  third();
}

function first() {
  second();
}

first();
```

Conceptual stack at `console.log`:

```text
Global
  -> first
    -> second
      -> third
```

Infinite recursion eventually produces a stack overflow because stack frames cannot grow indefinitely.

## 2.5 Event Loop, Microtasks, and Macrotasks

JavaScript execution is single-threaded at the language level, while host environments provide APIs for timers, networking, I/O, and rendering. The event loop coordinates queued work after the current stack becomes empty.

Microtasks (such as Promise reactions and `queueMicrotask`) are drained before the next task/macrotask (such as timers).

```js
console.log("A");

setTimeout(() => console.log("timeout"), 0);

Promise.resolve().then(() => console.log("promise"));

console.log("B");
```

Output:

```text
A
B
promise
timeout
```

Too many microtasks can starve timers/rendering. CPU-heavy synchronous work still blocks the main thread.

## 2.6 Higher-Order Functions

A higher-order function accepts a function, returns a function, or both.

```js
function withLogging(fn) {
  return (...args) => {
    console.log("arguments:", args);
    return fn(...args);
  };
}

const add = withLogging((a, b) => a + b);
```

Array methods (`map`, `filter`, `reduce`) are common higher-order functions.

Use them to separate policy/behavior from orchestration, but avoid nesting functional abstractions until code becomes harder to read than equivalent imperative code.

## 2.7 IIFE

An Immediately Invoked Function Expression executes as soon as it is defined.

```js
(() => {
  const internal = "private";
  console.log(internal);
})();
```

Before ES modules and block scope became standard, IIFEs were widely used to avoid leaking variables into global scope. Today they remain useful for isolated one-time initialization but are less central.

---

# 3. Data Structures & Built-ins

## 3.1 Arrays and Methods

Arrays are ordered, zero-indexed objects optimized for list-like data.

```js
const numbers = [1, 2, 3, 4, 5];

const doubled = numbers.map(n => n * 2);
const evens = numbers.filter(n => n % 2 === 0);
const total = numbers.reduce((sum, n) => sum + n, 0);
const found = numbers.find(n => n > 3);
const hasEven = numbers.some(n => n % 2 === 0);
const allPositive = numbers.every(n => n > 0);
```

Mutating methods include `push`, `pop`, `shift`, `unshift`, `splice`, `sort`, and `reverse`. Non-mutating alternatives include `map`, `filter`, `slice`, `concat`, and newer copying methods such as `toSorted()` where supported.

```js
const sorted = numbers.toSorted((a, b) => b - a);
```

**Common mistake:** `sort()` sorts strings by default.

```js
[2, 10, 1].sort();             // [1, 10, 2]
[2, 10, 1].sort((a, b) => a-b); // [1, 2, 10]
```

## 3.2 Objects and Object Methods

Objects model key-value records. Property keys are strings or symbols.

```js
const user = {
  id: 1,
  name: "Ava",
  active: true
};

Object.keys(user);
Object.values(user);
Object.entries(user);
Object.hasOwn(user, "name");
```

Transform with entries:

```js
const prices = { book: 100, pen: 20 };

const withTax = Object.fromEntries(
  Object.entries(prices).map(([key, value]) => [key, value * 1.18])
);
```

Use `Object.hasOwn()` rather than relying on `obj.hasOwnProperty()` when objects may have unusual prototypes.

## 3.3 Strings and Methods

Strings are immutable sequences of UTF-16 code units.

```js
const value = "  JavaScript Guide  ";

value.trim();
value.toLowerCase();
value.includes("Script");
value.startsWith("  Java");
value.replace("Guide", "Notes");
value.split(" ");
value.slice(2, 12);
```

Every transformation returns a new string.

Be cautious with Unicode: `.length` counts UTF-16 code units, not necessarily user-perceived characters.

```js
[..."👍"].length; // 1 code point in this case
```

## 3.4 `Map`, `Set`, `WeakMap`, `WeakSet`

### Map

`Map` supports keys of any type and preserves insertion order.

```js
const cache = new Map();
const key = { id: 1 };
cache.set(key, "result");
cache.get(key);
```

Use `Map` when keys are dynamic/non-string, frequent insertion/deletion is needed, or map semantics are clearer than plain objects.

### Set

`Set` stores unique values.

```js
const uniqueIds = new Set([1, 2, 2, 3]);
const deduped = [...new Set([1, 1, 2, 3])];
```

### WeakMap / WeakSet

Weak collections hold weak references to object keys/items and are not enumerable. They allow garbage collection when no other strong reference exists.

```js
const privateMetadata = new WeakMap();

function register(obj, metadata) {
  privateMetadata.set(obj, metadata);
}
```

Useful for metadata tied to object lifetime.

## 3.5 JSON

JSON is a text interchange format, not a complete representation of JavaScript values.

```js
const text = JSON.stringify({ id: 1, active: true });
const data = JSON.parse(text);
```

JSON supports objects, arrays, strings, numbers, booleans, and `null`. It does not directly represent functions, `undefined`, symbols, `BigInt`, `Map`, `Set`, or cyclic references.

```js
JSON.stringify({ value: undefined }); // "{}"
```

Always treat parsed external JSON as untrusted input and validate shape before business use.

---

# 4. ES6+ Features

## 4.1 `let` / `const` and Arrow Functions

Modern JavaScript favors block-scoped bindings and concise lexical callbacks.

```js
const multiplier = 2;
const values = [1, 2, 3];
const doubled = values.map(value => value * multiplier);
```

Use arrows when lexical `this` is desired; do not use arrows as a universal replacement for methods or constructor-capable functions.

## 4.2 Destructuring

Destructuring extracts values from arrays and objects declaratively.

```js
const user = { id: 1, name: "Ava", role: "admin" };
const { name, role } = user;

const coordinates = [28.6, 77.2];
const [latitude, longitude] = coordinates;
```

Aliases and defaults:

```js
const { name: displayName, theme = "light" } = user;
```

Function parameters:

```js
function createUser({ name, role = "user" }) {
  return { name, role };
}
```

Avoid deeply nested destructuring when it obscures validation and nullability.

## 4.3 Spread and Rest Operators

The same `...` syntax means different things depending on position.

### Spread

```js
const first = [1, 2];
const second = [3, 4];
const all = [...first, ...second];

const updatedUser = { ...user, active: true };
```

Spread is shallow; nested objects remain shared unless copied separately.

### Rest

```js
function sum(...numbers) {
  return numbers.reduce((total, n) => total + n, 0);
}

const { password, ...publicUser } = user;
```

## 4.4 Default Parameters

Defaults apply when the incoming value is `undefined`, not `null`.

```js
function connect(host = "localhost", port = 3000) {
  return `${host}:${port}`;
}

connect(undefined, 8080); // localhost:8080
connect(null, 8080);      // null:8080
```

Default expressions are evaluated at call time.

## 4.5 Classes and Inheritance

Classes provide declarative syntax around constructor/prototype mechanics.

```js
class User {
  constructor(name) {
    this.name = name;
  }

  greet() {
    return `Hi ${this.name}`;
  }
}

class Admin extends User {
  constructor(name, permissions) {
    super(name);
    this.permissions = permissions;
  }

  can(permission) {
    return this.permissions.includes(permission);
  }
}
```

Use inheritance when there is a stable "is-a" relationship. Prefer composition for flexible behavior assembly.

Private fields use `#`:

```js
class Counter {
  #value = 0;

  increment() {
    return ++this.#value;
  }
}
```

## 4.6 Modules (`import` / `export`)

ES modules give each file its own scope and enable static dependency analysis.

```js
// math.js
export function add(a, b) {
  return a + b;
}

export const PI = 3.14159;
```

```js
// app.js
import { add, PI } from "./math.js";
```

Default export:

```js
export default class ApiClient {}
```

```js
import ApiClient from "./ApiClient.js";
```

Prefer named exports for discoverability and refactoring in shared libraries unless a default export meaningfully represents the module's single primary value.

## 4.7 Optional Chaining and Nullish Coalescing

Optional chaining short-circuits when the left side is `null` or `undefined`.

```js
const city = user.address?.city;
const firstItem = order.items?.[0];
const result = callbacks.onSuccess?.();
```

Nullish coalescing provides a fallback only for `null`/`undefined`.

```js
const pageSize = settings.pageSize ?? 25;
```

This differs from `||`, which also treats `0`, `false`, and `""` as fallback conditions.

## 4.8 Generators and Iterators

An iterator implements a `next()` method returning `{ value, done }`. Iterable objects expose an iterator through `Symbol.iterator`.

```js
const range = {
  from: 1,
  to: 3,
  *[Symbol.iterator]() {
    for (let value = this.from; value <= this.to; value++) {
      yield value;
    }
  }
};

for (const value of range) {
  console.log(value);
}
```

Generators pause/resume execution around `yield`.

```js
function* ids() {
  let id = 1;
  while (true) {
    yield id++;
  }
}

const generator = ids();
generator.next().value; // 1
generator.next().value; // 2
```

Useful for lazy sequences, custom traversal, and stateful iteration.

---

# 5. Asynchronous JavaScript

## 5.1 Callbacks and Callback Hell

Callbacks pass continuation behavior into another function.

```js
function loadUser(id, callback) {
  database.get(id, (error, user) => {
    if (error) return callback(error);
    callback(null, user);
  });
}
```

Deeply nested dependent callbacks create "callback hell": difficult error propagation, control flow, and readability.

```js
getUser(id, (err, user) => {
  if (err) return handle(err);
  getOrders(user.id, (err, orders) => {
    if (err) return handle(err);
    getInvoice(orders[0].id, (err, invoice) => {
      if (err) return handle(err);
      render(invoice);
    });
  });
});
```

Promises and `async`/`await` address composition, not the underlying asynchronous nature itself.

## 5.2 Promises

A Promise represents a future fulfilled or rejected result.

```js
const promise = new Promise((resolve, reject) => {
  performTask((error, value) => {
    if (error) reject(error);
    else resolve(value);
  });
});

promise
  .then(value => transform(value))
  .catch(error => handle(error))
  .finally(() => cleanup());
```

### Promise combinators

```js
await Promise.all([fetchUser(), fetchOrders()]);
await Promise.race([request(), timeoutPromise]);
await Promise.allSettled([taskA(), taskB()]);
```

- `Promise.all` fails fast if any input rejects.
- `Promise.race` settles with the first settled input.
- `Promise.allSettled` waits for all inputs and reports each outcome.

Do not wrap an existing Promise in `new Promise()` without a specific reason.

## 5.3 `async` / `await`

An `async` function always returns a Promise. `await` pauses that async function until the awaited value settles; it does not block the JavaScript thread.

```js
async function loadDashboard() {
  const [user, orders] = await Promise.all([
    fetchUser(),
    fetchOrders()
  ]);

  return { user, orders };
}
```

Avoid accidental serialization:

```js
// slower if requests are independent
const user = await fetchUser();
const orders = await fetchOrders();

// parallel
const [user2, orders2] = await Promise.all([
  fetchUser(),
  fetchOrders()
]);
```

## 5.4 Fetch API and AJAX

AJAX describes asynchronous browser-server communication; modern applications commonly use `fetch`.

```js
async function fetchUser(id) {
  const response = await fetch(`/api/users/${id}`);

  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }

  return response.json();
}
```

`fetch` rejects on network failures, not automatically on HTTP 4xx/5xx responses, so check `response.ok`.

Cancellation:

```js
const controller = new AbortController();

fetch("/api/search", { signal: controller.signal });
controller.abort();
```

## 5.5 Error Handling in Async Code

Use local handling when you can recover or add context; otherwise propagate errors upward.

```js
async function saveUser(user) {
  try {
    const response = await fetch("/api/users", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(user)
    });

    if (!response.ok) {
      throw new Error(`Save failed: ${response.status}`);
    }

    return await response.json();
  } catch (error) {
    logError(error);
    throw error;
  }
}
```

Always decide whether an operation requires retry, fallback, user notification, rollback, or failure propagation. Avoid empty `catch` blocks.

---

# 6. DOM & Browser

## 6.1 DOM Manipulation

The DOM exposes an object model of the page.

```js
const heading = document.querySelector("#title");
heading.textContent = "Updated title";

const item = document.createElement("li");
item.textContent = "New item";
document.querySelector("ul").append(item);
```

Prefer `textContent` for untrusted text. Assigning untrusted strings to `innerHTML` can create XSS vulnerabilities.

Batch DOM updates where practical; repeated layout-triggering reads/writes can cause performance problems.

## 6.2 Event Handling, Delegation, Bubbling, Capturing

Events travel through capture -> target -> bubble phases.

```js
button.addEventListener("click", event => {
  console.log(event.target);
});
```

Delegation attaches one handler to a stable ancestor:

```js
list.addEventListener("click", event => {
  const button = event.target.closest("button[data-id]");
  if (!button) return;

  removeItem(button.dataset.id);
});
```

Delegation is efficient for dynamic lists and reduces many individual listeners.

```js
parent.addEventListener("click", captureHandler, { capture: true });
```

Use `stopPropagation()` sparingly because it can interfere with other application behavior and delegated handlers.

## 6.3 Browser Storage

### `localStorage`

Persists across browser sessions for the same origin and exposes a synchronous string-only API.

```js
localStorage.setItem("theme", "dark");
const theme = localStorage.getItem("theme");
```

### `sessionStorage`

Scoped to a browser tab/session.

```js
sessionStorage.setItem("step", "2");
```

### Cookies

Cookies can be automatically attached to HTTP requests depending on attributes and browser rules. Authentication cookies should generally use server-set `HttpOnly`, `Secure`, and appropriate `SameSite` attributes.

Do not store sensitive secrets in `localStorage` merely for convenience; JavaScript-accessible storage is exposed if XSS occurs.

## 6.4 BOM (`window`, `navigator`, `location`)

The Browser Object Model exposes browser/environment capabilities beyond the document.

```js
window.innerWidth;
navigator.language;
location.href;
location.assign("/dashboard");
```

Other common browser APIs include `history`, `screen`, timers, observers, and performance APIs.

Feature-detect capabilities rather than relying only on user-agent strings.

```js
if ("geolocation" in navigator) {
  // capability is present
}
```

---

# 7. Advanced Topics

## 7.1 Debounce and Throttle

Debouncing delays execution until calls stop for a specified period. Useful for search input, resize handling, and autosave.

```js
function debounce(fn, delay) {
  let timer;

  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

const search = debounce(query => fetchResults(query), 300);
```

Throttling limits execution to at most once per interval.

```js
function throttle(fn, interval) {
  let lastRun = 0;

  return function (...args) {
    const now = Date.now();
    if (now - lastRun < interval) return;

    lastRun = now;
    fn.apply(this, args);
  };
}
```

Use throttle for continuous events where periodic updates are desired; debounce when only the final settled action matters.

## 7.2 Currying and Memoization

### Currying

Currying converts a multi-argument function into a sequence of single-argument functions.

```js
const multiply = a => b => a * b;
const double = multiply(2);
double(5); // 10
```

Useful for partial configuration and function composition, but excessive currying can reduce readability in typical product code.

### Memoization

Memoization caches function results based on arguments.

```js
function memoize(fn) {
  const cache = new Map();

  return arg => {
    if (cache.has(arg)) return cache.get(arg);
    const value = fn(arg);
    cache.set(arg, value);
    return value;
  };
}
```

Use only when computation is expensive, inputs repeat, and cache growth is controlled. Memoization trades memory for computation.

## 7.3 Design Patterns

### Module Pattern

Encapsulates private state and exposes a public API.

```js
const counter = (() => {
  let value = 0;

  return {
    increment: () => ++value,
    getValue: () => value
  };
})();
```

Modern ES modules often provide a cleaner module boundary.

### Singleton

Ensures one shared instance in a particular scope/runtime.

```js
class ConfigStore {
  constructor() {
    if (ConfigStore.instance) return ConfigStore.instance;
    ConfigStore.instance = this;
  }
}

export const configStore = new ConfigStore();
```

Use cautiously; global mutable singletons complicate testing and hidden dependencies.

### Observer

Subscribers react to published events.

```js
class EventBus {
  #listeners = new Map();

  on(event, listener) {
    const listeners = this.#listeners.get(event) ?? [];
    listeners.push(listener);
    this.#listeners.set(event, listeners);
  }

  emit(event, payload) {
    for (const listener of this.#listeners.get(event) ?? []) {
      listener(payload);
    }
  }
}
```

### Factory

Centralizes object creation and can hide concrete implementation details.

```js
function createLogger(environment) {
  return environment === "production"
    ? new RemoteLogger()
    : new ConsoleLogger();
}
```

Patterns are vocabulary for recurring design problems, not rules to force into every module.

## 7.4 Functional Programming Basics

Core ideas include pure functions, immutability, function composition, and minimizing side effects.

```js
const addTax = price => price * 1.18;
const format = value => `₹${value.toFixed(2)}`;
const formatPriceWithTax = price => format(addTax(price));
```

Pure function:

```js
function add(a, b) {
  return a + b;
}
```

Impure function:

```js
let total = 0;
function addToTotal(value) {
  total += value;
}
```

Side effects are necessary for real applications (networking, storage, UI); isolate them at boundaries rather than pretending they do not exist.

## 7.5 Recursion

Recursion solves a problem by reducing it to smaller instances of itself. It requires a base case.

```js
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1);
}
```

Tree traversal:

```js
function visit(node) {
  console.log(node.value);
  for (const child of node.children ?? []) {
    visit(child);
  }
}
```

Prefer iteration for very deep linear recursion in JavaScript because engines may hit call stack limits.

## 7.6 Error Handling (`try` / `catch`, Custom Errors)

Throw errors for exceptional conditions that should interrupt normal flow.

```js
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = "ValidationError";
    this.field = field;
  }
}

function createAccount(input) {
  if (!input.email) {
    throw new ValidationError("Email is required", "email");
  }
}
```

Handle errors at a layer that can take meaningful action.

```js
try {
  createAccount(input);
} catch (error) {
  if (error instanceof ValidationError) {
    showFieldError(error.field, error.message);
  } else {
    throw error;
  }
}
```

Do not use exceptions as routine branching logic.

---

# 8. Tooling & Ecosystem

## 8.1 npm / Yarn and `package.json`

Package managers install dependencies, run scripts, and resolve package graphs.

```json
{
  "name": "example-app",
  "version": "1.0.0",
  "scripts": {
    "dev": "vite",
    "test": "jest",
    "lint": "eslint ."
  },
  "dependencies": {
    "react": "^19.0.0"
  },
  "devDependencies": {
    "eslint": "^9.0.0"
  }
}
```

- `dependencies`: required at runtime by the package/application.
- `devDependencies`: development/build/test tooling.
- lockfiles (`package-lock.json`, `yarn.lock`) provide reproducible dependency resolution and should normally be committed.

Understand semantic versioning ranges before using `^`, `~`, or exact versions.

## 8.2 Bundlers: Webpack / Vite Basics

Bundlers/build tools resolve modules and transform assets for deployment.

Webpack builds a dependency graph and is highly configurable. Vite provides a fast development server using native ESM concepts and typically uses Rollup for production builds.

```js
// vite.config.js
import { defineConfig } from "vite";

export default defineConfig({
  build: {
    sourcemap: true
  }
});
```

Common concerns include code splitting, tree shaking, asset processing, environment variables, aliases, source maps, and production optimization.

Do not optimize bundle configuration without measuring bundle size and runtime impact.

## 8.3 Babel and Transpilation

Babel transforms newer JavaScript syntax into syntax compatible with configured target environments.

```js
// input
const greet = name => `Hi ${name}`;
```

A transpiler may output older-compatible syntax depending on targets.

Babel syntax transformation is different from polyfilling missing runtime APIs. A runtime without `Promise` still needs a compatible polyfill even if arrow syntax is transpiled.

Modern build tools often hide Babel/SWC/esbuild configuration behind framework defaults.

## 8.4 ESLint and Prettier

ESLint performs static analysis for potential bugs, unsafe patterns, and code conventions. Prettier primarily formats code consistently.

```js
// eslint.config.js (simplified)
export default [
  {
    rules: {
      "no-unused-vars": "error",
      "eqeqeq": "error"
    }
  }
];
```

Use linting for semantic/code-quality enforcement and formatting tooling for style. Keep automated rules practical enough that teams do not routinely disable them.

## 8.5 Testing: Jest / Mocha Basics

Tests verify behavior and provide regression protection.

```js
function add(a, b) {
  return a + b;
}

test("adds two numbers", () => {
  expect(add(2, 3)).toBe(5);
});
```

Async test:

```js
test("loads a user", async () => {
  const user = await loadUser(1);
  expect(user.id).toBe(1);
});
```

Good tests focus on externally observable behavior rather than internal implementation details. Use the test pyramid pragmatically: many fast unit/component tests, fewer integration tests, and a targeted number of end-to-end tests.

---

# 9. Performance & Best Practices

## 9.1 Memory Leaks and Garbage Collection

JavaScript engines automatically reclaim memory that is no longer reachable. A memory leak occurs when objects remain reachable even though the application no longer needs them.

Common causes:

- forgotten event listeners
- uncleared intervals/timers
- unbounded caches
- global references
- closures retaining large objects
- detached DOM nodes still referenced by JavaScript

```js
function mount() {
  const handler = () => console.log("resize");
  window.addEventListener("resize", handler);

  return () => {
    window.removeEventListener("resize", handler);
  };
}
```

Weak collections can help when cache/metadata lifetime should follow object lifetime, but they are not a universal leak fix.

## 9.2 Performance Optimization

Optimize measured bottlenecks, not intuition.

High-impact areas often include:

- network payload and request count
- rendering frequency
- large DOM work
- expensive loops/algorithms
- repeated parsing/serialization
- unnecessary object allocation
- bundle size and startup work
- synchronous CPU-heavy work on the main thread

```js
const byId = new Map(users.map(user => [user.id, user]));

for (const order of orders) {
  order.user = byId.get(order.userId);
}
```

Using a `Map` avoids repeated `Array.find()` scans when joining many records, changing common lookup behavior from repeated linear scans to near-constant average-time lookups.

Use browser performance tooling, flame charts, memory profiles, Node profiling, and real-user metrics before/after optimization.

## 9.3 Clean Code Practices

Prefer code that makes invariants and intent obvious.

```js
// less expressive
if (u && u.a && u.r === "admin") {
  // ...
}

// clearer
const canManageUsers = user?.active && user.role === "admin";
if (canManageUsers) {
  // ...
}
```

Guidelines:

- keep functions focused on one coherent responsibility
- choose names that express business meaning
- minimize hidden mutation and shared mutable state
- validate at boundaries
- handle errors intentionally
- remove dead code rather than commenting it out indefinitely
- prefer simple control flow over clever abstractions
- separate domain logic from I/O/framework concerns where practical

DRY is useful, but premature abstraction can create worse coupling than a small amount of duplication.

## 9.4 Security Basics: XSS and CSRF

### XSS (Cross-Site Scripting)

XSS occurs when attacker-controlled content is executed as script in another user's browser.

Unsafe:

```js
container.innerHTML = userProvidedHtml;
```

Safer for plain text:

```js
container.textContent = userProvidedText;
```

Use context-aware output encoding/sanitization, framework-safe rendering defaults, Content Security Policy, and strict handling of dangerous HTML escape hatches.

### CSRF (Cross-Site Request Forgery)

CSRF tricks a browser into sending an authenticated request to another site when credentials are automatically included.

Typical defenses include:

- `SameSite` cookies
- CSRF tokens
- validating `Origin`/`Referer` where appropriate
- avoiding state-changing `GET` endpoints

XSS and CSRF are different threat models. CSRF tokens do not fix XSS, and XSS may bypass many application-level defenses because attacker code executes in the trusted origin.

Never treat client-side validation as a security boundary; enforce authorization and validation server-side.

---

# 10. Next Steps

## 10.1 TypeScript

TypeScript adds static type analysis on top of JavaScript and compiles/transpiles to JavaScript.

```ts
type User = {
  id: number;
  name: string;
  role: "admin" | "user";
};

function getDisplayName(user: User): string {
  return `${user.name} (${user.role})`;
}
```

Major concepts to study next:

- type inference
- unions/intersections
- interfaces and type aliases
- generics
- narrowing and discriminated unions
- utility types
- `unknown` vs `any`
- strict compiler options
- type-safe API boundaries

TypeScript improves developer-time guarantees; runtime validation is still required for untrusted external data because TypeScript types do not exist at runtime.

## 10.2 Move to React / Node.js

JavaScript fundamentals become the execution model beneath both frontend and backend frameworks.

### React direction

Apply closures, immutability, modules, async behavior, event handling, and functional composition to component architecture.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(current => current + 1)}>
      {count}
    </button>
  );
}
```

Important next concepts include component composition, state ownership, effects, memoization, rendering behavior, routing, server state, forms, testing, and performance.

### Node.js direction

Apply event-loop knowledge, async I/O, modules, error handling, security, and data modeling to server-side applications.

```js
import express from "express";

const app = express();
app.use(express.json());

app.get("/health", (req, res) => {
  res.json({ status: "ok" });
});

app.listen(3000);
```

Important next concepts include HTTP, Express/Fastify, middleware, validation, authentication/authorization, database access, caching, streams, worker threads, observability, testing, deployment, and production security.

---

# Cross-Cutting Interview Checklist

For senior JavaScript interviews, be able to explain—not just define—the following:

- lexical scope, closures, and retained environments
- `this` binding rules and arrow-function lexical `this`
- prototype chain resolution and class semantics
- execution contexts, stack behavior, event loop, microtasks, and tasks
- Promise error propagation and concurrency patterns
- mutation vs immutability and shallow vs deep copying
- `Map`/`Set` trade-offs compared with arrays/objects
- browser event propagation and delegation
- memory leak patterns and profiling strategy
- XSS/CSRF threat models and mitigations
- module boundaries, package management, bundling, transpilation, linting, and testing
- when an abstraction improves maintainability versus merely hiding straightforward logic
