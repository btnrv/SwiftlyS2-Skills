---
name: sws2-thread-management
description: Keep SwiftlyS2 game operations on the game thread while coordinating asynchronous I/O, scheduled work, and timer lifetimes.
---

Read [dispatch and lifetime](references/threads.md) when work crosses an `await`, background task, tick, or map boundary.

Use `Core.IsGameThread` and API thread annotations to choose the execution context. Complete I/O outside the game thread, then schedule a synchronous callback or use a documented async game API. Resolve current players and entities inside the callback. Keep timers and deferred work within their plugin and map lifetime.

Verify scheduler overloads through SwiftlyS2 MCP for the target runtime. Build and exercise the async route, including disconnect, map change, and hot reload when they affect the feature.
