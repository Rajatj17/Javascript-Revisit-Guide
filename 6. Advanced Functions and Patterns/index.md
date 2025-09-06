# Module 6: Advanced Functions & Patterns

## Why This Module Matters
JavaScript is a functional programming language at its core. Understanding higher-order functions, functional patterns, and advanced function techniques is crucial for writing clean, maintainable code and understanding modern frameworks (React hooks, Redux, RxJS). This builds on closures (Module 3) and async patterns (Module 5).

## Core Concept: Functions as First-Class Citizens

In JavaScript, functions are values that can be:
- Assigned to variables
- Passed as arguments
- Returned from other functions
- Stored in data structures

```javascript
// Functions are values
var greet = function(name) {
    return `Hello, ${name}!`;
};

// Functions can be passed around
var operations = {
    add: (a, b) => a + b,
    multiply: (a, b) => a * b,
    transform: greet
};

// Functions can be returned
function createMultiplier(factor) {
    return function(number) {
        return number * factor;
    };
}

var double = createMultiplier(2);
console.log(double(5)); // 10
```

## Higher-Order Functions

### Functions that Accept Functions
```javascript
// Array methods are higher-order functions
var numbers = [1, 2, 3, 4, 5];

var doubled = numbers.map(x => x * 2);
var evens = numbers.filter(x => x % 2 === 0);
var sum = numbers.reduce((acc, x) => acc + x, 0);

// Custom higher-order function
function withLogging(fn) {
    return function(...args) {
        console.log(`Calling ${fn.name} with:`, args);
        var result = fn.apply(this, args);
        console.log(`Result:`, result);
        return result;
    };
}

function add(a, b) {
    return a + b;
}

var loggedAdd = withLogging(add);
loggedAdd(2, 3); // Logs: "Calling add with: [2, 3]" then "Result: 5"
```

### Functions that Return Functions
```javascript
// Function factory pattern
function createValidator(rules) {
    return function(data) {
        var errors = [];
        
        for (var rule of rules) {
            if (!rule.test(data)) {
                errors.push(rule.message);
            }
        }
        
        return {
            isValid: errors.length === 0,
            errors: errors
        };
    };
}

var userValidator = createValidator([
    {
        test: data => data.name && data.name.length > 0,
        message: "Name is required"
    },
    {
        test: data => data.email && data.email.includes('@'),
        message: "Valid email is required"
    },
    {
        test: data => data.age && data.age >= 18,
        message: "Must be 18 or older"
    }
]);

var result = userValidator({ name: "John", email: "john@example.com", age: 25 });
console.log(result); // { isValid: true, errors: [] }
```

## Functional Programming Patterns

### Pure Functions
```javascript
// Pure function - same input always produces same output, no side effects
function pure(a, b) {
    return a + b; // No external state modified, no external dependencies
}

// Impure function - depends on external state
var counter = 0;
function impure(a) {
    counter++; // Side effect - modifies external state
    return a + counter;
}

// Converting impure to pure
function pureCounter(a, currentCount) {
    return {
        result: a + currentCount + 1,
        newCount: currentCount + 1
    };
}

// Usage of pure version
var state = { count: 0 };
var { result, newCount } = pureCounter(5, state.count);
state = { count: newCount }; // Explicit state update
```

### Immutability Patterns
```javascript
// Array operations without mutation
var originalArray = [1, 2, 3, 4, 5];

// Adding elements
var withNewElement = [...originalArray, 6]; // [1, 2, 3, 4, 5, 6]
var withPrepended = [0, ...originalArray]; // [0, 1, 2, 3, 4, 5]

// Removing elements
var withoutFirst = originalArray.slice(1); // [2, 3, 4, 5]
var withoutLast = originalArray.slice(0, -1); // [1, 2, 3, 4]
var withoutIndex = originalArray.filter((_, i) => i !== 2); // [1, 2, 4, 5]

// Updating elements
var doubled = originalArray.map(x => x * 2); // [2, 4, 6, 8, 10]
var updatedAtIndex = originalArray.map((x, i) => i === 2 ? x * 10 : x); // [1, 2, 30, 4, 5]

// Object operations without mutation
var originalObject = {
    name: "John",
    age: 30,
    address: {
        city: "New York",
        country: "USA"
    }
};

// Shallow updates
var updated = { ...originalObject, age: 31 };
var withNewProperty = { ...originalObject, email: "john@example.com" };

// Deep updates (nested objects)
var deepUpdated = {
    ...originalObject,
    address: {
        ...originalObject.address,
        city: "Boston"
    }
};

// Using helper functions for deep updates
function updateIn(obj, path, value) {
    var [head, ...tail] = path;
    
    if (tail.length === 0) {
        return { ...obj, [head]: value };
    }
    
    return {
        ...obj,
        [head]: updateIn(obj[head], tail, value)
    };
}

var deepUpdated2 = updateIn(originalObject, ['address', 'city'], 'Chicago');
```

### Composition and Piping
```javascript
// Function composition - combining simple functions to build complex ones
function compose(...functions) {
    return function(value) {
        return functions.reduceRight((acc, fn) => fn(acc), value);
    };
}

// Individual functions
var add1 = x => x + 1;
var multiply2 = x => x * 2;
var square = x => x * x;

// Composed function
var transform = compose(square, multiply2, add1);
console.log(transform(3)); // ((3 + 1) * 2)² = 64

// Pipe - left-to-right composition (more readable)
function pipe(...functions) {
    return function(value) {
        return functions.reduce((acc, fn) => fn(acc), value);
    };
}

var pipeline = pipe(add1, multiply2, square);
console.log(pipeline(3)); // ((3 + 1) * 2)² = 64

// Real-world example: data processing pipeline
var processUserData = pipe(
    data => ({ ...data, name: data.name.trim() }),
    data => ({ ...data, email: data.email.toLowerCase() }),
    data => ({ ...data, age: parseInt(data.age) }),
    data => ({ ...data, isAdult: data.age >= 18 })
);

var userData = {
    name: "  John Doe  ",
    email: "JOHN@EXAMPLE.COM",
    age: "25"
};

console.log(processUserData(userData));
// { name: "John Doe", email: "john@example.com", age: 25, isAdult: true }
```

## Currying and Partial Application

### Currying Implementation
```javascript
// Manual currying
function curryAdd(a) {
    return function(b) {
        return function(c) {
            return a + b + c;
        };
    };
}

console.log(curryAdd(1)(2)(3)); // 6

// Generic curry function
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

// Usage
function multiply(a, b, c) {
    return a * b * c;
}

var curriedMultiply = curry(multiply);

// All these work
console.log(curriedMultiply(2)(3)(4)); // 24
console.log(curriedMultiply(2, 3)(4)); // 24
console.log(curriedMultiply(2)(3, 4)); // 24
console.log(curriedMultiply(2, 3, 4)); // 24

// Partial application
var multiplyBy2 = curriedMultiply(2);
var multiplyBy2And3 = multiplyBy2(3);
console.log(multiplyBy2And3(4)); // 24
```

### Practical Currying Examples
```javascript
// Event handler creation
var addEventListener = curry((event, handler, element) => {
    element.addEventListener(event, handler);
});

var addClickListener = addEventListener('click');
var addChangeListener = addEventListener('change');

// Usage
addClickListener(handleButtonClick, button);
addChangeListener(handleInputChange, input);

// API request builder
var makeRequest = curry((method, url, options, data) => {
    return fetch(url, {
        method,
        ...options,
        body: data ? JSON.stringify(data) : undefined
    });
});

var get = makeRequest('GET');
var post = makeRequest('POST');
var apiGet = get('/api');
var apiPost = post('/api');

// Usage
apiGet('/users')({ headers: { 'Authorization': 'Bearer token' } });
apiPost('/users')({ headers: { 'Content-Type': 'application/json' } }, userData);
```

## Memoization

### Basic Memoization
```javascript
function memoize(fn) {
    var cache = new Map();
    
    return function(...args) {
        var key = JSON.stringify(args);
        
        if (cache.has(key)) {
            console.log('Cache hit');
            return cache.get(key);
        }
        
        console.log('Computing...');
        var result = fn.apply(this, args);
        cache.set(key, result);
        return result;
    };
}

// Expensive recursive function
function fibonacci(n) {
    if (n < 2) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}

var memoizedFib = memoize(fibonacci);

console.time('First call');
console.log(memoizedFib(40)); // Slow first time
console.timeEnd('First call');

console.time('Second call');
console.log(memoizedFib(40)); // Instant second time
console.timeEnd('Second call');
```

### Advanced Memoization with TTL
```javascript
function memoizeWithTTL(fn, ttlMs = 60000) {
    var cache = new Map();
    
    return function(...args) {
        var key = JSON.stringify(args);
        var cached = cache.get(key);
        
        if (cached && Date.now() - cached.timestamp < ttlMs) {
            console.log('Cache hit (within TTL)');
            return cached.value;
        }
        
        console.log('Computing/Cache expired...');
        var result = fn.apply(this, args);
        cache.set(key, {
            value: result,
            timestamp: Date.now()
        });
        
        return result;
    };
}

// API call with 5-second cache
var fetchUser = memoizeWithTTL(async (userId) => {
    var response = await fetch(`/api/users/${userId}`);
    return response.json();
}, 5000);
```

### Memoization with Size Limit (LRU)
```javascript
function memoizeWithLRU(fn, maxSize = 100) {
    var cache = new Map();
    
    return function(...args) {
        var key = JSON.stringify(args);
        
        if (cache.has(key)) {
            // Move to end (most recently used)
            var value = cache.get(key);
            cache.delete(key);
            cache.set(key, value);
            return value;
        }
        
        var result = fn.apply(this, args);
        
        // Remove oldest if at capacity
        if (cache.size >= maxSize) {
            var firstKey = cache.keys().next().value;
            cache.delete(firstKey);
        }
        
        cache.set(key, result);
        return result;
    };
}
```

## Debouncing and Throttling

### Debouncing Implementation
```javascript
function debounce(func, delay) {
    var timeoutId;
    
    return function(...args) {
        var context = this;
        
        clearTimeout(timeoutId);
        
        timeoutId = setTimeout(() => {
            func.apply(context, args);
        }, delay);
    };
}

// Usage: Search as user types
var searchInput = document.getElementById('search');
var performSearch = debounce((query) => {
    console.log('Searching for:', query);
    // Make API call
}, 300);

searchInput.addEventListener('input', (e) => {
    performSearch(e.target.value);
});

// Advanced debounce with immediate option
function advancedDebounce(func, delay, immediate = false) {
    var timeoutId;
    
    return function(...args) {
        var context = this;
        var callNow = immediate && !timeoutId;
        
        clearTimeout(timeoutId);
        
        timeoutId = setTimeout(() => {
            timeoutId = null;
            if (!immediate) func.apply(context, args);
        }, delay);
        
        if (callNow) func.apply(context, args);
    };
}
```

### Throttling Implementation
```javascript
function throttle(func, limit) {
    var inThrottle;
    
    return function(...args) {
        var context = this;
        
        if (!inThrottle) {
            func.apply(context, args);
            inThrottle = true;
            
            setTimeout(() => {
                inThrottle = false;
            }, limit);
        }
    };
}

// Usage: Scroll event handling
var handleScroll = throttle(() => {
    console.log('Scroll position:', window.scrollY);
    // Update scroll-based UI
}, 100);

window.addEventListener('scroll', handleScroll);

// Advanced throttle with trailing option
function advancedThrottle(func, limit, options = {}) {
    var timeout;
    var previous = 0;
    var result;
    
    var later = function(context, args) {
        previous = options.leading === false ? 0 : Date.now();
        timeout = null;
        result = func.apply(context, args);
    };
    
    return function(...args) {
        var now = Date.now();
        
        if (!previous && options.leading === false) previous = now;
        
        var remaining = limit - (now - previous);
        var context = this;
        
        if (remaining <= 0 || remaining > limit) {
            if (timeout) {
                clearTimeout(timeout);
                timeout = null;
            }
            previous = now;
            result = func.apply(context, args);
        } else if (!timeout && options.trailing !== false) {
            timeout = setTimeout(() => later(context, args), remaining);
        }
        
        return result;
    };
}
```

## Advanced Function Patterns

### Function Decorators
```javascript
// Timing decorator
function timed(target, propertyKey, descriptor) {
    var originalMethod = descriptor.value;
    
    descriptor.value = function(...args) {
        console.time(`${propertyKey} execution time`);
        var result = originalMethod.apply(this, args);
        console.timeEnd(`${propertyKey} execution time`);
        return result;
    };
    
    return descriptor;
}

// Retry decorator
function retry(attempts = 3) {
    return function(target, propertyKey, descriptor) {
        var originalMethod = descriptor.value;
        
        descriptor.value = async function(...args) {
            for (let i = 0; i < attempts; i++) {
                try {
                    return await originalMethod.apply(this, args);
                } catch (error) {
                    if (i === attempts - 1) throw error;
                    console.warn(`Attempt ${i + 1} failed, retrying...`);
                }
            }
        };
        
        return descriptor;
    };
}

// Usage with classes
class ApiService {
    @timed
    @retry(3)
    async fetchData(url) {
        var response = await fetch(url);
        if (!response.ok) throw new Error('API call failed');
        return response.json();
    }
}
```

### Function Factories
```javascript
// State machine factory
function createStateMachine(states, initialState) {
    var currentState = initialState;
    var listeners = {};
    
    return {
        getState() {
            return currentState;
        },
        
        transition(event) {
            var state = states[currentState];
            var nextState = state && state[event];
            
            if (nextState) {
                var previousState = currentState;
                currentState = nextState;
                
                // Trigger listeners
                if (listeners[nextState]) {
                    listeners[nextState].forEach(callback => 
                        callback(previousState, nextState, event)
                    );
                }
                
                return true;
            }
            
            return false;
        },
        
        on(state, callback) {
            if (!listeners[state]) listeners[state] = [];
            listeners[state].push(callback);
        }
    };
}

// Usage
var loginStateMachine = createStateMachine({
    'logged_out': { 'login': 'logging_in' },
    'logging_in': { 'success': 'logged_in', 'failure': 'logged_out' },
    'logged_in': { 'logout': 'logged_out' }
}, 'logged_out');

loginStateMachine.on('logged_in', (prev, curr, event) => {
    console.log('User successfully logged in');
});

loginStateMachine.transition('login');
loginStateMachine.transition('success');
```

### Observer Pattern with Functions
```javascript
function createObservable() {
    var observers = [];
    
    return {
        subscribe(observer) {
            observers.push(observer);
            
            // Return unsubscribe function
            return () => {
                var index = observers.indexOf(observer);
                if (index > -1) observers.splice(index, 1);
            };
        },
        
        notify(data) {
            observers.forEach(observer => observer(data));
        },
        
        map(transform) {
            var mapped = createObservable();
            
            this.subscribe(data => {
                mapped.notify(transform(data));
            });
            
            return mapped;
        },
        
        filter(predicate) {
            var filtered = createObservable();
            
            this.subscribe(data => {
                if (predicate(data)) {
                    filtered.notify(data);
                }
            });
            
            return filtered;
        }
    };
}

// Usage
var dataStream = createObservable();

var processedStream = dataStream
    .filter(x => x > 0)
    .map(x => x * 2);

var unsubscribe = processedStream.subscribe(data => {
    console.log('Processed data:', data);
});

dataStream.notify(5); // Output: "Processed data: 10"
dataStream.notify(-3); // No output (filtered out)
dataStream.notify(7); // Output: "Processed data: 14"

unsubscribe(); // Stop receiving updates
```

## Performance Optimization Patterns

### Lazy Evaluation
```javascript
function createLazySequence(generator) {
    return {
        map(transform) {
            return createLazySequence(function*() {
                for (var item of generator()) {
                    yield transform(item);
                }
            });
        },
        
        filter(predicate) {
            return createLazySequence(function*() {
                for (var item of generator()) {
                    if (predicate(item)) {
                        yield item;
                    }
                }
            });
        },
        
        take(count) {
            return createLazySequence(function*() {
                var taken = 0;
                for (var item of generator()) {
                    if (taken >= count) break;
                    yield item;
                    taken++;
                }
            });
        },
        
        toArray() {
            return Array.from(generator());
        }
    };
}

// Usage - only computes what's needed
var numbers = createLazySequence(function*() {
    for (let i = 1; i <= 1000000; i++) {
        console.log(`Generating ${i}`); // Only logs first 5
        yield i;
    }
});

var result = numbers
    .filter(x => x % 2 === 0)
    .map(x => x * x)
    .take(5)
    .toArray();

console.log(result); // [4, 16, 36, 64, 100]
```

### Function Pooling
```javascript
function createFunctionPool(factory, poolSize = 10) {
    var pool = [];
    var active = new Set();
    
    function createFunction() {
        var fn = factory();
        fn._poolId = Symbol('poolId');
        return fn;
    }
    
    // Pre-populate pool
    for (let i = 0; i < poolSize; i++) {
        pool.push(createFunction());
    }
    
    return {
        acquire() {
            var fn = pool.length > 0 ? pool.pop() : createFunction();
            active.add(fn._poolId);
            return fn;
        },
        
        release(fn) {
            if (active.has(fn._poolId)) {
                active.delete(fn._poolId);
                if (pool.length < poolSize) {
                    pool.push(fn);
                }
            }
        },
        
        getStats() {
            return {
                poolSize: pool.length,
                activeCount: active.size,
                totalCreated: pool.length + active.size
            };
        }
    };
}

// Usage for expensive function creation
var expensiveFunctionPool = createFunctionPool(() => {
    console.log('Creating expensive function');
    return function(data) {
        // Expensive computation
        return data.map(x => Math.sqrt(x * x + 1));
    };
}, 3);
```

## Interview Questions & Challenges

### Question 1: Implement reduce()
```javascript
// Implement Array.prototype.reduce from scratch
function myReduce(array, callback, initialValue) {
    // Your implementation here
    // Should handle:
    // - With initial value
    // - Without initial value
    // - Empty arrays
    // - Callback with 4 parameters (accumulator, current, index, array)
}

// Test cases
var numbers = [1, 2, 3, 4, 5];
console.log(myReduce(numbers, (a, b) => a + b, 0)); // 15
console.log(myReduce(numbers, (a, b) => a + b)); // 15
```

### Question 2: Deep Function Composition
```javascript
// Create a compose function that handles async functions
function asyncCompose(...functions) {
    // Your implementation here
    // Should work with both sync and async functions
}

var addOne = x => x + 1;
var double = x => x * 2;
var asyncSquare = async x => x * x;

var pipeline = asyncCompose(asyncSquare, double, addOne);
pipeline(3).then(console.log); // Should output 64
```

### Question 3: Smart Memoization
```javascript
// Create a memoization function that can handle object arguments
function smartMemoize(fn) {
    // Your implementation here
    // Should handle:
    // - Primitive arguments
    // - Object arguments (deep comparison)
    // - Circular references
    // - Cache size limits
}

function expensiveFunction(obj) {
    console.log('Computing...');
    return obj.a + obj.b;
}

var memoized = smartMemoize(expensiveFunction);
memoized({ a: 1, b: 2 }); // Computes
memoized({ a: 1, b: 2 }); // Cache hit
```

### Question 4: Function Pipeline Builder
```javascript
// Create a function that builds processing pipelines
function createPipeline() {
    // Your implementation here
    // Should support:
    // - Adding sync/async operations
    // - Error handling
    // - Conditional branches
    // - Parallel operations
}

var pipeline = createPipeline()
    .add(x => x + 1)
    .add(async x => x * 2)
    .branch(x => x > 10, 
        pipe => pipe.add(x => x - 5),
        pipe => pipe.add(x => x + 5)
    )
    .parallel([
        x => x * 2,
        x => x + 10
    ])
    .add(([doubled, added]) => doubled + added);

pipeline.execute(5).then(console.log);
```

### Question 5: Rate Limiter with Queuing
```javascript
// Create a rate limiter that queues function calls
function createRateLimiter(maxCalls, timeWindow) {
    // Your implementation here
    // Should:
    // - Limit calls per time window
    // - Queue excess calls
    // - Return promises for queued calls
    // - Support different priorities
}

var limiter = createRateLimiter(5, 1000); // 5 calls per second

var limitedFunction = limiter.wrap(expensiveApiCall);
limitedFunction(data1); // Executes immediately
limitedFunction(data2); // Executes immediately
// ... 5 more calls get queued
```

## Key Takeaways for Interviews

1. **Functions are first-class citizens** - can be stored, passed, returned
2. **Higher-order functions** accept or return other functions
3. **Closures enable** currying, memoization, and function factories
4. **Pure functions** are predictable and testable
5. **Immutability** prevents bugs and enables optimization
6. **Function composition** builds complex behavior from simple parts
7. **Memoization trades memory for speed** - cache expensive computations
8. **Debouncing delays execution**, throttling limits frequency
9. **Lazy evaluation** defers computation until needed
10. **Functional patterns** lead to more maintainable code

## Practice Challenges

### Challenge 1: Functional State Management
```javascript
// Create a functional state management system
function createStore(reducer, initialState) {
    // Your implementation
    // Should support: getState(), dispatch(action), subscribe(listener)
    // All without mutating state
}

function counterReducer(state = { count: 0 }, action) {
    switch (action.type) {
        case 'INCREMENT':
            return { count: state.count + 1 };
        case 'DECREMENT':
            return { count: state.count - 1 };
        default:
            return state;
    }
}

var store = createStore(counterReducer);
```

### Challenge 2: Async Function Queue
```javascript
// Create a queue that processes async functions sequentially
class AsyncQueue {
    constructor(concurrency = 1) {
        // Your implementation
        // Should process functions in order with limited concurrency
    }
    
    add(asyncFunction) {
        // Return promise that resolves when function completes
    }
}

var queue = new AsyncQueue(2);
queue.add(() => delay(1000).then(() => console.log('Task 1')));
queue.add(() => delay(500).then(() => console.log('Task 2')));
```

### Challenge 3: Function Dependency Injection
```javascript
// Create a dependency injection system using functions
function createContainer() {
    // Your implementation
    // Should support: register(name, factory), resolve(name)
    // Handle dependencies between services
}

var container = createContainer();
container.register('logger', () => console.log);
container.register('apiService', (logger) => new ApiService(logger));
var api = container.resolve('apiService');
```
