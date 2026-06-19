# SumatraPDF Project Notes

These notes are a compact working map for future code-reading and patching sessions. They are not user-facing documentation; the public docs live under `docs/md`.

SumatraPDF is a native Windows C++ application built mostly on Win32 APIs. It is a multi-format document reader for PDF, EPUB, MOBI, CBZ/CBR, FB2, CHM, XPS, DjVu, images, and related formats. The core source is in `src`, third-party code is in `ext` and `mupdf`, build automation is in `cmd`, Visual Studio project files are generated into `vs2022`, and issue regression tests live in `tests`.

The project intentionally avoids most STL-style application code and instead uses local utility types and helpers from `src/utils` and `src/common`. Common examples are `Vec<T>`, `StrVec`, `TempStr`, arena allocation, Win32 wrappers, custom string helpers, file/path helpers, and UI task dispatch.

## High-Level Shape

- `src/SumatraStartup.cpp` owns application startup, command-line flag handling, tool modes, initialization, and `WinMain`.
- `src/SumatraPDF.cpp` is the large central coordinator for frames, tabs, document loading, menus, commands, UI actions, and the main frame window procedure.
- `src/Canvas.cpp` owns the document canvas window procedure and most mouse/paint interaction for fixed-page, CHM, About, and error canvas modes.
- `src/MainWindow.h` describes one top-level SumatraPDF window and its Win32 child controls, sidebars, toolbars, tabs, selection state, and per-window UI state.
- `src/WindowTab.h` describes one loaded tab, including file path, controller, ToC state, selection, watcher, annotation state, and per-document colors.
- `src/DocController.h` is the shared document-control interface used by the UI for navigation, zoom, display mode, ToC, thumbnails, and document state.
- `src/DisplayModel.*` implements `DocController` for fixed-page documents and is the main model for page layout, scrolling, zooming, rendering requests, text selection, and page coordinate conversion.
- `src/EngineBase.h` defines the rendering/document abstraction implemented by PDF/MuPDF, DjVu, image, ebook, CHM, and other engines.
- `src/RenderCache.*` manages asynchronous page rendering, tiling, bitmap cache entries, render requests, and render worker threads.
- `src/wingui` contains lightweight Win32 UI wrappers and controls such as `Wnd`, tabs, layout, buttons, splitters, edit controls, tooltips, and WebView support.
- `src/utils` and `src/common` are the local foundation libraries; prefer them over introducing new generic helpers.

## Build And Test Defaults

For normal development, build with:

```powershell
bun ./cmd/build.ts
```

This updates `out/dbg64/SumatraPDF-dll.exe`. Use that executable for testing; `out/dbg64/SumatraPDF.exe` is a different static target and can be stale.

For ad-hoc manual testing, pass `-for-testing` so the app starts isolated, does not restore the prior session, and does not write normal settings:

```powershell
./out/dbg64/SumatraPDF-dll.exe -for-testing <file>
```

For debugger launch:

```powershell
windbgx -Q -o -g ./out/dbg64/SumatraPDF-dll.exe
```

See `workflows.md` for more command details.

For a step-by-step build checklist and the direct MSBuild fallback, see `build-instructions.md`.
