# Stakpak agent fork

This Rust workspace provides the Stakpak CLI/TUI, provider adapters, MCP support, and autonomous runtime. `cli/` owns commands and configuration; `tui/` owns UI events; `libs/ai`, `libs/api`, `libs/shared`, and MCP crates own provider, context, storage, and tool boundaries. Read [CONTRIBUTING.md](CONTRIBUTING.md) and [detailed guidance](docs/agent-guidance.md) for the code/data-flow map and subsystem rules.

## Verification

Follow [.github/workflows/ci.yml](.github/workflows/ci.yml), currently Rust `1.94.1`: `cargo fmt -- --check`, `cargo clippy --all-targets -- -D warnings`, `cargo build --verbose`, and `cargo test --workspace --verbose`. Run the relevant SQLite/libsql feature tests when changing those paths; network-provider tests are a distinct lane requiring their documented environment. The old guide's blanket nightly requirement is superseded by the checked-in CI toolchain.

## Preserve these invariants

- Production code denies `unwrap()`/`expect()`; use contextual errors (`anyhow` in application code, `thiserror` in libraries). Tests use existing helpers and async test conventions.
- Never truncate Rust strings using unchecked byte offsets. Use character iteration or validated UTF-8 boundaries.
- Keep tool call/result identifiers paired exactly once. Preserve cancellation/retry behavior, orphan/duplicate sanitization, alternating provider roles, and immediate-parent tool-result references.
- Cache breakpoints belong in the provider conversion layer. Budget trimming retains stable message structure and persists trimming metadata through checkpoint/hook flows.
- Preserve readonly profiles and secret substitution boundaries. Config/auth files can contain credentials; do not expose them in logs or patches.

`stakpak up`/autopilot scheduling/channel commands install or alter persistent runtime behavior and may reach infrastructure or send messages. Build/test work does not authorize starting services, changing schedules/channels, or contacting live systems. Use the detailed setup notes only for explicitly scoped runtime work.
