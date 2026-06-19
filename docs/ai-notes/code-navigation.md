# Code Navigation Notes

## Entry Points

- `src/SumatraStartup.cpp`: `WinMain`, process initialization, flag handling, tool modes, installer modes, startup restore/open logic.
- `src/Flags.h` and `src/Flags.cpp`: command-line state and parser.
- `src/SumatraPDF.cpp`: central app coordination, document opening/closing, frame commands, window procedure, tab/window operations, menus, notifications, read-aloud helpers.
- `src/Canvas.cpp`: canvas painting, input handling, hit testing, drag/drop, and canvas-mode-specific message handling.

## State Objects

- `src/MainWindow.h`: top-level window state and Win32 child handles.
- `src/WindowTab.h`: per-tab document state.
- `src/DocController.h`: UI-facing document controller interface.
- `src/DisplayModel.h` and `src/DisplayModel.cpp`: fixed-page document model, layout, zoom, scrolling, rendering requests, coordinate conversion.
- `src/FileHistory.*`: file state and session history.
- `src/Settings.h` and `src/Settings.cpp`: generated settings structs and serialization support.
- `src/AppSettings.*` and `src/GlobalPrefs.h`: application settings integration and global preferences.

## Document Engines

- `src/EngineBase.h` and `src/EngineBase.cpp`: common engine interface and shared behavior.
- `src/EngineCreate.cpp`: engine selection and supported file type checks.
- `src/EngineMupdf.h` and `src/EngineMupdf.cpp`: MuPDF-backed formats, PDF-specific features, annotations, text extraction, links, outlines, rendering.
- `src/EngineImages.cpp`: image files, image folders, comic archives.
- `src/EngineDjVu.cpp`: DjVu support.
- `src/EngineEbook.cpp`, `src/EbookDoc.*`, `src/MobiDoc.*`, `src/ChmModel.*`, `src/ChmFile.*`: ebook and CHM support.

## Rendering And Paint

- `src/RenderCache.h` and `src/RenderCache.cpp`: page render queue, worker threads, bitmap/tile cache, cache invalidation, painting cached pages.
- `src/DisplayModel.*`: decides page geometry and visible pages before asking `RenderCache` for output.
- `src/Canvas.cpp`: consumes model/cache state to paint and handle user interaction.

## Commands, Menus, And Input

- `cmd/gen-commands.ts`: command definitions and user-facing command names.
- `src/Commands.h` and `src/Commands.cpp`: generated command IDs and metadata.
- `src/Menu.*`: menu construction and command availability.
- `src/Accelerators.*`: keyboard shortcuts.
- `src/CommandPalette.*`: command palette UI and search.
- `src/Toolbar.*`: toolbar, search box, page box, and menu bar controls.

## Text, Search, Links, And Selection

- `src/TextSelection.*`: text selection state and operations.
- `src/TextSearch.*`: search implementation and search threading.
- `src/Selection.*`: selection helpers and copy behavior.
- `src/PdfSync.*`: SyncTeX/inverse-search support.
- `src/RefHover.*`: hover preview/reference UI.
- `src/ExternalViewers.*`: external viewer integration.

## Annotations And PDF Tools

- `src/Annotation.*`: annotation model wrappers.
- `src/EditAnnotations.*`: annotation editing UI.
- `src/PdfTools.*`: PDF manipulation dialogs and MuPDF command wrappers.
- `src/PdfCreator.*`: PDF creation helpers.

## UI Infrastructure

- `src/wingui/Wnd.cpp` and `src/wingui/WinGui.h`: base Win32 wrapper, subclassing, helper controls.
- `src/wingui/Layout.*`: layout primitives.
- `src/wingui/TabsCtrl.*`: custom tab control.
- `src/wingui/WebView.*` and `src/wingui/HtmlWindow.*`: WebView/HTML embedding.
- `src/mui`: custom UI/text rendering pieces.
- `src/uia`: UI Automation providers.

## Utilities

- `src/utils/BaseUtil.h`: common platform/compiler includes, type aliases, macros, assertions, and base dependencies.
- `src/utils/Vec.h`: local small-buffer vector used widely instead of STL containers.
- `src/utils/StrFormat.*`, `src/utils/StrUtil.*`, `src/common/str_util.cpp`: string helpers.
- `src/utils/FileUtil.*`, `src/common/file_util.cpp`: file/path helpers.
- `src/utils/WinUtil.*`, `src/common/win_util.cpp`: Win32 utility functions.
- `src/utils/ThreadUtil.*`, `src/utils/UITask.*`: threading and UI-thread task dispatch.
- `src/utils/ScopedWin.h`: RAII wrappers for Windows/GDI/GDI+ resources.

