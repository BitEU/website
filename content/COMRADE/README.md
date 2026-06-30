# COMRADE

**C**onnection **O**bject **M**apping **A**nd **R**elational **A**ssessment **D**atabase **E**ngine — a desktop investigator board for mapping people, notes, and the relationships between them. An analyst places **cards** (Person / Note / Legend) on an infinite, zoomable canvas and draws **labeled, directional relationships** between any of them. It is offline and single-user: everything lives in a local `.comrade` file, with no network access. Built with Rust + Tauri 2 + Svelte 5 and [Svelte Flow](https://svelteflow.dev) for the canvas.

## Screenshot

<!-- Drop a screenshot or animated GIF of the current UI here (STANDARDS §5.3). -->

## Install / run

Prerequisites: a [Rust toolchain](https://rustup.rs), [Node.js](https://nodejs.org) 20+,
and the [Tauri prerequisites](https://tauri.app/start/prerequisites/) for your OS
(on Linux, the WebKitGTK dev packages; on Windows/macOS the system webview is built in).

```powershell
npm install          # once, to pull frontend deps
npm run tauri dev    # run the app with hot-reload
```

`npm run tauri dev` starts the Vite dev server and the Rust shell together; edits
to Svelte/CSS hot-reload, edits to Rust trigger a recompile.

## Features

- **Cards:** Person (multiple labeled phones / addresses / emails, structured identifiers — SSN / DMV / FBI / NY — plus DOB, aliases, free-text *extra*, and arbitrary custom fields), Note (title + multi-line body), and Legend (a category → meaning key). Multiple legends allowed.
- **Relationships:** edges are first-class records between any two cards, with an optional label and a style — per-end arrowheads, solid / dashed / dotted lines, thickness, curve, and an optional category color.
- **Canvas:** infinite pan / zoom (wide range), minimap, background grid, box- and shift-select, and group move — all native to Svelte Flow.
- **Detail levels:** Full / Essential / Minimal control how much each card renders, so a data-rich board stays legible. Global and persisted.
- **Inspector:** a right-docked, modeless panel edits the selected card or edge live (no modal dialogs), with inline non-blocking validation.
- **Search & dedup:** the header search matches across *every* field of every card and dims non-matches; adding a Person whose name resembles an existing one shows an inline duplicate warning (fuzzy, Jaro–Winkler).
- **Relationship focus:** a toggle opens a panel of the selected card's connected component and traces the shortest path to any member, dimming everything else.
- **Attachments:** attach files to a card (picker or drag-and-drop); the first image renders as a thumbnail. Bytes live on disk, never in the document, so undo and IPC stay small.
- **Undo / redo, clipboard:** full undo history; in-session copy / cut / paste (internal relationships preserved) and duplicate.
- **Export:** the board to a high-resolution **PNG** or a one-page **PDF**.
- **Persistence:** a native `.comrade` bundle (a zip of `document.json` + an `assets/` tree), written atomically; a single rolling crash-recovery autosave offers to restore unsaved work on the next launch.
- **Backward compatibility:** one-way **Convert** of legacy COMRADE files (the old `.zip`-with-`data.csv`, and the bare people-only `.csv`) into a `.comrade` bundle, attachments and all.

### Supported file formats

| Format | Read | Write | Notes |
|--------|------|-------|-------|
| `.comrade` bundle | ✓ | ✓ | native format (zip: `manifest.json` + `document.json` + `assets/`) |
| Legacy COMRADE `.zip` (`data.csv` + `files/`) | ✓ (Convert) | — | one-way import → saved as `.comrade` |
| Legacy bare `.csv` (people-only) | ✓ (Convert) | — | one-way import → saved as `.comrade` |
| PNG / PDF | — | ✓ (export) | board image / one-page document |

## Keyboard shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+S` | Save |
| `Ctrl+F` | Find / focus search |
| `Ctrl+Z` | Undo |
| `Ctrl+Shift+Z` / `Ctrl+Y` | Redo |
| `Ctrl+C` / `Ctrl+X` / `Ctrl+V` | Copy / Cut / Paste (at the cursor) |
| `Ctrl+D` | Duplicate selection |
| `Delete` / `Backspace` | Remove selection (with confirmation) |
| `1` / `2` / `3` | Detail level: Full / Essential / Minimal |
| `F1` | Keyboard shortcuts help |
| `Esc` | Deselect / close panel or dialog |

Edges are created by dragging from a card's side handle to another card (drop on empty canvas to quick-connect a new card). Right-click a node, edge, or the canvas for a context menu.

## Architecture

A Cargo workspace of two Rust crates plus a Svelte frontend. `comrade-core` holds all
logic and depends on no UI or Tauri code (it is testable headless). `src-tauri` is a thin
Tauri shell that wires `#[tauri::command]`s to the core and owns the open document. `frontend/`
is the Svelte 5 + TypeScript app, which talks to the shell only through typed IPC wrappers and
treats its own state as a projection of what the core knows. `Cargo.toml` and `package.json`
stay at the root (the CLIs resolve their code dirs from there).

```
comrade/
├── Cargo.toml            # Rust workspace
├── package.json          # frontend deps + scripts (dev → vite frontend)
├── comrade-core/         # all logic; no UI/Tauri deps; testable headless
│   ├── src/
│   │   ├── model/        # Document, Node, Person/Note/Legend, Edge, Attachment, Category
│   │   ├── parser/       # legacy importers (old zip, bare csv)
│   │   ├── service/      # document, import, search, graph, history, assets,
│   │   │                 #   recovery, png/pdf export, config, logging, task
│   │   ├── app.rs        # identity constants (COMRADE / IcebergForensics)
│   │   └── error.rs      # crate Error enum (thiserror)
│   └── tests/fixtures/   # real input samples for the importer/round-trip tests
├── src-tauri/            # Tauri shell (Rust)
│   ├── src/{main.rs,lib.rs,commands.rs,dto.rs,state.rs,error.rs}
│   ├── capabilities/     # least-privilege permission grants
│   ├── icons/            # generated by `tauri icon`
│   └── tauri.conf.json
├── frontend/             # Svelte frontend (Vite root)
│   ├── index.html, vite.config.ts, svelte.config.js, tsconfig.json
│   └── src/
│       ├── App.svelte, main.ts
│       ├── styles/{theme.css,global.css}      # design tokens
│       └── lib/
│           ├── ipc.ts, types.ts, clipboard.svelte.ts, boardActions.ts, theme.ts
│           ├── stores/         # board / view / session / search / focus / attachments
│           ├── nodes/, edges/  # Svelte Flow card + edge components
│           ├── views/, components/
│           └── assets/         # fonts, license texts
├── .github/workflows/    # CI: fmt + clippy + test + svelte-check + bundle, 3 OSes
├── docs/
├── analysis_todo.md      # in-repo backlog (STANDARDS §5.4)
└── STANDARDS.md          # this workspace's conventions
```

## Build

```powershell
npm run tauri build   # release bundle
npm run check         # svelte-check (frontend type safety)
cargo test            # core integration + unit tests
```

`npm run tauri build` produces the platform bundle under `src-tauri/target/release/bundle/`
(`.msi`/`.exe` on Windows, `.dmg`/`.app` on macOS, `.deb`/`.AppImage` on Linux). CI builds and
tests for Windows 11 x64, Linux x86_64, and macOS Apple Silicon on every push and PR — checking
formatting, clippy (warnings denied), `cargo test`, `svelte-check`, and a full bundle build.

## License

Proprietary, all rights reserved. See [LICENSE.txt](./LICENSE.txt). Copyright held by
Steven R. Schiavone. Internal-use grants are extended to Iceberg Forensics LLC and the
Westchester County District Attorney's Office High Tech Crime Bureau, each for their own
forensic work (no redistribution or sublicensing).

Bundled third-party components ship under their respective upstream licenses: the Inter
font (SIL Open Font License 1.1), Bootstrap Icons (MIT), and the Tauri / Svelte / Vite
toolchain (MIT or Apache-2.0). The About panel surfaces the full license texts.

---

This project follows [STANDARDS.md](./STANDARDS.md).
