# my-node-js-refreshing-notes
A deep-dive repository documenting the internal mechanics, architectural patterns, and performance optimizations of the Node.js runtime. This isn't just a "how-to" guide—it's a "how-it-works" exploration

Core Philosophy
In a world of high-concurrency and real-time data, understanding the abstraction layers of Node.js is the difference between a scalable system and a "laggy" one. This repo tracks the journey through:

The Executioner (V8): Call stacks, heaps, and the Microtask "VIP" checkpoints.

The Orchestrator (libuv C ): The 6 phases of the Event Loop and asynchronous I/O management.

The Binding Layer (C/C++): How JavaScript talks to the OS via Node.js bindings.
📂 Repository StructureFolderContentKey Focus01-internals/The "Guts" of the system.Event Loop, libuv, V8 Bridge, Microtasks.02-core-modules/Standard Library Deep-dives.Streams, Buffers, EventEmitter, FS, Net.03-patterns/Architectural Solutions.Async/Await Flow, Error Handling, Concurrency.04-lab/Proof-of-Concept Scripts.Loop Starvation, I/O Racing, Memory Leak tests.🛠 Key Takeaways (Mental Models)The "After-Every-Task" RuleMicrotasks (process.nextTick and Promises) are interrupts. They don't wait for a phase to end; they drain immediately after every single callback finishes execution.The Poll Phase "Sleep"The Event Loop stays efficient by "parking" in the Poll Phase to wait for I/O. It will only sleep if the Check Phase is empty (no setImmediate) and no timers have expired.

```mermaid
graph TD
    A[Timers] --> B[Pending I/O]
    B --> C[Idle/Prepare]
    C --> D[Poll Phase]
    D --> E[Check Phase]
    E --> F[Close Callbacks]
    F --> A
