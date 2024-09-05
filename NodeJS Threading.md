# JavaScript is single threaded or multi  threaded and its event loop 

JavaScript is traditionally a single-threaded language. This means that it can only execute one task at a time in the main thread. 

## Event Loop in JavaScript
Though JavaScript is single-threaded, it handles asynchronous operations like I/O, timers, or network requests using an architecture built around the event loop. The event loop allows JavaScript to perform non-blocking operations despite having only one thread. Here’s how it works:

> Call Stack: JavaScript executes code in a call stack, which is a data structure that follows a Last In, First Out (LIFO) principle. The synchronous code is executed here.

>Web APIs: Asynchronous operations like setTimeout, HTTP requests (via fetch or XMLHttpRequest), or DOM event listeners are offloaded to Web APIs. These APIs handle these tasks and inform the event loop when they are ready.

>Callback Queue (Task Queue): When an asynchronous task completes (e.g., a network request finishes or a timer ends), its associated callback function is placed in the callback queue.

>Event Loop: The event loop constantly checks if the call stack is empty. If it is, the event loop picks up tasks from the callback queue and pushes them onto the call stack for execution.

>Microtask Queue: This is a special queue that holds higher-priority tasks such as promises. The event loop will clear the microtask queue before moving on to the callback queue.

>This cycle allows JavaScript to handle multiple asynchronous operations without blocking the main thread.

## Functions and classes available to Web Workers in short

Web Workers in JavaScript run in a separate thread and have access to a limited set of functions, APIs, and classes. They don't have access to the DOM or certain browser-specific objects, but they can still perform many useful tasks.

>read more about [Functions and classes available to Web Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Functions_and_classes_available_to_workers)

>note you can not execute js code except these functions in web workers for example for loop or other function will not be execute in web worker it will be send to call stack async operation

```javascript
console.log('Start of script');
setTimeout(() => {
    const end = Date.now() + 6000; 
    console.log("wait");
            while (Date.now() < end) {
                /*Note after comming out of timer it will run for more 5 sec beacuse 
                in setTimeout only time will be checked and then it will passed to 
                event queue and further passed to call stack, there it execute callback function */
            }
    console.log("Delayed for 5 second.");
  }, "5000");
  console.log('End of script');

```
Functions and Classes Available to Web Workers
Global Functions:

postMessage(): Sends messages from the worker to the main thread.
onmessage: Event listener for receiving messages from the main thread.

importScripts(): Imports external scripts into the worker.

setTimeout() / clearTimeout(): Sets and clears timers.

setInterval() / clearInterval(): Sets and clears intervals.
Worker-Specific APIs:

WebSockets: WebSocket for real-time communication.

XMLHttpRequest: For making HTTP requests (although fetch is more common).
Fetch API: For network requests.
File API: For reading and writing files (FileReader, Blob, etc.).

MessagePort: Allows communication between different workers and the main thread.
BroadcastChannel: Enables message broadcasting to multiple contexts (windows, workers).

IndexedDB: For local database storage.

Crypto API: For secure random number generation, hashing, etc. (crypto.subtle).

SharedArrayBuffer: For sharing memory between threads.

Atomics: For performing atomic operations on shared memory.

Classes:

Worker: Represents a Web Worker instance.
MessageEvent: Represents an event triggered by postMessage().
ErrorEvent: Handles errors that occur in the worker.
EventTarget: Basic class for handling events.
Restrictions:
No DOM access: Workers can't manipulate the DOM (document, window objects are not available).
No localStorage or sessionStorage.
No access to alert, prompt, or confirm dialogs.

# NodeJS is single threaded or multi  threaded and its event loop 

NodeJs is single threaded its JavaScript engine is V8 and file I/O, Network calls  is handled by [libuv](https://docs.libuv.org/en/v1.x/) 

>Alothough we can achieve multi threading in NodeJS using Child Process, Cluster and Worker Thread 

## Child Process:
The child process module in Node.js allows you to create new processes from your Node.js code. It can run shell commands or spawn other Node.js scripts. This is useful for executing heavy tasks in parallel to the main event loop.

Types:

spawn(): Starts a new process and streams input/output.

exec(): Runs a command in a shell and buffers the output.

fork(): Creates a new Node.js process specifically for communication between parent and child.

execFile(): Similar to exec(), but directly executes a file without spawning a shell.

Example:
```javascript

const { spawn } = require('child_process');
const ls = spawn('ls', ['-lh', '/usr']);

ls.stdout.on('data', (data) => {
  console.log(`Output: ${data}`);
});
```
Read more about [child_process](https://nodejs.org/api/child_process.html)

## Cluster API:
The Cluster API is used to scale a Node.js application across multiple CPU cores. It allows you to run multiple instances (workers) of your Node.js application, each handling incoming requests. Each worker is a separate Node.js process but shares the same server port.

Usage: To improve the performance of applications by distributing workload across available CPUs.

Master-Worker Model: The master process controls and spawns multiple worker processes. If a worker dies, the master can restart it.

Example:

```javascript

const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
} else {
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end('Hello World');
  }).listen(8000);
}
```
Read more about [Cluster](https://nodejs.org/api/cluster.html)

## Worker Threads:
Worker Threads in Node.js allow for running JavaScript in parallel in multiple threads. Unlike child processes, worker threads share memory with the main thread and other workers, which makes them more efficient for CPU-bound tasks.

Usage: To offload CPU-intensive tasks like data processing, image manipulation, etc., without blocking the main thread.

Shared Memory: Uses SharedArrayBuffer and Atomics for shared memory access between threads.

Example:

```javascript

const { Worker, isMainThread, parentPort } = require('worker_threads');

if (isMainThread) {
  const worker = new Worker(__filename);
  worker.on('message', message => console.log('From worker:', message));
} else {
  parentPort.postMessage('Hello from worker');
}
```

Summary:

Child Process: Runs separate OS processes, useful for running shell commands or Node.js scripts.
Cluster API: Enables running multiple Node.js processes to take advantage of multi-core systems, used primarily for scaling servers.
Worker Threads: Provides multithreading in Node.js, useful for offloading CPU-intensive tasks while sharing memory.


###  when you need to go multitaking using cluster 

You would need to use the Cluster API in Node.js when you want to scale your application to handle more load by utilizing multiple CPU cores. Since Node.js runs on a single thread, it can only handle one task at a time on one core. However, most modern machines have multiple cores, so to fully utilize all available cores, you can use the Cluster API to create multiple instances (workers) of your application running on each core.

Here are some scenarios when you should use the Cluster API:

1. High Traffic Web Servers:
When building a server (like with http or express), Node.js can handle requests efficiently but only on one core. If your application is under heavy load, a single-threaded instance might not be able to handle all incoming requests. With the Cluster API, you can create multiple workers, each handling requests on a separate core, thereby distributing the load and increasing performance.

2. CPU-Bound Tasks:
If your application involves tasks that are CPU-intensive (e.g., image processing, video encoding, or cryptographic calculations), a single core may become overburdened. Using the Cluster API, you can split the tasks across multiple processes, with each process running on a different CPU core. This allows you to fully utilize multi-core processors and distribute the load.

3. Horizontal Scaling:
If you’re building a scalable web app that needs to handle many concurrent users or requests, the Cluster API allows you to achieve horizontal scaling on a single machine by leveraging multiple cores. You can run one worker per CPU core, all sharing the same network port, thereby increasing your server's throughput.

4. Fault Tolerance:
With the Cluster API, if a worker process crashes (due to errors in code, heavy processing, etc.), the master process can automatically spawn a new worker to replace it. This provides fault tolerance and ensures that the application remains available despite individual worker failures.

5. Avoiding Blocking Operations:
In certain cases, even though Node.js is non-blocking, long-running synchronous or heavy tasks (like large file processing or computation) can block the event loop. By clustering, you can ensure these tasks are distributed across workers, which prevents blocking the main event loop and keeps your application responsive.

### When NOT to Use the Cluster API:
I/O-bound applications: If your app is mostly performing I/O-bound tasks (like database access, file reads/writes, network requests), Node.js's asynchronous, non-blocking nature can handle this efficiently without the need for clustering.
Low traffic apps: If your app doesn't face heavy traffic or complex CPU tasks, clustering might add unnecessary complexity.

### Summary: You should go for a multitasking engine using Cluster API when:

1. Your application experiences high traffic or concurrent connections.
2. You have CPU-intensive tasks that need to utilize multiple cores.
3. You need to improve fault tolerance and load balancing.
4. You want to maximize CPU core usage to increase performance.