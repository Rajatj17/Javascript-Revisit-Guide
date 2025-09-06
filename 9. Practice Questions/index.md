# JavaScript Interview: Exercises & Tricky Questions

## 🔥 Most Frequently Asked Tricky Questions

### 1. Predict the Output (Hoisting & Scope)

**Question:**
```javascript
console.log(a);
console.log(b);
console.log(c);

var a = 1;
let b = 2;
const c = 3;

function test() {
    console.log(a);
    console.log(b);
    console.log(c);
    
    var a = 4;
    let b = 5;
    const c = 6;
}

test();
```

<details>
<summary>Answer</summary>

```
undefined
ReferenceError: Cannot access 'b' before initialization
ReferenceError: Cannot access 'c' before initialization
undefined
ReferenceError: Cannot access 'b' before initialization
ReferenceError: Cannot access 'c' before initialization
```

**Explanation:** `var` is hoisted and initialized with `undefined`. `let` and `const` are hoisted but not initialized (temporal dead zone).
</details>

---

### 2. The Classic Loop Closure Problem

**Question:**
```javascript
for (var i = 0; i < 3; i++) {
    setTimeout(() => {
        console.log(i);
    }, 100);
}

// What gets logged? How do you fix it?
```

<details>
<summary>Answer</summary>

**Output:** `3, 3, 3`

**Why:** All callbacks share the same `i` variable, which is `3` when callbacks execute.

**Fixes:**
```javascript
// Fix 1: Use let
for (let i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 100);
}

// Fix 2: IIFE
for (var i = 0; i < 3; i++) {
    (function(j) {
        setTimeout(() => console.log(j), 100);
    })(i);
}

// Fix 3: bind
for (var i = 0; i < 3; i++) {
    setTimeout(console.log.bind(null, i), 100);
}
```
</details>

---

### 3. 'this' Binding Confusion

**Question:**
```javascript
var obj = {
    name: 'Object',
    getName: function() {
        return this.name;
    }
};

var getName = obj.getName;
var getName2 = obj.getName.bind(obj);

console.log(obj.getName());     // ?
console.log(getName());         // ?
console.log(getName2());        // ?

var obj2 = { name: 'Object2', getName: obj.getName };
console.log(obj2.getName());    // ?
```

<details>
<summary>Answer</summary>

```
"Object"     // Implicit binding
undefined    // Default binding (global/undefined in strict)
"Object"     // Explicit binding with bind
"Object2"    // Implicit binding to obj2
```
</details>

---

### 4. Event Loop Order

**Question:**
```javascript
console.log('1');

setTimeout(() => console.log('2'), 0);

Promise.resolve().then(() => console.log('3'));

console.log('4');

setTimeout(() => console.log('5'), 0);

Promise.resolve().then(() => console.log('6'));

console.log('7');
```

<details>
<summary>Answer</summary>

**Output:** `1, 4, 7, 3, 6, 2, 5`

**Explanation:** 
1. Synchronous code runs first: `1, 4, 7`
2. Microtasks (Promises) run before macrotasks: `3, 6`
3. Macrotasks (setTimeout) run last: `2, 5`
</details>

---

### 5. Prototype Chain Confusion

**Question:**
```javascript
function Person(name) {
    this.name = name;
}

Person.prototype.greet = function() {
    return `Hello, I'm ${this.name}`;
};

var john = new Person('John');

console.log(john.greet());
console.log(john.hasOwnProperty('name'));
console.log(john.hasOwnProperty('greet'));
console.log('greet' in john);

delete Person.prototype.greet;
console.log(john.greet());
```

<details>
<summary>Answer</summary>

```
"Hello, I'm John"
true
false
true
TypeError: john.greet is not a function
```

**Explanation:** `name` is own property, `greet` is inherited from prototype. Deleting from prototype affects all instances.
</details>

---

## 💪 Coding Exercises

### Exercise 1: Implement Array.prototype.map()

**Question:** Implement your own version of `Array.prototype.map()`

```javascript
Array.prototype.myMap = function(callback, thisArg) {
    // Your implementation here
};

// Test cases
const numbers = [1, 2, 3, 4];
console.log(numbers.myMap(x => x * 2)); // [2, 4, 6, 8]
```

<details>
<summary>Solution</summary>

```javascript
Array.prototype.myMap = function(callback, thisArg) {
    if (this == null) {
        throw new TypeError('Array.prototype.myMap called on null or undefined');
    }
    
    if (typeof callback !== 'function') {
        throw new TypeError(callback + ' is not a function');
    }
    
    const O = Object(this);
    const len = parseInt(O.length) || 0;
    const result = new Array(len);
    
    for (let i = 0; i < len; i++) {
        if (i in O) {
            result[i] = callback.call(thisArg, O[i], i, O);
        }
    }
    
    return result;
};
```
</details>

---

### Exercise 2: Deep Clone Object

**Question:** Implement a deep clone function that handles circular references

```javascript
function deepClone(obj) {
    // Your implementation here
}

// Test cases
const original = {
    name: 'John',
    age: 30,
    hobbies: ['reading', 'swimming'],
    address: {
        city: 'New York',
        country: 'USA'
    }
};

original.self = original; // Circular reference

const cloned = deepClone(original);
console.log(cloned !== original); // true
console.log(cloned.address !== original.address); // true
```

<details>
<summary>Solution</summary>

```javascript
function deepClone(obj, visited = new WeakMap()) {
    // Handle primitives and null
    if (obj === null || typeof obj !== 'object') {
        return obj;
    }
    
    // Handle circular references
    if (visited.has(obj)) {
        return visited.get(obj);
    }
    
    // Handle Date
    if (obj instanceof Date) {
        return new Date(obj.getTime());
    }
    
    // Handle Array
    if (Array.isArray(obj)) {
        const clonedArray = [];
        visited.set(obj, clonedArray);
        
        for (let i = 0; i < obj.length; i++) {
            clonedArray[i] = deepClone(obj[i], visited);
        }
        
        return clonedArray;
    }
    
    // Handle Object
    const clonedObj = {};
    visited.set(obj, clonedObj);
    
    for (const key in obj) {
        if (obj.hasOwnProperty(key)) {
            clonedObj[key] = deepClone(obj[key], visited);
        }
    }
    
    return clonedObj;
}
```
</details>

---

### Exercise 3: Implement Promise.all()

**Question:** Create your own version of `Promise.all()`

```javascript
function promiseAll(promises) {
    // Your implementation here
}

// Test cases
const p1 = Promise.resolve(1);
const p2 = new Promise(resolve => setTimeout(() => resolve(2), 1000));
const p3 = Promise.resolve(3);

promiseAll([p1, p2, p3]).then(console.log); // [1, 2, 3]
```

<details>
<summary>Solution</summary>

```javascript
function promiseAll(promises) {
    return new Promise((resolve, reject) => {
        if (!Array.isArray(promises)) {
            reject(new TypeError('Input must be an array'));
            return;
        }
        
        if (promises.length === 0) {
            resolve([]);
            return;
        }
        
        const results = new Array(promises.length);
        let completedCount = 0;
        
        promises.forEach((promise, index) => {
            Promise.resolve(promise)
                .then(value => {
                    results[index] = value;
                    completedCount++;
                    
                    if (completedCount === promises.length) {
                        resolve(results);
                    }
                })
                .catch(error => {
                    reject(error);
                });
        });
    });
}
```
</details>

---

### Exercise 4: Debounce Function

**Question:** Implement a debounce function

```javascript
function debounce(func, delay) {
    // Your implementation here
}

// Test case
const debouncedLog = debounce(() => console.log('Called!'), 1000);
debouncedLog(); // Will be cancelled
debouncedLog(); // Will be cancelled  
debouncedLog(); // Will execute after 1000ms
```

<details>
<summary>Solution</summary>

```javascript
function debounce(func, delay) {
    let timeoutId;
    
    return function(...args) {
        const context = this;
        
        clearTimeout(timeoutId);
        
        timeoutId = setTimeout(() => {
            func.apply(context, args);
        }, delay);
    };
}

// Advanced version with immediate option
function advancedDebounce(func, delay, immediate = false) {
    let timeoutId;
    
    return function(...args) {
        const context = this;
        const callNow = immediate && !timeoutId;
        
        clearTimeout(timeoutId);
        
        timeoutId = setTimeout(() => {
            timeoutId = null;
            if (!immediate) func.apply(context, args);
        }, delay);
        
        if (callNow) func.apply(context, args);
    };
}
```
</details>

---

### Exercise 5: Flatten Nested Array

**Question:** Flatten a nested array to a specified depth

```javascript
function flatten(arr, depth = 1) {
    // Your implementation here
}

// Test cases
console.log(flatten([1, [2, [3, [4]]]])); // [1, 2, [3, [4]]]
console.log(flatten([1, [2, [3, [4]]]], 2)); // [1, 2, 3, [4]]
console.log(flatten([1, [2, [3, [4]]]], Infinity)); // [1, 2, 3, 4]
```

<details>
<summary>Solution</summary>

```javascript
function flatten(arr, depth = 1) {
    if (depth <= 0) return arr.slice();
    
    return arr.reduce((acc, item) => {
        if (Array.isArray(item)) {
            acc.push(...flatten(item, depth - 1));
        } else {
            acc.push(item);
        }
        return acc;
    }, []);
}

// Alternative using stack (iterative)
function flattenIterative(arr, depth = 1) {
    const stack = [...arr.map(item => [item, depth])];
    const result = [];
    
    while (stack.length > 0) {
        const [item, currentDepth] = stack.pop();
        
        if (Array.isArray(item) && currentDepth > 0) {
            stack.push(...item.map(subItem => [subItem, currentDepth - 1]));
        } else {
            result.push(item);
        }
    }
    
    return result.reverse();
}
```
</details>

---

## 🧠 Advanced Tricky Questions

### 6. Type Coercion Madness

**Question:**
```javascript
console.log([] + []);
console.log([] + {});
console.log({} + []);
console.log({} + {});
console.log(true + false);
console.log("5" + 3);
console.log("5" - 3);
console.log(+"5");
console.log(!!"");
console.log(!!0);
```

<details>
<summary>Answer</summary>

```
""                    // [] becomes "", "" + "" = ""
"[object Object]"     // [] becomes "", {} becomes "[object Object]"
0                     // In some contexts, {} is block, +[] becomes 0
"[object Object][object Object]"  // Both become "[object Object]"
1                     // true = 1, false = 0, 1 + 0 = 1
"53"                  // String concatenation
2                     // String to number conversion
5                     // Unary + converts to number
false                 // Empty string is falsy
false                 // 0 is falsy
```
</details>

---

### 7. Function Hoisting vs Variable Hoisting

**Question:**
```javascript
console.log(typeof foo);
console.log(typeof bar);

var foo = 'Hello';
function bar() {
    return 'World';
}

console.log(typeof foo);
console.log(typeof bar);

var foo = function() {
    return 'Hello';
};

console.log(typeof foo);
```

<details>
<summary>Answer</summary>

```
"undefined"    // var foo is hoisted, initialized with undefined
"function"     // function bar is fully hoisted
"string"       // foo is now "Hello"
"function"     // bar is still a function
"function"     // foo is now a function
```
</details>

---

### 8. Async/Await vs Promises

**Question:**
```javascript
async function test() {
    console.log('1');
    
    await Promise.resolve().then(() => console.log('2'));
    
    console.log('3');
}

console.log('4');
test();
console.log('5');
```

<details>
<summary>Answer</summary>

```
4
1
5
2
3
```

**Explanation:** 
1. `4` - synchronous
2. `1` - synchronous inside async function
3. `5` - synchronous (test() returns immediately)
4. `2` - microtask from Promise.resolve().then()
5. `3` - after await completes
</details>

---

### 9. Object Property Access

**Question:**
```javascript
const obj = {
    a: 1,
    b: 2,
    c: 3
};

const keys = ['a', 'b', 'c'];
const values = [];

for (var i = 0; i < keys.length; i++) {
    setTimeout(() => {
        values.push(obj[keys[i]]);
        if (values.length === keys.length) {
            console.log(values);
        }
    }, 100);
}
```

<details>
<summary>Answer</summary>

```
[undefined, undefined, undefined]
```

**Why:** Classic closure problem - all callbacks access `i` when it's 3, so `keys[3]` is undefined.

**Fix:**
```javascript
for (let i = 0; i < keys.length; i++) {
    setTimeout(() => {
        values.push(obj[keys[i]]);
        if (values.length === keys.length) {
            console.log(values); // [1, 2, 3]
        }
    }, 100);
}
```
</details>

---

### 10. Constructor Function Edge Case

**Question:**
```javascript
function User(name) {
    this.name = name;
    
    return { age: 25 };
}

const user1 = new User('John');
const user2 = User('Jane');

console.log(user1.name);
console.log(user1.age);
console.log(user2.name);
console.log(user2.age);
```

<details>
<summary>Answer</summary>

```
undefined
25
undefined
25
```

**Explanation:** When constructor returns an object, it overrides the `this` binding. Without `new`, it's just a regular function call.
</details>

---

## 🎯 Company-Specific Questions

### Google/Meta Style

**Question:** Implement a LRU Cache
```javascript
class LRUCache {
    constructor(capacity) {
        // Your implementation
    }
    
    get(key) {
        // Return value if exists, -1 if not
    }
    
    put(key, value) {
        // Add/update key-value pair
        // Remove least recently used if over capacity
    }
}

// Usage
const cache = new LRUCache(2);
cache.put(1, 1);
cache.put(2, 2);
console.log(cache.get(1)); // 1
cache.put(3, 3); // evicts key 2
console.log(cache.get(2)); // -1
```

<details>
<summary>Solution</summary>

```javascript
class LRUCache {
    constructor(capacity) {
        this.capacity = capacity;
        this.cache = new Map();
    }
    
    get(key) {
        if (this.cache.has(key)) {
            // Move to end (most recently used)
            const value = this.cache.get(key);
            this.cache.delete(key);
            this.cache.set(key, value);
            return value;
        }
        return -1;
    }
    
    put(key, value) {
        if (this.cache.has(key)) {
            // Update existing
            this.cache.delete(key);
        } else if (this.cache.size >= this.capacity) {
            // Remove least recently used (first item)
            const firstKey = this.cache.keys().next().value;
            this.cache.delete(firstKey);
        }
        
        this.cache.set(key, value);
    }
}
```
</details>

---

### Amazon Style

**Question:** Find the longest substring without repeating characters
```javascript
function lengthOfLongestSubstring(s) {
    // Your implementation
}

console.log(lengthOfLongestSubstring("abcabcbb")); // 3 ("abc")
console.log(lengthOfLongestSubstring("bbbbb")); // 1 ("b")
console.log(lengthOfLongestSubstring("pwwkew")); // 3 ("wke")
```

<details>
<summary>Solution</summary>

```javascript
function lengthOfLongestSubstring(s) {
    const seen = new Set();
    let left = 0;
    let maxLength = 0;
    
    for (let right = 0; right < s.length; right++) {
        while (seen.has(s[right])) {
            seen.delete(s[left]);
            left++;
        }
        
        seen.add(s[right]);
        maxLength = Math.max(maxLength, right - left + 1);
    }
    
    return maxLength;
}
```
</details>

---

## 📋 Quick Fire Round

### One-Liners to Explain:

1. **What's the difference between `==` and `===`?**
2. **Explain event bubbling and capturing**
3. **What is the difference between `null` and `undefined`?**
4. **How do you create a private variable in JavaScript?**
5. **What's the difference between `.call()`, `.apply()`, and `.bind()`?**

### Code Golf Challenges:

1. **Remove duplicates from array:** `[...new Set(arr)]`
2. **Get random array element:** `arr[Math.floor(Math.random() * arr.length)]`
3. **Check if number is even:** `num % 2 === 0`
4. **Capitalize first letter:** `str[0].toUpperCase() + str.slice(1)`
5. **Sum array of numbers:** `arr.reduce((a, b) => a + b, 0)`

## 🎪 Interview Simulation

### 30-Minute Coding Challenge:

**Build a simple event emitter that:**
- Supports `on(event, callback)` 
- Supports `emit(event, ...args)`
- Supports `off(event, callback)`
- Supports `once(event, callback)`

**Follow-up questions:**
- How would you handle memory leaks?
- How would you add namespace support?
- How would you make it async-safe?

## 🔒 Closure Deep Dive Questions

### Closure Question 1: Function Factory
**Question:** What will this output and why?
```javascript
function createCounter() {
    let count = 0;
    
    return function() {
        return ++count;
    };
}

const counter1 = createCounter();
const counter2 = createCounter();

console.log(counter1()); // ?
console.log(counter1()); // ?
console.log(counter2()); // ?
console.log(counter1()); // ?
```

<details>
<summary>Answer</summary>

```
1
2
1
3
```

**Explanation:** Each call to `createCounter()` creates a new execution context with its own `count` variable. The returned functions form closures over their respective `count` variables.
</details>

---

### Closure Question 2: Module Pattern Challenge
**Question:** Fix this broken module pattern:
```javascript
const Calculator = (function() {
    let result = 0;
    
    return {
        add: function(x) {
            result += x;
        },
        
        subtract: function(x) {
            result -= x;
        },
        
        getResult: function() {
            return result;
        },
        
        reset: function() {
            result = 0;
        }
    };
})();

Calculator.add(5);
Calculator.subtract(2);
console.log(Calculator.getResult()); // Should be 3

// But there's a problem - how can you make this chainable?
// Calculator.add(5).subtract(2).getResult() should work
```

<details>
<summary>Solution</summary>

```javascript
const Calculator = (function() {
    let result = 0;
    
    const calculator = {
        add: function(x) {
            result += x;
            return this; // Return this for chaining
        },
        
        subtract: function(x) {
            result -= x;
            return this;
        },
        
        multiply: function(x) {
            result *= x;
            return this;
        },
        
        divide: function(x) {
            if (x !== 0) {
                result /= x;
            }
            return this;
        },
        
        getResult: function() {
            return result; // Don't return this here
        },
        
        reset: function() {
            result = 0;
            return this;
        }
    };
    
    return calculator;
})();

// Usage
console.log(Calculator.add(10).subtract(3).multiply(2).getResult()); // 14
```
</details>

---

### Closure Question 3: Memory Leak Trap
**Question:** This code has a memory leak. Find it and fix it:
```javascript
function attachListeners() {
    const data = new Array(1000000).fill('data'); // Large array
    
    document.getElementById('button').addEventListener('click', function() {
        console.log('Button clicked');
    });
    
    return function() {
        return data.length;
    };
}

// Called multiple times
for (let i = 0; i < 100; i++) {
    attachListeners();
}
```

<details>
<summary>Solution</summary>

**Problem:** The event listener creates a closure that keeps the entire `data` array in memory, even though it's not used.

**Fix 1 - Clear unused references:**
```javascript
function attachListeners() {
    const data = new Array(1000000).fill('data');
    const dataLength = data.length; // Store what we need
    
    document.getElementById('button').addEventListener('click', function() {
        console.log('Button clicked');
    });
    
    // Clear the reference
    data.length = 0;
    
    return function() {
        return dataLength;
    };
}
```

**Fix 2 - Separate concerns:**
```javascript
function attachListeners() {
    const data = new Array(1000000).fill('data');
    
    // Move listener outside to avoid closure over data
    document.getElementById('button').addEventListener('click', handleClick);
    
    return function() {
        return data.length;
    };
}

function handleClick() {
    console.log('Button clicked');
}
```
</details>

---

### Closure Question 4: Advanced Loop Problem
**Question:** Explain the output and provide multiple fixes:
```javascript
const operations = [];

for (var i = 0; i < 3; i++) {
    operations.push({
        index: i,
        square: function() {
            return i * i;
        },
        cube: function() {
            return i * i * i;
        }
    });
}

operations.forEach((op, idx) => {
    console.log(`Index: ${op.index}, Square: ${op.square()}, Cube: ${op.cube()}`);
});
```

<details>
<summary>Answer & Solutions</summary>

**Output:**
```
Index: 3, Square: 9, Cube: 27
Index: 3, Square: 9, Cube: 27
Index: 3, Square: 9, Cube: 27
```

**Why:** All functions close over the same `i` variable which is 3 after the loop.

**Fix 1 - Use let:**
```javascript
const operations = [];

for (let i = 0; i < 3; i++) {
    operations.push({
        index: i,
        square: function() {
            return i * i;
        },
        cube: function() {
            return i * i * i;
        }
    });
}
```

**Fix 2 - IIFE:**
```javascript
const operations = [];

for (var i = 0; i < 3; i++) {
    operations.push((function(index) {
        return {
            index: index,
            square: function() {
                return index * index;
            },
            cube: function() {
                return index * index * index;
            }
        };
    })(i));
}
```

**Fix 3 - Factory function:**
```javascript
function createOperation(index) {
    return {
        index: index,
        square: function() {
            return index * index;
        },
        cube: function() {
            return index * index * index;
        }
    };
}

const operations = [];
for (var i = 0; i < 3; i++) {
    operations.push(createOperation(i));
}
```
</details>

---

## 🍛 Currying Challenges

### Currying Question 1: Basic Implementation
**Question:** Implement a curry function that works with any number of arguments:
```javascript
function curry(fn) {
    // Your implementation
}

// Test cases
function add(a, b, c) {
    return a + b + c;
}

const curriedAdd = curry(add);

console.log(curriedAdd(1)(2)(3)); // 6
console.log(curriedAdd(1, 2)(3)); // 6
console.log(curriedAdd(1)(2, 3)); // 6
console.log(curriedAdd(1, 2, 3)); // 6
```

<details>
<summary>Solution</summary>

```javascript
function curry(fn) {
    return function curried(...args) {
        if (args.length >= fn.length) {
            return fn.apply(this, args);
        } else {
            return function(...nextArgs) {
                return curried(...args, ...nextArgs);
            };
        }
    };
}

// Advanced version with better performance
function curryAdvanced(fn) {
    const arity = fn.length;
    
    return function curried(...args) {
        if (args.length >= arity) {
            return fn.apply(this, args);
        }
        
        return curried.bind(null, ...args);
    };
}
```
</details>

---

### Currying Question 2: Infinite Currying
**Question:** Create a function that can be called infinitely and returns the sum:
```javascript
// Should work like this:
console.log(add(1)(2)(3)(4)()); // 10
console.log(add(1)(2)()); // 3
console.log(add(5)()); // 5
console.log(add()); // 0
```

<details>
<summary>Solution</summary>

```javascript
function add(a) {
    let sum = a || 0;
    
    function inner(b) {
        if (b === undefined) {
            return sum;
        }
        sum += b;
        return inner;
    }
    
    return inner;
}

// Alternative using closure
function addAlternative() {
    const numbers = Array.from(arguments);
    
    function inner() {
        if (arguments.length === 0) {
            return numbers.reduce((a, b) => a + b, 0);
        }
        numbers.push(...arguments);
        return inner;
    }
    
    return inner;
}

// Using toString trick
function addWithToString() {
    const args = Array.from(arguments);
    
    function inner() {
        args.push(...arguments);
        return inner;
    }
    
    inner.toString = () => args.reduce((a, b) => a + b, 0);
    inner.valueOf = () => args.reduce((a, b) => a + b, 0);
    
    return inner;
}

console.log(+addWithToString(1)(2)(3)); // 6
```
</details>

---

### Currying Question 3: Real-World Use Case
**Question:** Create a curried function for API calls:
```javascript
// Create a curried fetch function that allows:
// const getUser = apiCall('GET')('/api/users');
// const postUser = apiCall('POST')('/api/users');
// 
// getUser({ id: 123 }).then(console.log);
// postUser({ name: 'John', email: 'john@example.com' }).then(console.log);
```

<details>
<summary>Solution</summary>

```javascript
function apiCall(method) {
    return function(url) {
        return function(data = {}) {
            const config = {
                method: method,
                headers: {
                    'Content-Type': 'application/json',
                }
            };
            
            if (method === 'GET') {
                // Add query parameters for GET
                const params = new URLSearchParams(data);
                const fullUrl = `${url}?${params}`;
                return fetch(fullUrl, config);
            } else {
                // Add body for other methods
                config.body = JSON.stringify(data);
                return fetch(url, config);
            }
        };
    };
}

// Usage
const getUser = apiCall('GET')('/api/users');
const postUser = apiCall('POST')('/api/users');
const putUser = apiCall('PUT')('/api/users');
const deleteUser = apiCall('DELETE')('/api/users');

// More advanced version with default headers
function createApiClient(baseURL, defaultHeaders = {}) {
    return function(method) {
        return function(endpoint) {
            return function(data = {}, customHeaders = {}) {
                const url = `${baseURL}${endpoint}`;
                const headers = { ...defaultHeaders, ...customHeaders };
                
                const config = { method, headers };
                
                if (method === 'GET' || method === 'DELETE') {
                    const params = new URLSearchParams(data);
                    return fetch(`${url}?${params}`, config);
                } else {
                    config.body = JSON.stringify(data);
                    return fetch(url, config);
                }
            };
        };
    };
}

// Usage
const api = createApiClient('https://api.example.com', {
    'Authorization': 'Bearer token123'
});

const getUsers = api('GET')('/users');
const createUser = api('POST')('/users');
```
</details>

---

### Currying Question 4: Partial Application vs Currying
**Question:** Explain the difference and implement both:
```javascript
// Partial Application
function partial(fn, ...args1) {
    // Your implementation
}

// Currying
function curry(fn) {
    // Your implementation
}

// Test function
function multiply(a, b, c, d) {
    return a * b * c * d;
}

// Should work differently:
const partialMultiply = partial(multiply, 2, 3);
console.log(partialMultiply(4, 5)); // 120

const curriedMultiply = curry(multiply);
console.log(curriedMultiply(2)(3)(4)(5)); // 120
```

<details>
<summary>Solution</summary>

```javascript
// Partial Application - fixes some arguments
function partial(fn, ...args1) {
    return function(...args2) {
        return fn(...args1, ...args2);
    };
}

// Currying - transforms function to sequence of single-argument functions
function curry(fn) {
    return function curried(...args) {
        if (args.length >= fn.length) {
            return fn.apply(this, args);
        } else {
            return function(...nextArgs) {
                return curried(...args, ...nextArgs);
            };
        }
    };
}

// Demonstration of the difference:
function add(a, b, c) {
    return a + b + c;
}

// Partial - pre-fill some arguments
const add5and3 = partial(add, 5, 3);
console.log(add5and3(2)); // 10 - still need to call with remaining args

// Curry - transform to chain of single-argument functions
const curriedAdd = curry(add);
console.log(curriedAdd(5)(3)(2)); // 10 - must call with one arg at a time

// But curry is more flexible:
console.log(curriedAdd(5, 3)(2)); // 10 - also works
console.log(curriedAdd(5)(3, 2)); // 10 - also works
```
</details>

---

## 🔧 Polyfill Challenges

### Polyfill 1: Array.prototype.reduce()
**Question:** Implement a polyfill for `Array.prototype.reduce()`:
```javascript
Array.prototype.myReduce = function(callback, initialValue) {
    // Your implementation
};

// Test cases
const numbers = [1, 2, 3, 4];
console.log(numbers.myReduce((acc, curr) => acc + curr, 0)); // 10
console.log(numbers.myReduce((acc, curr) => acc + curr)); // 10
console.log([].myReduce((acc, curr) => acc + curr, 5)); // 5
```

<details>
<summary>Solution</summary>

```javascript
Array.prototype.myReduce = function(callback, initialValue) {
    if (this == null) {
        throw new TypeError('Array.prototype.myReduce called on null or undefined');
    }
    
    if (typeof callback !== 'function') {
        throw new TypeError(callback + ' is not a function');
    }
    
    const O = Object(this);
    const len = parseInt(O.length) || 0;
    
    let k = 0;
    let accumulator;
    
    if (arguments.length >= 2) {
        accumulator = initialValue;
    } else {
        if (len === 0) {
            throw new TypeError('Reduce of empty array with no initial value');
        }
        
        // Find first valid element
        let kPresent = false;
        while (k < len && !kPresent) {
            if (k in O) {
                accumulator = O[k];
                kPresent = true;
            }
            k++;
        }
        
        if (!kPresent) {
            throw new TypeError('Reduce of empty array with no initial value');
        }
    }
    
    while (k < len) {
        if (k in O) {
            accumulator = callback(accumulator, O[k], k, O);
        }
        k++;
    }
    
    return accumulator;
};
```
</details>

---

### Polyfill 2: Function.prototype.bind()
**Question:** Implement `Function.prototype.bind()`:
```javascript
Function.prototype.myBind = function(thisArg, ...args) {
    // Your implementation
};

// Test cases
const obj = { name: 'John' };
function greet(greeting, punctuation) {
    return `${greeting}, ${this.name}${punctuation}`;
}

const boundGreet = greet.myBind(obj, 'Hello');
console.log(boundGreet('!')); // "Hello, John!"
```

<details>
<summary>Solution</summary>

```javascript
Function.prototype.myBind = function(thisArg, ...args) {
    if (typeof this !== 'function') {
        throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
    }
    
    const originalFunction = this;
    const noop = function() {};
    
    const boundFunction = function(...newArgs) {
        // If called with 'new', use new target as 'this'
        if (new.target) {
            return new originalFunction(...args, ...newArgs);
        }
        
        return originalFunction.apply(thisArg, [...args, ...newArgs]);
    };
    
    // Maintain prototype chain for constructor functions
    if (originalFunction.prototype) {
        noop.prototype = originalFunction.prototype;
        boundFunction.prototype = new noop();
    }
    
    return boundFunction;
};

// Advanced version with proper new.target handling
Function.prototype.myBindAdvanced = function(thisArg, ...args) {
    if (typeof this !== 'function') {
        throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
    }
    
    const originalFunction = this;
    
    function boundFunction(...newArgs) {
        if (this instanceof boundFunction) {
            // Called with 'new'
            return new originalFunction(...args, ...newArgs);
        } else {
            // Called normally
            return originalFunction.apply(thisArg, [...args, ...newArgs]);
        }
    }
    
    // Copy prototype
    boundFunction.prototype = Object.create(originalFunction.prototype);
    
    return boundFunction;
};
```
</details>

---

### Polyfill 3: Promise.allSettled()
**Question:** Implement `Promise.allSettled()`:
```javascript
Promise.myAllSettled = function(promises) {
    // Your implementation
};

// Test case
const promises = [
    Promise.resolve(1),
    Promise.reject('error'),
    Promise.resolve(3)
];

Promise.myAllSettled(promises).then(console.log);
// Should output:
// [
//   { status: 'fulfilled', value: 1 },
//   { status: 'rejected', reason: 'error' },
//   { status: 'fulfilled', value: 3 }
// ]
```

<details>
<summary>Solution</summary>

```javascript
Promise.myAllSettled = function(promises) {
    if (!Array.isArray(promises)) {
        return Promise.reject(new TypeError('Promise.allSettled expects an array'));
    }
    
    return new Promise((resolve) => {
        if (promises.length === 0) {
            resolve([]);
            return;
        }
        
        const results = new Array(promises.length);
        let completedCount = 0;
        
        promises.forEach((promise, index) => {
            Promise.resolve(promise)
                .then(
                    value => {
                        results[index] = { status: 'fulfilled', value };
                    },
                    reason => {
                        results[index] = { status: 'rejected', reason };
                    }
                )
                .finally(() => {
                    completedCount++;
                    if (completedCount === promises.length) {
                        resolve(results);
                    }
                });
        });
    });
};

// Alternative implementation without using finally
Promise.myAllSettledAlt = function(promises) {
    return Promise.all(
        promises.map(promise =>
            Promise.resolve(promise)
                .then(
                    value => ({ status: 'fulfilled', value }),
                    reason => ({ status: 'rejected', reason })
                )
        )
    );
};
```
</details>

---

### Polyfill 4: Array.prototype.flat()
**Question:** Implement `Array.prototype.flat()` with depth support:
```javascript
Array.prototype.myFlat = function(depth = 1) {
    // Your implementation
};

// Test cases
console.log([1, [2, [3, [4]]]].myFlat()); // [1, 2, [3, [4]]]
console.log([1, [2, [3, [4]]]].myFlat(2)); // [1, 2, 3, [4]]
console.log([1, [2, [3, [4]]]].myFlat(Infinity)); // [1, 2, 3, 4]
```

<details>
<summary>Solution</summary>

```javascript
Array.prototype.myFlat = function(depth = 1) {
    if (this == null) {
        throw new TypeError('Array.prototype.myFlat called on null or undefined');
    }
    
    const O = Object(this);
    const len = parseInt(O.length) || 0;
    
    function flattenHelper(arr, currentDepth) {
        const result = [];
        
        for (let i = 0; i < arr.length; i++) {
            if (i in arr) {
                const element = arr[i];
                
                if (Array.isArray(element) && currentDepth > 0) {
                    result.push(...flattenHelper(element, currentDepth - 1));
                } else {
                    result.push(element);
                }
            }
        }
        
        return result;
    }
    
    return flattenHelper(O, depth);
};

// Iterative version for better performance with deep arrays
Array.prototype.myFlatIterative = function(depth = 1) {
    if (this == null) {
        throw new TypeError('Array.prototype.myFlat called on null or undefined');
    }
    
    let stack = [...this.map(item => [item, depth])];
    let result = [];
    
    while (stack.length > 0) {
        let [item, currentDepth] = stack.pop();
        
        if (Array.isArray(item) && currentDepth > 0) {
            stack.push(...item.map(subItem => [subItem, currentDepth - 1]));
        } else {
            result.push(item);
        }
    }
    
    return result.reverse();
};
```
</details>

---

### Polyfill 5: Object.create()
**Question:** Implement `Object.create()`:
```javascript
Object.myCreate = function(proto, propertiesObject) {
    // Your implementation
};

// Test cases
const animal = { type: 'Animal' };
const dog = Object.myCreate(animal, {
    breed: {
        value: 'Labrador',
        writable: true,
        enumerable: true,
        configurable: true
    }
});

console.log(dog.type); // 'Animal'
console.log(dog.breed); // 'Labrador'
```

<details>
<summary>Solution</summary>

```javascript
Object.myCreate = function(proto, propertiesObject) {
    if (typeof proto !== 'object' && typeof proto !== 'function') {
        if (proto !== null) {
            throw new TypeError('Object prototype may only be an Object or null');
        }
    }
    
    // Create a temporary constructor
    function TempConstructor() {}
    TempConstructor.prototype = proto;
    
    // Create new object
    const obj = new TempConstructor();
    
    // Handle properties if provided
    if (propertiesObject !== undefined) {
        Object.defineProperties(obj, propertiesObject);
    }
    
    return obj;
};

// Modern implementation using __proto__
Object.myCreateModern = function(proto, propertiesObject) {
    if (typeof proto !== 'object' && typeof proto !== 'function') {
        if (proto !== null) {
            throw new TypeError('Object prototype may only be an Object or null');
        }
    }
    
    const obj = {};
    obj.__proto__ = proto;
    
    if (propertiesObject !== undefined) {
        Object.defineProperties(obj, propertiesObject);
    }
    
    return obj;
};

// Most accurate polyfill
if (!Object.create) {
    Object.create = (function() {
        // Shared empty constructor
        function Temp() {}
        
        return function(proto, propertiesObject) {
            if (typeof proto !== 'object' && typeof proto !== 'function') {
                if (proto !== null) {
                    throw new TypeError('Object prototype may only be an Object or null: ' + proto);
                }
            }
            
            Temp.prototype = proto;
            const result = new Temp();
            Temp.prototype = null; // Clean up
            
            if (propertiesObject !== undefined) {
                Object.defineProperties(result, propertiesObject);
            }
            
            // Handle null prototype case
            if (proto === null) {
                result.__proto__ = null;
            }
            
            return result;
        };
    })();
}
```
</details>

---

## 🎯 Advanced Challenge: Combo Questions

### Challenge 1: Curried Memoization
**Question:** Create a curried function that also implements memoization:
```javascript
function memoizedCurry(fn) {
    // Should curry the function AND cache results
    // const add = (a, b, c) => a + b + c;
    // const curriedAdd = memoizedCurry(add);
    // curriedAdd(1)(2)(3); // Computed
    // curriedAdd(1)(2)(3); // From cache
}
```

### Challenge 2: Closure-based Private Methods
**Question:** Create a class-like structure using only closures that has private methods and properties:
```javascript
function createUser(name, email) {
    // Should have private methods and properties
    // Public methods: getName, getEmail, updateEmail
    // Private methods: validateEmail, log
    // Private properties: _name, _email, _logs
}
```

### Challenge 3: Polyfill with Currying
**Question:** Create a polyfill for `Array.prototype.filter()` that can be curried:
```javascript
// Should work both ways:
// arr.myFilter(predicate)
// myFilter(predicate)(arr)
```

This comprehensive collection covers the most challenging and frequently asked questions about closures, currying, and polyfills in JavaScript interviews!