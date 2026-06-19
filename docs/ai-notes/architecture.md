# Architecture Notes

## Startup And Process Modes

`src/SumatraStartup.cpp` contains `WinMain`. Startup initializes process-level services, parses flags through `ParseFlags()`, handles installer/uninstaller/tool/headless modes, registers window classes, creates global state such as the render cache, and eventually opens or restores windows/tabs.

Command-line state is represented by `Flags` in `src/Flags.h` and parsed in `src/Flags.cpp`. Generated flag declarations come from `cmd/gen-flags.ts`. Several regression tests use dedicated headless flags such as `-test-synctex`, `-test-search`, `-test-dest`, `-test-named-dest`, and `-test-chm`; these are preferred when adding machine-readable tests.

## Window, Tab, And Controller Model

`MainWindow` represents a top-level frame window. It owns or references Win32 child controls, toolbar/menu/sidebar state, tab control state, caption buttons, selection state, annotation drag state, and current mouse action. It points at the current tab and keeps `ctrl` as a convenience alias for `CurrentTab()->ctrl`.

`WindowTab` holds document-specific state that can conceptually move between windows: file path, display name, `DocController`, ToC state, file watcher, selection rectangles, previous zoom/display mode, annotation state, lazy-loaded tab state, per-document colors, and read-aloud resume data.

`DocController` is the UI-facing abstraction for a loaded document. The UI can ask it for page count, current page, navigation, zoom, display mode, ToC, named destinations, display state, and thumbnails. `DisplayModel` implements it for fixed-page documents; `ChmModel` implements it for CHM/HTML-style content.

The UI callback direction is modeled by `DocControllerCallback`. Controllers use it to request repainting, scrollbar updates, page render requests, thumbnail rendering, cleanup, focus handoff, and link/download handling.

## Fixed-Page Documents

`DisplayModel` is the central fixed-page document model. It owns the `EngineBase`, an array of `PageInfo`, navigation history, current display mode, zoom, rotation, viewport, text selection/search helpers, and PDF synchronization state.

Important responsibilities:

- Build and relayout page geometry after zoom, rotation, display mode, or viewport changes.
- Convert between screen coordinates and page/user coordinates.
- Track visible pages and visible page regions.
- Dispatch rendering through `RenderCache`.
- Preserve and restore display state through `FileState`.
- Implement navigation, page labels, ToC links, named destinations, and common shortcuts.

`PageInfo` separates stable page data (`mediaBox`), derived content data (`contentBox`), current display rectangle, visible ratio, and render failure state.

## Engines

`EngineBase` is the document/rendering abstraction. Engine implementations load files/streams, expose page count and page boxes, render pages to bitmaps, extract text, expose links/elements, provide metadata, handle ToC and named destinations, and optionally support annotations.

Main engine entry points:

- `EngineMupdf.*` handles PDF, XPS, CBZ-like MuPDF formats, PDF annotations, attachments, outlines, named destinations, page labels, and MuPDF rendering.
- `EngineDjVu.cpp` handles DjVu.
- `EngineImages.cpp` handles images, image directories, and comic book archives.
- `EngineEbook.cpp`, `EbookDoc.*`, `MobiDoc.*`, and `ChmModel.*` handle ebook and CHM-style formats.
- `EngineCreate.cpp` selects the appropriate engine from file type and feature flags.

`EngineMupdf` has an explicit lock hierarchy documented in `src/EngineMupdf.h`: `pagesLock`, `renderLock`, then `docLock`, with MuPDF's own short-lived `fz_locks` separate from those longer-held locks. Keep that hierarchy intact when touching rendering, page loading, annotations, or document metadata.

## Rendering

`RenderCache` owns bitmap cache entries, page render requests, render threads, and tile management. Display code asks it to render visible pages and paint cached output. Large pages can be split into tiles, and render worker threads are spawned lazily up to `gMaxRenderThreads`.

The cache key is effectively display model, page number, rotation, zoom, and tile. `DisplayModel` tells the cache which pages are visible or nearby, and the cache can abort stale work, drop entries, invalidate pages, and keep cache entries when display models are replaced.

## UI

The application is mostly Win32 UI. The main frame window procedure is `WndProcSumatraFrame()` in `src/SumatraPDF.cpp`. The canvas window procedure is `WndProcCanvas()` in `src/Canvas.cpp`, with sub-handlers for fixed-page, CHM, About, and load-error modes.

`src/wingui/Wnd.cpp` provides a small object-oriented wrapper around Win32 window procedures and subclassing. Other `wingui` files provide local controls and layout helpers. Prefer these wrappers and existing control patterns before adding new UI infrastructure.

