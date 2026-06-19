# Build Instructions

Use these steps when the goal is to build the development executable for local testing.

## Preferred Command

From the repository root:

```powershell
bun ./cmd/build.ts
```

This script builds the Visual Studio solution target `SumatraPDF-dll` with configuration `Debug|x64`.

The expected output is:

```text
out/dbg64/SumatraPDF-dll.exe
```

Use `SumatraPDF-dll.exe` for testing. Do not rely on `out/dbg64/SumatraPDF.exe`; it is a different static target and `cmd/build.ts` does not update it.

## What The Script Does

`cmd/build.ts`:

- Selects the `SumatraPDF-dll` MSBuild target.
- Uses `vs2022/SumatraPDF.sln`.
- Builds with `/p:Configuration=Debug;Platform=x64`.
- Enables parallel MSBuild with `/m`.

The equivalent MSBuild shape is:

```powershell
MSBuild.exe vs2022\SumatraPDF.sln /t:SumatraPDF-dll /p:Configuration=Debug`;Platform=x64 /m
```

The semicolon in `Debug;Platform=x64` must be escaped as ``` `; ``` when running directly in PowerShell.

## Fallback When Bun Is Not In PATH

If `bun` is not available but Visual Studio is installed, locate MSBuild and run the same target directly. A known VS 2022 Community path is:

```powershell
& "C:\Program Files\Microsoft Visual Studio\2022\Community\MSBuild\Current\Bin\MSBuild.exe" "vs2022\SumatraPDF.sln" /t:SumatraPDF-dll /p:Configuration=Debug`;Platform=x64 /m
```

This fallback was verified in this workspace and produced:

```text
out/dbg64/SumatraPDF-dll.exe
```

## Manual Smoke Launch

For ad-hoc manual testing, always use `-for-testing`:

```powershell
./out/dbg64/SumatraPDF-dll.exe -for-testing <file>
```

This starts an isolated instance, does not restore the previous session, and does not write normal user settings.

## Debug Launch

```powershell
windbgx -Q -o -g ./out/dbg64/SumatraPDF-dll.exe
```

## Common Build Pitfalls

- If PowerShell says `bun` is not recognized, use the direct MSBuild fallback above or add Bun to PATH.
- If MSBuild fails in a sandbox with `Microsoft.Build.Utilities.FileTracker` and `E_ACCESSDENIED`, rerun the same MSBuild command outside the sandbox or with elevated permission.
- If source files were added or removed, regenerate the Visual Studio projects first with `bun ./cmd/premake.ts`.
- After changing `.cpp`, `.c`, or `.h` files under `src`, run clang-format on those touched files before building.

