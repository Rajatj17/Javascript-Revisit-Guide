# JavaScript Interview Revision Guide

## 1. Core Language Fundamentals

### Variable Declarations
```javascript
var x = 1;    // Function-scoped, hoisted, can be redeclared
let y = 2;    // Block-scoped, temporal dead zone, cannot be redeclared
const z = 3;  // Block-scoped, must be initialized, immutable binding
```

**Key Points:**
- `var` is hoisted and initialized with `undefined`
- `let/const` are hoisted but not initialized (temporal dead zone)
- `const` objects/arrays are mutable, but the binding is immutable

### Data Types & Type Coercion
**Primitives:** string, number, boolean, undefined, null, symbol, bigint
**Non-primitives:** object (including arrays, functions, dates)

```javascript
// Type coercion examples
console.log([] + []);        // ""
console.log([] + {});        // "[object Object]"
console.log({} + []);        // 0 (in some contexts)
console.log("5" + 3);        // "53"
console.log("5" - 3);        // 2
console.log(!!0);            // false
console.log(!!"");           // false
```

## 2. Functions & Scope

### Function Types
```javascript
// Function declaration (hoisted)
function foo() { return "declaration"; }

// Function expression (not hoisted)
const bar = function() { return "expression"; };

// Arrow function (lexical this, no hoisting)
const baz = () => "arrow";

// IIFE (Immediately Invoked Function Expression)
(function() { console.log("IIFE"); })();
```

### Closures
```javascript
function outer(x) {
    return function inner(y) {
        return x + y; // inner has access to x
    };
}
const addFive = outer(5);
console.log(addFive(3)); // 8
```

**Interview Questions:**
- What variables does the inner function have access to?
- What happens to outer's variables after outer returns?

## 3. `this` Keyword & Binding

### Context Rules
1. **Default binding:** `this` = global object (or undefined in strict mode)
2. **Implicit binding:** `obj.method()` → `this` = obj
3. **Explicit binding:** `call()`, `apply()`, `bind()`
4. **Arrow functions:** Inherit `this` from enclosing scope

```javascript
const obj = {
    name: "Alice",
    greet() {
        console.log(`Hi, I'm ${this.name}`);
        
        const arrow = () => {
            console.log(`Arrow: ${this.name}`); // Still "Alice"
        };
        
        function regular() {
            console.log(`Regular: ${this.name}`); // undefined or global
        }
        
        arrow();
        regular();
    }
};
```

## 4. Prototypes & Inheritance

### Prototype Chain
```javascript
function Person(name) {
    this.name = name;
}

Person.prototype.greet = function() {
    return `Hello, I'm ${this.name}`;
};

const alice = new Person("Alice");
console.log(alice.greet()); // "Hello, I'm Alice"

// Prototype chain: alice → Person.prototype → Object.prototype → null
```

### ES6 Classes
```javascript
class Animal {
    constructor(name) {
        this.name = name;
    }
    
    speak() {
        return `${this.name} makes a sound`;
    }
}

class Dog extends Animal {
    speak() {
        return `${this.name} barks`;
    }
}
```

## 5. Asynchronous JavaScript

### Promises
```javascript
const promise = new Promise((resolve, reject) => {
    setTimeout(() => resolve("Done!"), 1000);
});

promise
    .then(result => console.log(result))
    .catch(error => console.error(error))
    .finally(() => console.log("Cleanup"));
```

### Async/Await
```javascript
async function fetchData() {
    try {
        const response = await fetch('/api/data');
        const data = await response.json();
        return data;
    } catch (error) {
        console.error('Error:', error);
    }
}

// Parallel execution
async function fetchMultiple() {
    const [users, posts] = await Promise.all([
        fetch('/users'),
        fetch('/posts')
    ]);
}
```

### Event Loop Concepts
```javascript
console.log('1');

setTimeout(() => console.log('2'), 0);

Promise.resolve().then(() => console.log('3'));

console.log('4');

// Output: 1, 4, 3, 2
// Microtasks (Promises) have priority over macrotasks (setTimeout)
```

## 6. Array Methods (Functional Programming)

### Essential Methods
```javascript
const numbers = [1, 2, 3, 4, 5];

// Transform
const doubled = numbers.map(n => n * 2);

// Filter
const evens = numbers.filter(n => n % 2 === 0);

// Reduce
const sum = numbers.reduce((acc, n) => acc + n, 0);

// Find
const found = numbers.find(n => n > 3);

// Check conditions
const hasEven = numbers.some(n => n % 2 === 0);
const allPositive = numbers.every(n => n > 0);

// Side effects
numbers.forEach(n => console.log(n));
```

## 7. Object Manipulation

### Object Methods
```javascript
const obj = { a: 1, b: 2, c: 3 };

Object.keys(obj);        // ['a', 'b', 'c']
Object.values(obj);      // [1, 2, 3]
Object.entries(obj);     // [['a', 1], ['b', 2], ['c', 3]]

// Destructuring
const { a, b, ...rest } = obj;

// Spread
const newObj = { ...obj, d: 4 };

// Property shorthand
const name = "Alice";
const person = { name, age: 30 }; // { name: "Alice", age: 30 }
```

## 8. ES6+ Features

### Destructuring
```javascript
// Arrays
const [first, second, ...rest] = [1, 2, 3, 4, 5];

// Objects
const { name, age, city = "Unknown" } = person;

// Function parameters
function greet({ name, age }) {
    return `${name} is ${age} years old`;
}
```

### Template Literals
```javascript
const name = "World";
const message = `Hello, ${name}!
This is a multi-line string.`;
```

### Modules
```javascript
// export
export const PI = 3.14159;
export default function calculate() { /* ... */ }

// import
import calculate, { PI } from './math.js';
import * as MathUtils from './math.js';
```

## 9. Common Interview Patterns

### Debouncing
```javascript
function debounce(func, delay) {
    let timeoutId;
    return function(...args) {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => func.apply(this, args), delay);
    };
}

const debouncedSearch = debounce(search, 300);
```

### Throttling
```javascript
function throttle(func, limit) {
    let inThrottle;
    return function(...args) {
        if (!inThrottle) {
            func.apply(this, args);
            inThrottle = true;
            setTimeout(() => inThrottle = false, limit);
        }
    };
}
```

### Memoization
```javascript
function memoize(fn) {
    const cache = {};
    return function(...args) {
        const key = JSON.stringify(args);
        if (key in cache) {
            return cache[key];
        }
        const result = fn.apply(this, args);
        cache[key] = result;
        return result;
    };
}
```

### Currying
```javascript
function curry(fn) {
    return function curried(...args) {
        if (args.length >= fn.length) {
            return fn.apply(this, args);
        }
        return function(...nextArgs) {
            return curried(...args, ...nextArgs);
        };
    };
}

const add = (a, b, c) => a + b + c;
const curriedAdd = curry(add);
console.log(curriedAdd(1)(2)(3)); // 6
```

## 10. Error Handling

### Try-Catch with Async Code
```javascript
// Promises
promise
    .catch(error => {
        console.error('Promise error:', error);
        throw new Error('Custom error');
    });

// Async/Await
async function handleAsync() {
    try {
        await riskyOperation();
    } catch (error) {
        console.error('Async error:', error);
    }
}

// Custom errors
class ValidationError extends Error {
    constructor(message) {
        super(message);
        this.name = 'ValidationError';
    }
}
```

## 11. Performance & Best Practices

### Memory Management
- Avoid global variables
- Remove event listeners
- Clear intervals/timeouts
- Use WeakMap for object metadata

### Common Gotchas
```javascript
// Loop closure problem
for (var i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 100); // Prints 3, 3, 3
}

// Solutions:
for (let i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 100); // Prints 0, 1, 2
}

// Or with closure:
for (var i = 0; i < 3; i++) {
    (function(j) {
        setTimeout(() => console.log(j), 100);
    })(i);
}
```

## Quick Interview Prep Checklist

**Practice these concepts:**
- [ ] Explain hoisting with examples
- [ ] Demonstrate closure with practical use case
- [ ] Show different ways to bind `this`
- [ ] Write a Promise chain and convert to async/await
- [ ] Implement debounce/throttle from scratch
- [ ] Explain prototype chain
- [ ] Demonstrate array methods (map, filter, reduce)
- [ ] Handle asynchronous errors properly
- [ ] Explain event loop with code example
- [ ] Use destructuring and spread operator

**Common Algorithm Questions:**
- Flatten nested arrays
- Find duplicate elements
- Implement array methods (map, filter, reduce)
- Deep clone objects
- Check if object is empty
- Merge objects/arrays
- Remove falsy values from arrays

Remember: Practice explaining your code verbally and be ready to discuss trade-offs and alternative approaches!