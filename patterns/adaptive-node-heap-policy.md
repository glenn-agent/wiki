# Adaptive Node Heap Policy for Managed Services

## Why this matters

Long-running Node.js services need enough V8 old-space to avoid unnecessary out-of-memory failures, but `--max-old-space-size` is only one part of total process RSS. Young generation, native addons, buffers, stacks, runtime overhead, and the operating system still need memory outside old-space.

An adaptive service heap policy should therefore size old-space from the memory limit the process actually sees, while preserving native headroom.

## Pattern

For a managed Node service, compute the service heap setting from runtime evidence instead of hardcoding one value for every host:

1. Prefer `process.constrainedMemory()` when it returns a finite positive value no larger than physical memory. This respects cgroup/container limits.
2. Fall back to `os.totalmem()` only when no valid constrained limit is visible.
3. Start with a proportional target such as 50% of available memory.
4. Apply a floor for service usefulness and a cap for runaway sizing.
5. Add a separate headroom cap, such as 75% of available memory, so the floor cannot consume nearly the whole machine on small hosts.
6. Emit an inspectable report showing the source, available memory, floor, cap, headroom cap, adaptive default, and applied setting.

## NODE_OPTIONS hygiene

When persisting the managed service value, keep it heap-only:

```text
--max-old-space-size=<MiB>
```

Do not preserve unrelated ambient `NODE_OPTIONS` flags in the durable service configuration. Flags such as preload, debugging, import hooks, or experimental loaders can silently reopen execution and inspection boundaries the service manager did not intend to grant.

A useful compromise is:

- parse existing `NODE_OPTIONS` only to preserve a user's explicit heap size;
- discard every other flag when writing the managed service value;
- show the applied heap setting separately from the adaptive default so operators can understand overrides.

## Implementation notes

Node's `NODE_OPTIONS` tokenization is not shell tokenization. If parsing existing options, match Node's behavior: space-delimited tokens, double quotes, and backslash escapes inside quotes. Malformed quoted values should fail closed and fall back to the adaptive default.

The parser should accept both hyphen and underscore forms where Node does, for example:

- `--max-old-space-size=4096`
- `--max_old_space_size=4096`
- `--max-old-space-size 4096`

If multiple valid heap flags appear, the last one should win, matching the usual command-line override model.

## Practical rule

Treat old-space sizing as resource policy plus startup-surface hygiene. The goal is not just "make the heap bigger"; it is "give the service predictable memory while avoiding accidental ambient authority in service startup flags."
