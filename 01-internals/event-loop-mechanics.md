Event Loop Mechanics: The libuv Orchestrator

The Node.js Event Loop is the mechanism that allows Node to perform non-blocking I/O operations—despite JavaScript being single-threaded—by offloading operations to the system kernel whenever possible.

1. The 6 Phases of libuv
The loop is a "train" that stops at specific stations. Each station handles a specific type of task.

| Phase | Responsibility | Key Mechanics |
| :--- | :--- | :--- |
| **1. Timers** | `setTimeout` / `setInterval` | Executes callbacks of timers whose threshold (ms) has passed. |
| **2. Pending** | System I/O Callbacks | Executes I/O callbacks deferred from the previous loop (e.g., TCP errors). |
| **3. Idle/Prepare** | Internal Housekeeping | Used only by the runtime; keeps the loop "ready" for the next transition. |
| **4. Poll** | I/O Execution & Waiting | Retrieves new I/O events. Node will "park" here to wait for network/file data. |
| **5. Check** | `setImmediate` | Executes callbacks immediately after the Poll phase completes. |
| **6. Close** | Cleanup Logic | Executes cleanup handlers, such as `socket.on('close', ...)`. |

2. The Microtask "Checkpoints"
Unlike the phases above (which are managed by libuv), Microtasks are managed by Node's C++ layer and V8. They do not wait for a phase to finish.

Crucial Architect Note: A microtask checkpoint is triggered immediately after every single callback that completes in any phase.

Priority Hierarchy:
process.nextTick: The "High Priority" queue.

Promises (.then, await): Handled by the V8 engine.

If we  recursively call process.nextTick, the loop will stay at the checkpoint forever, starving the rest of the phases (Timers, Poll, etc.).

3. The Poll Phase Decision LogicThe loop "parks" in the Poll phase to wait for work. It uses the following logic to decide when to move:If setImmediate callbacks are waiting $\rightarrow$ Move to Check Phase.If the queue is empty and no timers are due $\rightarrow$ Sleep (Wait for I/O).
If timers are due $\rightarrow$ Move through Check and Close to start a New Loop.

4. Architectural Patterns: setImmediate vs setTimeout(0)
When executed inside an I/O callback (Poll phase), the order is guaranteed:

/**
 * ARCHITECTURAL PATTERN: Non-blocking Background Tasks
 * * Logic: When inside an I/O callback (Poll Phase), the 'Check' phase
 * is the very next stop in the loop. This guarantees that setImmediate
 * runs before the loop starts over to check for timers.
 */

const fs = require('fs');

fs.readFile('large-file.json', () => {
    // We are now in the Poll Phase
    
    setImmediate(() => {
        // This runs in the Check Phase (Station 5)
        // Guaranteed to run before any setTimeout(fn, 0)
        console.log('Processed after I/O response is sent');
    });

    setTimeout(() => {
        // This runs in the Timers Phase (Station 1) of the NEXT loop iteration
        console.log('Processed in the next loop rotation');
    }, 0);
});
