# MXM patch set for egui-baseview 0.7.1

Upstream crate: `egui-baseview 0.7.1` (0.7.0 until the 2026-10-06 refresh below), MIT OR Apache-2.0. The upstream README and both licence texts
remain unchanged. `screenshot.png` from the published package is intentionally omitted: it is not
needed to build and the software repository carries no third-party image assets.

## Native file drag-and-drop delivery

**File:** `src/window.rs`; every hunk contains `MXM PATCH`.

`baseview 0.3.2` exposes `MouseEvent::{DragEntered, DragMoved, DragLeft, DragDropped}` with portable
`DropData::Files(Vec<PathBuf>)` (unchanged in `baseview 0.3.4`), but egui-baseview 0.7.0's event
match ignored all four variants, and 0.7.1's still does.
That made a native sample drop impossible to observe in an egui editor even though the platform
adapter had already delivered it.

The patch:

- adds a native path-backed implementation of `egui::DroppedFile`;
- updates egui's logical pointer position for drag enter/move/drop;
- fills `RawInput::hovered_files` on enter/move and clears it on leave/drop;
- fills `RawInput::dropped_files` on drop; and
- returns `EventStatus::AcceptDrop(DropEffect::Copy)` for non-empty file payloads.

It adds no platform-specific code or event source. Baseview remains responsible for Windows, Linux
and macOS native delivery. The sampler's editor consumes only file paths and performs all I/O on its
background task executor.

## Refresh/drop procedure

1. Extract a new published egui-baseview over this folder, retaining this file.
2. Check whether its event adapter now maps all four baseview drag variants into egui raw input and
   accepts file drops.
3. If it does, remove the root `[patch.crates-io]` entry and this directory, then run the native
   sampler drop gate on Windows, Linux and macOS.
4. Otherwise reapply only the `MXM PATCH` hunks and build every editor consumer.

## Refreshes

- **2026-10-06: 0.7.0 → 0.7.1.** Needed because nice-plug's refresh onto 0.4.2 brings
  nice-plug-egui 0.5.1, which requires egui-baseview 0.7.1 (that refresh fixes embedded editors on
  macOS, which scaled instead of reflowing). 0.7.1's `src/` is identical to 0.7.0's; only the
  baseview requirement (0.3.1 → 0.3.4), examples and packaging metadata changed. Step 2: the event
  adapter still ignores the four drag variants, so the `MXM PATCH` hunks were reapplied; they applied
  unchanged. baseview 0.3.4 keeps the drag variants and `DropData::Files`. Checked:
  `cargo check --no-default-features --features tracing,wgpu` on Windows, no warnings; every editor
  consumer is built with the rest of the refresh.
