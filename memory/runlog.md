# Desk Monkey Run Log

Append-only log of every routine + local-task run. Each entry uses expected-vs-actual format and lands ✅ Healthy / ⚠️ Drift / 🔴 Failed.

Routines append a `[IN_PROGRESS]` stub at start and replace it with the final report at end. If you see a lingering `[IN_PROGRESS]` entry, that run died mid-execution and the routine needs investigation.
