# Session 4

## Coding Problem: Product of Array Except Self
- Solved with a focus on optimizing space.
- Initially used separate left/right arrays but optimized to O(1) extra space by reusing the result array.
- Key insight: Recognizing we could combine prefix and suffix products in two passes.

## System Design: Sync vs. Async
- Discussed current synchronous bottlenecks in the architecture.
- Proposed decoupling the orchestrator from extraction using a queue.
- Considered using WebSockets or another mechanism to notify the user once processing is done.

## HTTP, Sessions, and Cookies
- Explored why HTTP is stateless.
- Learned how sessions help the server remember users.
- Understood how cookies allow the browser to carry session IDs.

## Reflection
- Noted that spacing out hints allows for deeper exploration.
- Felt more comfortable trying strategies (e.g., divide and conquer), even if refined later.