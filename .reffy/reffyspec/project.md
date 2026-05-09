# Project Context

## Purpose
Warp is an agentic development environment built on a GPU-accelerated terminal emulator. It combines traditional terminal emulation with AI-powered coding assistance, smart command completion, cloud synchronization (Warp Drive), and team collaboration features. The client is open source (this repo); the server and Warp Drive backend are closed source.

## Tech Stack
- **Primary Language:** Rust (Edition 2021, toolchain 1.92.0)
- **Build System:** Cargo workspace with 63 crates
- **Async Runtime:** Tokio
- **UI Framework:** Custom WarpUI (Entity-Component-Handle pattern, inspired by Flutter)
- **Database:** SQLite via Diesel ORM
- **API:** GraphQL (Cynic), WebSocket subscriptions, JSON-RPC
- **HTTP:** Reqwest, Hyper
- **Terminal Emulation:** VTE (VT100-compatible)
- **Serialization:** Serde + serde_json
- **Error Tracking:** Sentry
- **Platforms:** macOS (Metal/Cocoa), Windows (Win32/ConPTY), Linux (X11/Wayland), WASM
- **Secondary Languages:** TypeScript/JavaScript (GraphQL schema generation, command signatures)

## Project Conventions

### Code Style
- Formatter: `cargo fmt` (Edition 2018 in `.rustfmt.toml`)
- Linter: `cargo clippy` with strict rules (`.clippy.toml`)
- Disallowed: `dbg!()` macro, `std::time::Instant` (use `instant::Instant` for WASM compat), `std::process::Command` (use `command::blocking::Command`)
- Avoid unnecessary type annotations in closure params
- Prefer imports over path qualifiers; place imports at top of file
- Context parameters (`AppContext`, `ViewContext`, `ModelContext`) named `ctx` and placed last
- Always remove unused parameters completely (don't prefix with `_`)
- Prefer inline format args: `eprintln!("{message}")` over `eprintln!("{}", message)`
- Do not remove existing comments when making unrelated changes
- Use exhaustive `match` statements (avoid `_` wildcards)

### Architecture Patterns
- **Entity-Handle System:** Views reference other views via `ViewHandle<T>`, not direct ownership
- **Global App Object:** Owns all views/models; `AppContext` provides temporary handle access
- **Modular Crate Structure:** 63 crates in `crates/` covering UI, terminal, AI, persistence, networking, etc.
- **Feature Flags:** Runtime checks preferred over compile-time directives; used for channel-specific builds (stable, preview, dev)
- **Async-First:** Heavy use of Tokio and async patterns throughout
- **Event-Driven:** Action system for UI event handling
- **Cross-Platform:** Native implementations per platform with shared core logic
- **Critical:** Be extremely careful with `model.lock()` on `TerminalModel` — can cause deadlocks (UI freeze). Prefer passing already-locked references.

### Testing Strategy
- **Test Runner:** `cargo nextest` for parallel execution (configured in `.config/nextest.toml`)
- **Unit Tests:** Placed in `${filename}_tests.rs` or `mod_test.rs`, included via `#[cfg(test)] #[path = "filename_tests.rs"] mod tests;`
- **Integration Tests:** Custom framework in `crates/integration/` with builder pattern, 32+ test suites
- **Doc Tests:** `cargo test --doc`
- **Presubmit:** `./script/presubmit` runs fmt, clippy, and tests — must pass before PR submission

### Git Workflow
- **Branch Names:** `<handle>/brief-description` (e.g., `alice/fix-parser`)
- **Commits:** Explain *what* and *why*, not just *what*
- **Readiness Labels:** `ready-to-spec` (design open) and `ready-to-implement` (design settled)
- **Spec-First for Features:** Open spec PR with `product.md` + `tech.md` before code PR
- **Changelog Prefixes:** `CHANGELOG-NEW-FEATURE:`, `CHANGELOG-IMPROVEMENT:`, `CHANGELOG-BUG-FIX:`, `CHANGELOG-IMAGE:` in commit messages
- **PR Review:** Oz agent auto-reviews; re-review via `/oz-review` (up to 3x)

## Domain Context
Warp operates in the **developer tools / terminal emulator** domain. Key areas:
- **Terminal Emulation:** VT100-compatible with GPU acceleration
- **AI Agent Mode:** Built-in coding assistant powered by LLMs, plus support for external agents (Claude Code, Codex, Gemini CLI)
- **Command Intelligence:** Smart completion, suggestions, and corrections via shell integration (bash, zsh, fish)
- **Workspace Management:** Multi-session, multi-device synchronization
- **Team/Enterprise Features:** Session sharing, cloud sync, collaborative workflows

## Important Constraints
- **Dual License:** WarpUI crates (`warpui_core`, `warpui`) are MIT; everything else is AGPL-3.0-only
- **WASM Compatibility:** Must use `instant::Instant` instead of `std::time::Instant`; custom `Command` wrappers instead of `std::process::Command`
- **Deadlock Risk:** `TerminalModel` locking requires extreme care to avoid UI freezes
- **Presubmit Gate:** `cargo fmt` and `cargo clippy` must pass before any PR

## External Dependencies
- **Warp Server:** Closed-source backend API
- **Warp Drive:** Cloud sync backend (closed source)
- **Sentry:** Crash/error reporting
- **Firebase:** Cloud services integration
- **OAuth2:** Authentication flows
- **Docker:** Development containers (`docker/agent-dev/`, `docker/linux-dev/`)
- **GitHub Actions:** CI/CD pipelines (`.github/workflows/`)
