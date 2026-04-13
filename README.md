# my-node-js-refreshing-notes
A deep-dive repository documenting the internal mechanics, architectural patterns, and performance optimizations of the Node.js runtime. This isn't just a "how-to" guide—it's a "how-it-works" exploration

Core Philosophy
In a world of high-concurrency and real-time data, understanding the abstraction layers of Node.js is the difference between a scalable system and a "laggy" one. This repo tracks the journey through:

The Executioner (V8): Call stacks, heaps, and the Microtask "VIP" checkpoints.

The Orchestrator (libuv C ): The 6 phases of the Event Loop and asynchronous I/O management.

The Binding Layer (C/C++): How JavaScript talks to the OS via Node.js bindings.

📂 Repository Structure

| Folder | Content | Key Focus |
| :--- | :--- | :--- |
| **`01-internals/`** | The "Guts" of the system. | Event Loop, libuv, V8 Bridge, Microtasks. |
| **`02-core-modules/`** | Standard Library Deep-dives. | Streams, Buffers, EventEmitter, FS, Net. |
| **`03-patterns/`** | Architectural Solutions. | Async/Await Flow, Error Handling, Concurrency. |
| **`04-lab/`** | Proof-of-Concept Scripts. | Loop Starvation, I/O Racing, Memory Leak tests. |


```mermaid
graph TD
    A[Timers] --> B[Pending I/O]
    B --> C[Idle/Prepare]
    C --> D[Poll Phase]
    D --> E[Check Phase]
    E --> F[Close Callbacks]
    F --> A
