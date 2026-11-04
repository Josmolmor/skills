---
name: "rust-egui-guidelines"
description: "Mandatory rules for any edit or review in the Rust + egui codebase: no unsafe, no new dependencies, enforced lints, ownership, per-frame cost, error display, crate layering, tests and CI gates."
---

# Rust + egui codebase guidelines (mandatory)

Apply this skill every time you write, edit, refactor or review code in this Rust + egui codebase. Every rule below is **mandatory**. If a task seems to require breaking one, stop and ask the user before doing it — never break a rule silently to make something compile.

The compiler checks the types; your job is to get the intent right: behaviour, state, performance and structure.

Remember: there is no error boundary in egui. A panic anywhere takes down the whole app, which is why the escape-hatch rules below are strict.

## 1. Hard prohibitions

1. **No `unsafe`.** No `unsafe` blocks, `unsafe fn`, `unsafe impl` or `unsafe trait`, anywhere, for any reason. Keep `unsafe_code = "forbid"` under `[workspace.lints.rust]` and never weaken it.
2. **No new crates (dependencies) unless there is truly no alternative.** Solve the problem with the standard library, egui/eframe, and dependencies already in the workspace. If a new third-party dependency seems unavoidable, stop and ask the user first, explaining why existing code and std can't do it. Never add one silently. Never bump versions, enable extra features or pull in an async runtime as a side effect.
3. **No new escape hatches:** no `.unwrap()`, `.expect()`, `todo!()`, `unimplemented!()`, `panic!()` or `dbg!()` in non-test code. Return or handle a `Result`/`Option` instead.
4. **No muted warnings.** No new `#[allow(...)]` to silence a lint. Fix the cause. If a lint is genuinely wrong for one spot, ask the user, then use `#[expect(lint_name, reason = "…")]` — never a bare `#[allow]`.
5. **No unexplained `let _ =`.** A `#[must_use]` value (every `Result`, many egui/std functions) must be handled with `?`, `match`, or `if let Err(e)` and surfaced per section 4. Dropping it is only allowed with a comment on the same line saying why ignoring it is correct.

## 2. Lints enforce the rules

The machines review first. The workspace root `Cargo.toml` should carry these lints, and every member crate should have `[lints] workspace = true`:

```toml
[workspace.lints.rust]
unsafe_code = "forbid"

[workspace.lints.clippy]
unwrap_used = "deny"
expect_used = "deny"
todo = "deny"
unimplemented = "deny"
dbg_macro = "deny"
let_underscore_must_use = "deny"
allow_attributes_without_reason = "deny"
```

and a `clippy.toml` with:

```toml
allow-unwrap-in-tests = true
allow-expect-in-tests = true
```

- If any of these are missing, tell the user and propose adding them. Ask before adding them, because existing violations may need fixing first.
- Never remove, downgrade or locally override these lints. A crate opting out of `[lints] workspace = true` is a blocked change.

## 3. Ownership, signatures and design

- **Parameters borrow, struct fields own.** Take `&str` not `&String`, `&[T]` not `&Vec<T>`, `&Path` not `&PathBuf`. State structs own their data (`String`, `Vec<T>`, `PathBuf`) — no references or lifetime parameters in UI state structs.
- **Don't clone your way past the borrow checker.** `.clone()` / `.to_string()` is fine for small values or a genuine need for a second owner; never added only to silence a moved-value or borrow error. Restructure instead.
- **Avoid shared mutable state.** No `Rc<RefCell<_>>` in UI code (runtime-checked, can panic). `Arc<Mutex<_>>` only at real thread boundaries, never as a general UI state container.
- `&self` methods must not hide mutation through interior mutability unless that's clearly intended and documented.
- **Make signatures the contract:** `&mut` only when the function really needs exclusive access; fallible functions return `Result<T, E>` with a meaningful error type (follow the error-type pattern the codebase already uses).
- Prefer `enum` + exhaustive `match` for UI/state modes; avoid catch-all `_ =>` arms that would swallow new variants.
- **No over-abstraction.** Don't introduce traits, generics, trait objects, builders or macros where a plain struct and function do the job. Add an abstraction only when there are already two or more real uses that need it.

## 4. Errors reach the user

Handling a `Result` means the failure ends up somewhere visible, not just logged.

- Use the codebase's existing convention for showing errors (status line, toast, inline message next to the widget, etc.). Find it before writing new error display code. If there is no convention yet, ask the user which one to use rather than inventing one per screen.
- Failures of user-initiated actions (save, load, send, import) are always shown to the user. Logging alone is not enough.
- Store pending errors on the screen's state struct so they survive across frames until dismissed or replaced.
- Logging (via whatever logging the workspace already uses) is for diagnostics in addition to display, not instead of it.

## 5. Immediate mode (egui)

The UI function runs every frame. Read it top to bottom and you've read the screen.

- **State lives on a struct, never in a local.** A `let mut open = false;` inside a `ui()` function resets every frame. Persistent state goes on the screen's struct (or egui memory for small widget-internal state).
- **Nothing blocks the frame.** No file I/O, network calls, sleeping or heavy compute inside `update()` / `ui()`. Move it to background work (see section 7 for the mechanism), store the receiver on the struct (e.g. `self.pending: Option<Receiver<_>>`), poll it with `try_recv` each frame, and send back a `Result` so the view can show errors. Handle the case where a new request replaces an old pending one.
- **Always `ctx.request_repaint()`** after background work delivers a result, or the UI won't update until the mouse moves.
- **Unique IDs in loops.** Widgets created in a loop get a unique id (`ui.push_id(id, |ui| …)` or `id_salt`) so rows don't share state.
- **Organise each screen as State / View / Logic:**
  - State: `struct Screen` — plain data.
  - View: `fn ui(&mut self, ui: &mut egui::Ui)` — draws and reacts; thin; calls into logic.
  - Logic: plain functions with **no egui types** — validation, calculations, transitions.
  - Test: could this logic be unit-tested without any UI? If not, move it.
- **Use the pinned egui version's API.** Check the version in `Cargo.toml`/`Cargo.lock` and use that version's docs; don't write code for older egui releases or introduce deprecated calls.

## 6. Per-frame cost

There is no re-render to memoise against, but the whole UI function is the render cost, paid on every frame.

- **Don't recompute every frame.** Sorting, filtering, parsing, layout-heavy string building or any O(n) work over large data happens when the input changes, with the result cached on the state struct. The view reads the cache.
- **Never create textures in the frame loop.** Load an image once with `ctx.load_texture` (or the codebase's image-loading approach), store the `TextureHandle` on the struct and reuse it. Calling it every frame leaks GPU memory and slows everything down.
- **Don't repaint continuously.** No unconditional `ctx.request_repaint()` every frame. Request a repaint only when something changed, or use `request_repaint_after(duration)` for timed updates such as animations or polling.
- **Virtualise long lists.** For lists or tables with many rows, use `ScrollArea::show_rows` (or the table widget's row virtualisation) so only visible rows are drawn.

## 7. Platform targets (native and web)

Before writing any background work, check whether the app also builds for the web (`wasm32-unknown-unknown`): look for a `Trunk.toml`, an `index.html`, eframe's web features, wasm targets in CI or `.cargo/config.toml`.

- **Native only:** std `thread::spawn` + `mpsc::channel` is the default mechanism.
- **Also targets web:** `std::thread` and blocking calls don't work there. Use the mechanism the codebase already uses for web-compatible background work (or ask the user if there is none — don't add an async runtime on your own). All code must compile for every target the app ships; run `cargo check --target wasm32-unknown-unknown` (for the relevant crates) alongside the native gates.

## 8. Crate structure (workspace layering)

The workspace is wide and shallow; crate boundaries decide build speed.

- Layers, top to bottom: **app** (thin binary, wiring only) → **feature UI crates** (`ui-<feature>`) → **shared** (`ui-widgets`, `storage`, `network`, …) → **core** (domain types and logic, no egui).
- **Dependencies only point down. Siblings never depend on each other.**
- Blocked changes:
  - `core` → `egui` (or any UI crate).
  - One feature crate → another feature crate.
  - A catch-all `utils` crate that everything depends on.
- Crates live in a flat `crates/<name>` folder; shared dependency versions go in `[workspace.dependencies]` and members use `dep.workspace = true`.
- Put new code in the lowest layer it belongs to. Don't create a new workspace crate without asking the user.
- Reuse widgets from `ui-widgets` instead of re-implementing similar ones in a feature crate.
- When you touch any `Cargo.toml` or `Cargo.lock`, state explicitly what changed in dependencies and why. Unexpected `Cargo.lock` churn (new packages, duplicate versions of the same crate) must be explained or reverted.

## 9. Testing

- Pure logic gets ordinary `#[test]` unit tests in its crate.
- New or changed UI flows get an `egui_kittest` test (`Harness`) that walks the spec's flow: find widgets by label/role, simulate clicks and typing, `harness.run()`, assert on state or shown text. Match the `egui_kittest` API to the pinned version. (Only if `egui_kittest` is already a dev-dependency — otherwise ask before adding it, per rule 1.2.)
- Error paths get tests too: a failed load or save shows an error rather than crashing.
- `unwrap()`/`expect()` are acceptable inside tests.

## 10. Reviewing a change

When reviewing a PR or your own diff, read it in this order, because each step tells you the blast radius of the next:

1. `Cargo.toml` / `Cargo.lock` changes — new dependencies, new arrows between crates, lint changes.
2. Function and struct signatures — `&mut`, `Result`, ownership.
3. Tests — do they walk the spec's flow and the error paths?
4. Bodies — against the checklist below.

## 11. Before you finish: the gates

Run these and make them pass before declaring work done (report any you couldn't run):

```
cargo fmt --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
cargo deny check                                   # if configured
cargo check --target wasm32-unknown-unknown        # if the app builds for web
```

Then self-review the diff against this checklist and grep it for `unwrap(`, `expect(`, `unsafe`, `todo!`, `dbg!`, `#[allow`, `let _ =`, `clone()`, `RefCell`, `Mutex`, `load_texture`, `request_repaint`:

- [ ] It does what the spec says.
- [ ] A test (unit or kittest) covers the flow and its error paths.
- [ ] No `unsafe`, no new escape hatches (`unwrap` · `expect` · `#[allow]` · `todo!`).
- [ ] Workspace lints are intact; no crate opts out.
- [ ] No new crate dependency (or the user approved it explicitly); `Cargo.lock` changes are explained.
- [ ] Nothing blocks the frame; background results trigger `request_repaint()`.
- [ ] Nothing expensive runs every frame: no per-frame recompute, texture loading or unconditional repaint; long lists are virtualised.
- [ ] Every `Result` is handled — no unexplained `let _ =` — and user-facing failures are shown using the codebase's convention.
- [ ] State lives on a struct, not a local; loop widgets have unique IDs.
- [ ] Logic is outside the UI and free of egui types; no needless traits or generics.
- [ ] Code compiles for every target the app ships (native, and web if applicable).
- [ ] Crate dependencies still point down.

In your summary to the user, mention any rule you had to bend (with their approval) and any gate you could not run.