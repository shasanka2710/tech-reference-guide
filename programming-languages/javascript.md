# 🟨 JavaScript Quick Reference

## Variables

```js
var x = 1;    // function-scoped, hoisted (avoid)
let y = 2;    // block-scoped, reassignable
const z = 3;  // block-scoped, not reassignable
```

## Data Types

```js
// Primitives
typeof 42          // "number"
typeof "hello"     // "string"
typeof true        // "boolean"
typeof undefined   // "undefined"
typeof null        // "object" (quirk!)
typeof Symbol()    // "symbol"
typeof 42n         // "bigint"

// Objects
typeof {}          // "object"
typeof []          // "object"
typeof function(){} // "function"
```

## Arrays

```js
const arr = [1, 2, 3, 4, 5];

// Mutating
arr.push(6);         // add to end
arr.pop();           // remove from end
arr.unshift(0);      // add to start
arr.shift();         // remove from start
arr.splice(1, 2);    // remove 2 at index 1

// Non-mutating (returns new array)
arr.slice(1, 3);
arr.concat([6, 7]);
arr.map(x => x * 2);
arr.filter(x => x > 2);
arr.reduce((acc, x) => acc + x, 0);
arr.find(x => x > 3);
arr.findIndex(x => x > 3);
arr.some(x => x > 4);
arr.every(x => x > 0);
arr.includes(3);
arr.flat();          // flatten nested
arr.flatMap(x => [x, x * 2]);

// Sorting
arr.sort((a, b) => a - b);   // ascending
arr.sort((a, b) => b - a);   // descending
```

## Objects

```js
const obj = { name: "Alice", age: 30 };

// Access
obj.name;
obj["name"];

// Destructuring
const { name, age = 25 } = obj;

// Spread
const copy = { ...obj, city: "NYC" };

// Object methods
Object.keys(obj);
Object.values(obj);
Object.entries(obj);
Object.assign({}, obj, { extra: 1 });
Object.freeze(obj);     // immutable

// Optional chaining
obj?.address?.street;

// Nullish coalescing
const val = obj.value ?? "default";
```

## Functions

```js
// Declaration
function add(a, b) { return a + b; }

// Expression
const multiply = function(a, b) { return a * b; };

// Arrow function
const square = x => x * x;
const greet  = (name) => `Hello, ${name}!`;
const noop   = () => {};

// Default params
function greet(name = "World") { ... }

// Rest params
function sum(...nums) { return nums.reduce((a, b) => a + b, 0); }

// Spread
Math.max(...[1, 2, 3]);

// Destructuring params
function display({ name, age }) { ... }
```

## Classes

```js
class Animal {
    #name;  // private field

    constructor(name) {
        this.#name = name;
    }

    get name() { return this.#name; }

    speak() { return `${this.#name} makes a sound`; }

    static create(name) { return new Animal(name); }
}

class Dog extends Animal {
    speak() { return `${this.name} barks`; }
}

const d = new Dog("Rex");
```

## Promises & Async/Await

```js
// Promise
const p = new Promise((resolve, reject) => {
    setTimeout(() => resolve("done"), 1000);
});

p.then(val => console.log(val))
 .catch(err => console.error(err))
 .finally(() => console.log("finished"));

// Promise combinators
Promise.all([p1, p2, p3]);        // all resolve / any rejects
Promise.allSettled([p1, p2]);     // all settle
Promise.race([p1, p2]);           // first settles
Promise.any([p1, p2]);            // first resolves

// Async / Await
async function fetchData(url) {
    try {
        const res  = await fetch(url);
        const data = await res.json();
        return data;
    } catch (err) {
        console.error(err);
    }
}
```

## Modules (ES6)

```js
// Export
export const PI = 3.14;
export function add(a, b) { return a + b; }
export default class MyClass { ... }

// Import
import MyClass from "./my-class.js";
import { PI, add } from "./utils.js";
import * as utils from "./utils.js";
```

## Destructuring & Spread

```js
// Array destructuring
const [first, second, ...rest] = [1, 2, 3, 4, 5];

// Object destructuring with rename
const { name: personName, age: personAge } = person;

// Nested
const { address: { city } } = person;

// Swap
[a, b] = [b, a];
```

## Iterators & Generators

```js
function* counter(start = 0) {
    while (true) yield start++;
}

const gen = counter(1);
gen.next(); // { value: 1, done: false }

// for...of
for (const val of [1, 2, 3]) { }
for (const [k, v] of map) { }
for (const char of "hello") { }
```

## Error Handling

```js
try {
    throw new Error("Something went wrong");
} catch (err) {
    console.error(err.message);
    console.error(err.stack);
} finally {
    cleanup();
}

// Custom error
class ValidationError extends Error {
    constructor(message) {
        super(message);
        this.name = "ValidationError";
    }
}
```

## Useful Methods

```js
// String
"hello world".toUpperCase()
"  hi  ".trim()
"a,b,c".split(",")
["a","b"].join("-")
"hello".includes("ell")
"repeat".repeat(3)
`Template ${literal}`

// Number
Number.isInteger(x)
Number.isFinite(x)
Number.isNaN(x)
parseInt("42px")
parseFloat("3.14")
(3.14159).toFixed(2)

// Math
Math.floor(3.7)   // 3
Math.ceil(3.2)    // 4
Math.round(3.5)   // 4
Math.abs(-5)      // 5
Math.max(1,2,3)   // 3
Math.pow(2, 10)   // 1024
Math.sqrt(16)     // 4
Math.random()     // 0..1

// JSON
JSON.stringify(obj, null, 2)
JSON.parse(str)

// Date
new Date()
Date.now()
new Date("2024-01-01")
```
