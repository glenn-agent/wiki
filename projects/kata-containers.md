# Kata Containers

Kata Containers is an open-source project for lightweight virtual machines that feel and perform like containers while providing VM-style workload isolation.

## Local Setup

- Upstream clone: `/workspace/openclaw/projects/kata-containers/upstream`
- Upstream remote: `https://github.com/kata-containers/kata-containers.git`
- Glenn-Agent fork: `https://github.com/glenn-agent/kata-containers`
- Fork remote name in local clone: `glenn`

Contribution posture: study only for now. Do not open issues, PRs, or push contribution branches for Kata Containers until coder-glenn explicitly approves that the timing is mature.

## Initial Architecture Map

Main components from the repository README:

- `src/runtime` — Go runtime and containerd shim v2 implementation.
- `src/runtime-rs` — Rust runtime implementation.
- `src/agent` — guest-side management process running inside the VM/pod to set up container environments.
- `src/dragonball` — optional built-in VMM optimized for container workloads.
- `tools/packaging` — packaging scripts and metadata for components, hypervisors, kernel, and rootfs.
- `tools/osbuilder` — tooling for rootfs/initrd image creation.
- `src/tools/agent-ctl` — low-level agent testing utility.
- `src/tools/kata-ctl` — advanced debug/control utility.
- `tests` — integration, functional, and stability tests.
- `docs` — installation, design, developer, threat model, tracing, release, and contribution documentation.

## Languages and Scale Snapshot

Initial repository scan showed a large mixed Go/Rust/shell/docs codebase:

- Go files: 589
- Rust files: 637
- Shell files: 142
- Markdown files: 225

Top-level file distribution is dominated by `src`, `tools`, `tests`, `docs`, `.github`, and `ci`.

## Contribution Norms to Learn Before Touching Code

From `docs/code-pr-advice.md` and project documentation:

- Prefer generic code over architecture-specific code when possible.
- Keep code readable rather than clever.
- Minimize required privileges.
- Avoid magic numbers/strings; use constants when repeated.
- Add copyright and SPDX headers to new files.
- Link incomplete TODO/FIXME work to GitHub issues.
- Keep functions relatively short and return errors rather than discarding them.
- Prefer structured logging.
- For Go structs, consider constructors to enforce required fields.
- In Rust, minimize `unsafe`; avoid `unwrap()`/`expect()` outside accepted cases.
- Code changes should usually include unit tests where possible.

## Study Plan

1. Read the architecture docs:
   - `docs/design/architecture`
   - `docs/design/architecture_4.0`
   - `docs/design/end-to-end-flow.md`
2. Understand runtime boundaries:
   - host runtime/shim
   - guest agent
   - VMM/hypervisor integration
   - image/rootfs/initrd creation
3. Learn test categories and local feasibility:
   - unit tests
   - functional tests
   - integration tests requiring containerd/Kubernetes/hypervisor setup
4. Watch issues and PRs without intervening until explicitly approved.

## Relevance to Glenn-Agent

Kata is relevant because Glenn-Agent is already working around sandboxing, isolation, runtime boundaries, and secure execution in OpenClaw/NemoClaw. Kata offers a deeper VM-isolation stack to study, especially for long-term understanding of secure agent runtimes and container-like developer experience backed by virtual machines.
