# Module 5: Asynchronous JavaScript

## Why This Module Matters
JavaScript is single-threaded but needs to handle I/O operations (network requests, file reads, timers) without blocking. Understanding async patterns is crucial for modern web development, Node.js, and handling user interactions. This builds on execution context (Module 1) since async operations create new execution contexts.

## Core Concept: JavaScript's Single-Threaded Nature

JavaScript has **one call stack** but can handle async operations through the **event loop** and **callback queues**.

```javascript
console.log("1"); // Synchronous

setTimeout(() => {
    console.log("2"); // Asynchronous - goes to task queue
}, 0);

Promise.resolve().then(() => {
    console.log("3"); // Asynchronous - goes to microtask queue
});

console.log("4"); // Synchronous

// Output: 1, 4, 3, 2
// Synchronous code runs first, then microtasks, then macrotasks
```

## The Event Loop Deep Dive

### Call Stack, Queues, and Event Loop
```javascript
function first() {
    console.log("First");
    second();
}

function second() {
    console.log("Second");
    setTimeout(() => {
        console.log("Timeout");
    }, 0);
    
    Promise.resolve().then(() => {
        console.log("Promise");
    });
    
    console.log("End of second");
}

first();
console.log("Main thread end");

// Execution order:
// 1. Call Stack: first() → second() → console.logs
// 2. Async operations queued
// 3. Call stack empties
// 4. Event loop processes microtasks first (Promise)
// 5. Then macrotasks (setTimeout)

// Output: First, Second, End of second, Main thread end, Promise, Timeout
```

### Microtasks vs Macrotasks
```javascript
// Macrotasks (Task Queue)
setTimeout(() => console.log("setTimeout 1"), 0);
setInterval(() => console.log("setInterval"), 0);
setImmediate(() => console.log("setImmediate")); // Node.js only

// Microtasks (Job Queue) - Higher Priority
Promise.resolve().then(() => console.log("Promise 1"));
queueMicrotask(() => console.log("queueMicrotask"));
Promise.resolve().then(() => console.log("Promise 2"));

// Microtasks always run before macrotasks
// Output: Promise 1, queueMicrotask, Promise 2, setTimeout 1, setInterval, setImmediate
```

### Event Loop Phases (Node.js)
```javascript
// Timer phase
setTimeout(() => console.log("Timer"), 0);

// I/O callbacks phase
const fs = require('fs');
fs.readFile(__filename, () => console.log("I/O"));

// Immediate phase
setImmediate(() => console.log("Immediate"));

// Close callbacks phase
const server = require('http').createServer();
server.close(() => console.log("Close"));

// Process.nextTick and Promises (microtasks) run between each phase
process.nextTick(() => console.log("NextTick"));
Promise.resolve().then(() => console.log("Promise"));
```

## Callback Patterns

### Basic Callbacks
```javascript
function fetchUserData(userId, callback) {
    // Simulate async operation
    setTimeout(() => {
        if (userId > 0) {
            callback(null, { id: userId, name: "User " + userId });
        } else {
            callback(new Error("Invalid user ID"));
        }
    }, 1000);
}

// Usage
fetchUserData(1, (error, user) => {
    if (error) {
        console.error("Error:", error.message);
    } else {
        console.log("User:", user);
    }
});
```

### Callback Hell Problem
```javascript
// Pyramid of Doom
getUserData(userId, (err, user) => {
    if (err) throw err;
    
    getOrderHistory(user.id, (err, orders) => {
        if (err) throw err;
        
        getOrderDetails(orders[0].id, (err, details) => {
            if (err) throw err;
            
            getPaymentInfo(details.paymentId, (err, payment) => {
                if (err) throw err;
                
                // Finally have all the data
                console.log("Payment info:", payment);
            });
        });
    });
});
```

### Error Handling with Callbacks
```javascript
function asyncOperation(data, callback) {
    setTimeout(() => {
        try {
            if (!data) {
                throw new Error("No data provided");
            }
            
            var result = processData(data);
            callback(null, result); // Success: error = null
        } catch (error) {
            callback(error); // Error: pass error as first argument
        }
    }, 100);
}

// Node.js convention: callback(error, result)
asyncOperation("valid data", (err, result) => {
    if (err) {
        console.error("Error:", err.message);
        return;
    }
    console.log("Result:", result);
});
```

## Promises: The Modern Approach

### Promise States and Creation
```javascript
// Promise has 3 states: pending, fulfilled, rejected
var promise = new Promise((resolve, reject) => {
    var success = Math.random() > 0.5;
    
    setTimeout(() => {
        if (success) {
            resolve("Operation successful!"); // fulfilled
        } else {
            reject(new Error("Operation failed!")); // rejected
        }
    }, 1000);
});

// Once settled (fulfilled/rejected), state never changes
promise
    .then(result => {
        console.log("Success:", result);
        return "Modified result";
    })
    .then(modifiedResult => {
        console.log("Chain:", modifiedResult);
    })
    .catch(error => {
        console.error("Error:", error.message);
    })
    .finally(() => {
        console.log("Cleanup always runs");
    });
```

### Promise Chaining
```javascript
function fetchUser(id) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            if (id > 0) {
                resolve({ id, name: `User ${id}` });
            } else {
                reject(new Error("Invalid ID"));
            }
        }, 500);
    });
}

function fetchPosts(userId) {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve([
                { id: 1, title: "Post 1", userId },
                { id: 2, title: "Post 2", userId }
            ]);
        }, 500);
    });
}

// Chaining avoids callback hell
fetchUser(1)
    .then(user => {
        console.log("Got user:", user);
        return fetchPosts(user.id); // Return promise to chain
    })
    .then(posts => {
        console.log("Got posts:", posts);
        return posts.length; // Return value becomes resolved value
    })
    .then(postCount => {
        console.log("Post count:", postCount);
    })
    .catch(error => {
        console.error("Any error in chain:", error.message);
    });
```

### Promise Static Methods
```javascript
var promise1 = Promise.resolve("Already resolved");
var promise2 = new Promise(resolve => setTimeout(() => resolve("Async resolve"), 1000));
var promise3 = fetch('/api/data'); // Returns promise

// Promise.all - All must succeed
Promise.all([promise1, promise2, promise3])
    .then(results => {
        console.log("All succeeded:", results);
        // results is array with all resolved values
    })
    .catch(error => {
        console.error("At least one failed:", error);
        // If any promise rejects, this catch runs
    });

// Promise.allSettled - Wait for all to complete (success or failure)
Promise.allSettled([promise1, promise2, promise3])
    .then(results => {
        results.forEach((result, index) => {
            if (result.status === 'fulfilled') {
                console.log(`Promise ${index} succeeded:`, result.value);
            } else {
                console.log(`Promise ${index} failed:`, result.reason);
            }
        });
    });

// Promise.race - First to complete wins
Promise.race([promise1, promise2, promise3])
    .then(result => {
        console.log("First to complete:", result);
    })
    .catch(error => {
        console.error("First to fail:", error);
    });

// Promise.any - First to succeed wins (rejects only if all fail)
Promise.any([promise1, promise2, promise3])
    .then(result => {
        console.log("First success:", result);
    })
    .catch(aggregateError => {
        console.error("All failed:", aggregateError.errors);
    });
```

### Promise Error Handling Patterns
```javascript
// Error propagation
Promise.resolve("start")
    .then(value => {
        throw new Error("Something went wrong");
        return "This won't execute";
    })
    .then(value => {
        // This won't execute due to error above
        console.log("Won't see this");
    })
    .catch(error => {
        console.error("Caught error:", error.message);
        return "Recovery value"; // Recover from error
    })
    .then(value => {
        console.log("After recovery:", value); // "Recovery value"
    });

// Multiple catch blocks
fetch('/api/data')
    .then(response => {
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
        return response.json();
    })
    .then(data => {
        if (!data.valid) {
            throw new Error("Invalid data received");
        }
        return processData(data);
    })
    .catch(error => {
        if (error.message.includes('HTTP')) {
            console.error("Network error:", error.message);
            return getFromCache(); // Fallback to cache
        } else {
            console.error("Data error:", error.message);
            return getDefaultData(); // Fallback to defaults
        }
    })
    .then(finalData => {
        console.log("Final data:", finalData);
    });
```

## Async/Await: Synchronous-Looking Async Code

### Basic Async/Await
```javascript
// Function declaration
async function fetchUserData(id) {
    try {
        var user = await fetchUser(id);
        var posts = await fetchPosts(user.id);
        var comments = await fetchComments(posts[0].id);
        
        return {
            user,
            posts,
            comments
        };
    } catch (error) {
        console.error("Error in fetchUserData:", error.message);
        throw error; // Re-throw or handle appropriately
    }
}

// Function expression
var fetchData = async (id) => {
    var result = await fetchUserData(id);
    return result;
};

// Usage
fetchUserData(1)
    .then(data => console.log("Success:", data))
    .catch(error => console.error("Failed:", error.message));

// Or with async/await
async function main() {
    try {
        var data = await fetchUserData(1);
        console.log("Success:", data);
    } catch (error) {
        console.error("Failed:", error.message);
    }
}

main();
```

### Parallel vs Sequential Execution
```javascript
// Sequential (slow) - each waits for previous
async function sequentialFetch() {
    console.time("Sequential");
    
    var user = await fetchUser(1);      // 1 second
    var posts = await fetchPosts(1);    // 1 second  
    var comments = await fetchComments(1); // 1 second
    
    console.timeEnd("Sequential"); // ~3 seconds
    return { user, posts, comments };
}

// Parallel (fast) - all start simultaneously
async function parallelFetch() {
    console.time("Parallel");
    
    var [user, posts, comments] = await Promise.all([
        fetchUser(1),
        fetchPosts(1),
        fetchComments(1)
    ]);
    
    console.timeEnd("Parallel"); // ~1 second
    return { user, posts, comments };
}

// Mixed approach - sequential dependencies, parallel independents
async function mixedFetch() {
    var user = await fetchUser(1); // Must get user first
    
    // These can run in parallel since they depend on user.id
    var [posts, profile, settings] = await Promise.all([
        fetchPosts(user.id),
        fetchProfile(user.id),
        fetchSettings(user.id)
    ]);
    
    return { user, posts, profile, settings };
}
```

### Error Handling in Async/Await
```javascript
async function robustAsyncFunction() {
    try {
        // Try primary approach
        var result = await primaryDataSource();
        return result;
    } catch (primaryError) {
        console.warn("Primary failed:", primaryError.message);
        
        try {
            // Fallback to secondary approach
            var fallbackResult = await secondaryDataSource();
            return fallbackResult;
        } catch (secondaryError) {
            console.error("Secondary also failed:", secondaryError.message);
            
            // Last resort - return cached or default data
            var cachedData = getCachedData();
            if (cachedData) {
                return cachedData;
            }
            
            // If all else fails, throw descriptive error
            throw new Error("All data sources failed. Check network connection.");
        }
    }
}

// Handling specific error types
async function specificErrorHandling() {
    try {
        var response = await fetch('/api/data');
        
        if (response.status === 401) {
            throw new Error("UNAUTHORIZED");
        }
        
        if (response.status === 404) {
            throw new Error("NOT_FOUND");
        }
        
        if (!response.ok) {
            throw new Error(`HTTP_ERROR_${response.status}`);
        }
        
        return await response.json();
    } catch (error) {
        switch (error.message) {
            case "UNAUTHORIZED":
                await refreshAuthToken();
                return specificErrorHandling(); // Retry
                
            case "NOT_FOUND":
                return getDefaultData();
                
            default:
                if (error.message.startsWith("HTTP_ERROR_")) {
                    console.error("Server error:", error.message);
                    return getCachedData();
                }
                throw error; // Re-throw unknown errors
        }
    }
}
```

## Advanced Async Patterns

### Async Iterators and for-await-of
```javascript
// Async generator
async function* asyncNumberGenerator() {
    for (let i = 0; i < 5; i++) {
        await new Promise(resolve => setTimeout(resolve, 1000));
        yield i;
    }
}

// Consuming async iterator
async function consumeAsyncIterator() {
    for await (var number of asyncNumberGenerator()) {
        console.log("Generated:", number);
    }
}

// Processing array items asynchronously
async function processItemsSequentially(items) {
    for await (var item of items) {
        var result = await processItem(item);
        console.log("Processed:", result);
    }
}

async function processItemsInParallel(items) {
    var promises = items.map(item => processItem(item));
    var results = await Promise.all(promises);
    console.log("All processed:", results);
}
```

### Async Pool (Limited Concurrency)
```javascript
async function asyncPool(poolLimit, array, iteratorFn) {
    var results = [];
    var executing = [];
    
    for (var [index, item] of array.entries()) {
        var promise = Promise.resolve().then(() => iteratorFn(item, array));
        results.push(promise);
        
        if (array.length >= poolLimit) {
            var e = promise.then(() => executing.splice(executing.indexOf(e), 1));
            executing.push(e);
            
            if (executing.length >= poolLimit) {
                await Promise.race(executing);
            }
        }
    }
    
    return Promise.all(results);
}

// Usage: Process 1000 items with max 5 concurrent operations
var items = Array.from({ length: 1000 }, (_, i) => i);

asyncPool(5, items, async (item) => {
    await delay(Math.random() * 1000);
    return item * 2;
}).then(results => {
    console.log("All items processed with limited concurrency");
});
```

### Timeout and Cancellation
```javascript
// Promise with timeout
function withTimeout(promise, timeoutMs) {
    return Promise.race([
        promise,
        new Promise((_, reject) => {
            setTimeout(() => reject(new Error("Operation timed out")), timeoutMs);
        })
    ]);
}

// Usage
async function fetchWithTimeout() {
    try {
        var data = await withTimeout(fetch('/api/slow'), 5000);
        return data;
    } catch (error) {
        if (error.message === "Operation timed out") {
            console.error("Request took too long");
            return getDefaultData();
        }
        throw error;
    }
}

// AbortController for cancellation (modern approach)
async function cancellableRequest() {
    var controller = new AbortController();
    var timeoutId = setTimeout(() => controller.abort(), 5000);
    
    try {
        var response = await fetch('/api/data', {
            signal: controller.signal
        });
        
        clearTimeout(timeoutId);
        return await response.json();
    } catch (error) {
        if (error.name === 'AbortError') {
            console.log("Request was cancelled");
        } else {
            throw error;
        }
    }
}
```

### Retry Patterns
```javascript
async function retryOperation(operation, maxRetries = 3, delay = 1000) {
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
        try {
            return await operation();
        } catch (error) {
            console.warn(`Attempt ${attempt} failed:`, error.message);
            
            if (attempt === maxRetries) {
                throw new Error(`Operation failed after ${maxRetries} attempts: ${error.message}`);
            }
            
            // Exponential backoff
            var backoffDelay = delay * Math.pow(2, attempt - 1);
            await new Promise(resolve => setTimeout(resolve, backoffDelay));
        }
    }
}

// Usage
async function unreliableApiCall() {
    if (Math.random() < 0.7) {
        throw new Error("Random failure");
    }
    return { data: "Success!" };
}

retryOperation(unreliableApiCall, 3, 500)
    .then(result => console.log("Eventually succeeded:", result))
    .catch(error => console.error("Failed after retries:", error.message));
```

## Common Async Antipatterns and Solutions

### Antipattern 1: Unnecessary async/await
```javascript
// Antipattern - unnecessary async wrapper
async function badExample() {
    return await fetch('/api/data'); // Just return the promise!
}

// Better - return promise directly
function goodExample() {
    return fetch('/api/data');
}

// Only use async when you need to await something
async function properAsyncUsage() {
    var response = await fetch('/api/data');
    var data = await response.json();
    return processData(data); // This transformation justifies async
}
```

### Antipattern 2: Not handling Promise rejections
```javascript
// Antipattern - unhandled promise rejection
async function badAsyncCall() {
    fetch('/api/data'); // Promise not awaited or handled
    return "done";
}

// Better - handle the promise
async function goodAsyncCall() {
    try {
        await fetch('/api/data');
        return "done";
    } catch (error) {
        console.error("API call failed:", error);
        throw error;
    }
}

// Or if you don't need to wait
function fireAndForget() {
    fetch('/api/analytics').catch(error => {
        console.error("Analytics failed:", error);
    });
    return "done";
}
```

### Antipattern 3: Sequential when parallel is possible
```javascript
// Antipattern - unnecessary sequential execution
async function slowLoadUserDashboard(userId) {
    var user = await fetchUser(userId);
    var posts = await fetchUserPosts(userId);  // Could be parallel
    var friends = await fetchUserFriends(userId); // Could be parallel
    var notifications = await fetchNotifications(userId); // Could be parallel
    
    return { user, posts, friends, notifications };
}

// Better - parallel independent operations
async function fastLoadUserDashboard(userId) {
    var [user, posts, friends, notifications] = await Promise.all([
        fetchUser(userId),
        fetchUserPosts(userId),
        fetchUserFriends(userId),
        fetchNotifications(userId)
    ]);
    
    return { user, posts, friends, notifications };
}
```

## Interview Questions & Challenges

### Question 1: Event Loop Understanding
```javascript
console.log('1');

setTimeout(() => console.log('2'), 0);

Promise.resolve().then(() => {
    console.log('3');
    return Promise.resolve();
}).then(() => console.log('4'));

queueMicrotask(() => console.log('5'));

setTimeout(() => console.log('6'), 0);

console.log('7');

// What is the output order and why?
```

### Question 2: Promise Chain Debugging
```javascript
// What's wrong with this code? Fix it.
function fetchUserProfile(userId) {
    return fetch(`/api/users/${userId}`)
        .then(response => response.json())
        .then(user => {
            fetch(`/api/users/${userId}/avatar`)
                .then(avatarResponse => avatarResponse.blob())
                .then(avatar => {
                    user.avatar = avatar;
                    return user;
                });
        });
}
```

### Question 3: Async Error Handling
```javascript
// This function has error handling issues. Identify and fix them.
async function processUserData(userId) {
    var user = await fetchUser(userId);
    
    var posts = await fetchPosts(user.id).catch(error => {
        console.log("Posts failed, using empty array");
        return [];
    });
    
    var processed = await processData(user, posts);
    
    return processed;
}

// What happens if fetchUser fails?
// What happens if processData fails?
// How would you improve this?
```

### Question 4: Implement Promise.all
```javascript
// Implement your own version of Promise.all
function myPromiseAll(promises) {
    // Your implementation here
    // Should:
    // - Return a promise
    // - Resolve with array of all resolved values (in order)
    // - Reject immediately if any promise rejects
}

// Test cases
var p1 = Promise.resolve(1);
var p2 = new Promise(resolve => setTimeout(() => resolve(2), 100));
var p3 = Promise.resolve(3);

myPromiseAll([p1, p2, p3]).then(console.log); // [1, 2, 3]
```

### Question 5: Rate Limited API Calls
```javascript
// Create a function that makes API calls with rate limiting
// Max 5 calls per second, queue additional calls
function createRateLimitedFetcher(maxCallsPerSecond) {
    // Your implementation here
    // Should return a function that behaves like fetch
    // but respects rate limits
}

var limitedFetch = createRateLimitedFetcher(5);

// These should be rate limited
for (let i = 0; i < 20; i++) {
    limitedFetch(`/api/data/${i}`)
        .then(response => console.log(`Response ${i}`));
}
```

## Performance and Best Practices

### Memory Management with Async
```javascript
// Memory leak - keeping references in closures
function createLeakyAsync() {
    var largeData = new Array(1000000).fill("data");
    
    return async function() {
        await delay(1000);
        return "done"; // largeData is kept alive due to closure
    };
}

// Better - clear references
function createCleanAsync() {
    var largeData = new Array(1000000).fill("data");
    var result = processLargeData(largeData);
    largeData = null; // Clear reference
    
    return async function() {
        await delay(1000);
        return result;
    };
}
```

### Error Boundary Pattern
```javascript
// Global error handling for unhandled promise rejections
window.addEventListener('unhandledrejection', event => {
    console.error('Unhandled promise rejection:', event.reason);
    
    // Prevent default behavior (console error)
    event.preventDefault();
    
    // Report to error tracking service
    reportError(event.reason);
});

// Node.js equivalent
process.on('unhandledRejection', (reason, promise) => {
    console.error('Unhandled Rejection at:', promise, 'reason:', reason);
});
```

## Key Takeaways for Interviews

1. **JavaScript is single-threaded** but handles async via event loop
2. **Microtasks have priority** over macrotasks in event loop
3. **Promises are better** than callbacks for error handling and chaining
4. **async/await is syntactic sugar** over Promises
5. **Parallel vs sequential execution** affects performance significantly
6. **Error handling** is crucial - use try/catch with async/await
7. **Promise.all vs Promise.allSettled** - know when to use each
8. **Avoid unnecessary async/await** - return promises directly when possible
9. **Handle unhandled rejections** to prevent crashes
10. **Consider timeouts and cancellation** for robust applications

## Practice Challenges

### Challenge 1: Task Queue Manager
```javascript
// Create a task queue that processes tasks with different priorities
// Higher priority tasks should run first
class TaskQueue {
    constructor(concurrency = 3) {
        // Your implementation
        // Should support: add(task, priority), start(), stop()
    }
}

var queue = new TaskQueue(2);
queue.add(() => longRunningTask("high"), 1);
queue.add(() => longRunningTask("medium"), 2);
queue.add(() => longRunningTask("low"), 3);
queue.start();
```

### Challenge 2: Async Cache with TTL
```javascript
// Create an async cache with time-to-live expiration
class AsyncCache {
    constructor(ttlMs = 60000) {
        // Your implementation
        // Should support: get(key, fetcher), set(key, value), clear()
    }
}

var cache = new AsyncCache(5000);

// Should cache for 5 seconds, then refetch
var data = await cache.get('user:123', () => fetchUser(123));
```

### Challenge 3: Async State Machine
```javascript
// Create an async state machine for user authentication flow
// States: logged_out → logging_in → logged_in → logging_out
class AuthStateMachine {
    constructor() {
        // Your implementation
        // Should handle state transitions with async operations
    }
}

var auth = new AuthStateMachine();
await auth.login(credentials);
console.log(auth.state); // 'logged_in'
```
