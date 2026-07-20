# Task M0 — Repository bootstrap

## Operating mode

Begin in planning mode. Do not modify files until you have inspected the repository and presented a concrete plan.

## Goal

Create a clean stable-Rust Cargo workspace and development scaffold for MOSAIC without implementing protocol behavior.

## Required context

Read `AGENTS.md` and every document under `docs/` before planning.

## Required deliverables

1. Create a Cargo workspace.
2. Add only the crates needed to begin M1:
   - `mosaic-core`;
   - `mosaic-codec`;
   - `mosaic-packet-device`;
   - `mosaic-carrier`;
   - `mosaic-sim`;
   - `mosaic-cli`.
3. Each crate must compile and contain crate-level documentation explaining its current empty scope.
4. Add workspace-wide lint configuration appropriate for stable Rust.
5. Add `.gitignore`, `rust-toolchain.toml`, and formatting configuration only when justified.
6. Add CI that runs formatting, clippy, and tests on a supported stable Rust environment.
7. Create `docs/adr/0001-workspace-and-language.md` recording:
   - stable Rust choice;
   - edition choice;
   - initial crate boundaries;
   - why TUN, cryptography, HTTP/3, and scheduler crates are not yet created.
8. Add a small smoke test proving the workspace test command executes.
9. Update README with exact local build and test commands.

## Constraints

- No codec implementation.
- No wire-format constants.
- No cryptographic dependencies.
- No async runtime unless an empty abstraction genuinely requires it; prefer none.
- No sockets.
- No TUN.
- No route or firewall commands.
- No HTTP libraries.
- No generated secrets.
- Do not create all future crates as empty placeholders.

## Plan response must include

- proposed file tree;
- dependency graph;
- edition/toolchain decision;
- CI approach;
- exact commands to verify M0;
- risks and open questions.

## Acceptance criteria

- `cargo metadata` succeeds;
- `cargo fmt --all --check` succeeds;
- `cargo clippy --workspace --all-targets --all-features -- -D warnings` succeeds;
- `cargo test --workspace --all-features` succeeds;
- no later-milestone dependency or implementation exists;
- ADR and README match the workspace.

## Required final report

Use the report format from `AGENTS.md` and explicitly confirm that no protocol, crypto, network, TUN, HTTP/3, or scheduler code was added.
