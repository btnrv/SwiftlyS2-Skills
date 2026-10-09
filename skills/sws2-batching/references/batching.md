# Measuring and scheduling batches

Measure the reported burst: native operation count, callback duration, worst tick/frame time, and total completion time at expected load. Loop elapsed time measures the loop; server tick measurements establish the actual spike. Keep per-item logs out of measured work.

Queuing one `NextTick` callback per item places that work in the same next-tick pass. To distribute an authorized, measured burst, schedule the next bounded slice from inside the current slice. Keep the item operation and its required ordering together.

When the user opts in and measurements support batching, prepare managed work data, process a bounded slice on the game thread, and schedule the next slice from that callback. The synchronous `NextTick` path swaps its pending list before executing callbacks, so work scheduled by a running callback enters a later tick. Preserve operations that must occur together and any required follow-up phase.

Carry stable identities or entity handles into later slices and resolve each target before its native operation. End map-owned work at map change and plugin-owned work at unload. Report completion after the final slice when the contract expects completed work. Retain all-at-once timing when the feature requires it.

Choose batch size from observed costs. Compare worst ticks and completion latency under equivalent load. Native calls retain their thread requirements inside `Task.Run`. Database batching has a separate I/O contract.

Reference: [SchedulerManager](https://github.com/swiftly-solution/swiftlys2/blob/master/managed/src/SwiftlyS2.Core/Modules/Scheduler/SchedulerManager.cs).
