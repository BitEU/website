# NPPListView

A Notepad++ plugin for reading the `.plist` files that show up in iOS
extractions, including the Snapchat `TSAF` files that other bplist plugins
reject. You can view/edit the plist in either the editing window or in the 
dockable 3-panel outline window.

It's a fork of [azerg/NppBplistPlugin](https://github.com/azerg/NppBplistPlugin)
(BSD-3) with some signifigant changes:

1. The original plugin pulled bytes back out of Notepad++'s text buffer, after 
   Notepad++ had already run them through encoding detection, and corrupted valid 
   binary plists.
2. It decodes Snapchat `TSAF` files across several versions.

## File types

| Type | Detected by | Shown as | Editable |
|------|-------------|----------|----------|
| Apple binary plist | `bplist00` magic | XML, via libplist | Yes; saves back to a valid bplist |
| Snapchat TSAF | `TSAF` magic | decoded tree | No, read-only |
| Apple XML plist | `<?xml` | unchanged | Yes, normal Notepad++ |

## Why TSAF is read-only

TSAF is Snapchat's own serialization format, not an Apple property list, and
it's undocumented. The decoder handles the values it can read without ambiguity
(strings, string references, ints, floats, doubles, data blobs, class/dict/array
markers, Unix timestamps). Anything it can't be sure of, it prints as a raw
opcode with its byte offset, like `<op 0xNN @123>`, rather than guessing.

[This was a really helpful article for solving this!](https://doubleblak.com/blogPost.php?k=snapchat)

## Menu commands

- **Show Plist Outline** — toggle the editable outline panel described above.
- **Show file format / decode info** — what the plugin recognized the current
  file as.
- **Export decoded view to file...** — saves the decoded text to a new file.
- **About**

## Install (64-bit Notepad++)

1. Build it (below) or grab `NppListView.dll`.
2. Put it in a folder of the same name under Notepad++'s `plugins` directory:

   ```
   <Notepad++>\plugins\NppListView\NppListView.dll
   ```

   `<Notepad++>` is usually `C:\Program Files\Notepad++`, or the folder next to
   `notepad++.exe` for a portable install.
3. Delete any old bplist plugin you had installed, so the two don't both try to
   convert the same buffer.
4. Restart Notepad++ and open a `.plist`. Binary plists and TSAF files render
   automatically.

The DLL is x64 and won't load in a 32-bit Notepad++.

## Build

You need Visual Studio 2022 with the "Desktop development with C++" workload,
CMake 3.20 or newer, and vcpkg (bundled with VS).

PowerShell:

```powershell
$env:VCPKG_ROOT = "C:\Program Files\Microsoft Visual Studio\2022\Community\VC\vcpkg"
cmake -S . -B build -G "Visual Studio 17 2022" -A x64 `
  -DCMAKE_TOOLCHAIN_FILE="$env:VCPKG_ROOT\scripts\buildsystems\vcpkg.cmake" `
  -DVCPKG_TARGET_TRIPLET=x64-windows-static -DTARGET_TRIPLET=x64-windows-static
cmake --build build --config RelWithDebInfo
```

Command Prompt (`cmd.exe`):

```bat
set "VCPKG_ROOT=C:\Program Files\Microsoft Visual Studio\2022\Community\VC\vcpkg"
cmake -S . -B build -G "Visual Studio 17 2022" -A x64 ^
  -DCMAKE_TOOLCHAIN_FILE="%VCPKG_ROOT%\scripts\buildsystems\vcpkg.cmake" ^
  -DVCPKG_TARGET_TRIPLET=x64-windows-static -DTARGET_TRIPLET=x64-windows-static
cmake --build build --config RelWithDebInfo
```

The build writes `bin\RelWithDebInfo\NppListView.dll`.

## Layout

```
src/NppListView/src/
  BplistMngr.cpp/.h    disk read, format detection, buffer handling, export,
                       and the bridge between the editor and the outline panel
  PlistMngr.cpp/.h     libplist bplist <-> XML wrapper (unchanged)
  SafeBplist.cpp/.h    crash-proof bplist00 -> XML decoder
  TsafDecoder.cpp/.h   self-contained TSAF decoder, no dependencies
  PlistModel.cpp/.h    editable tree model; XML <-> model conversion
  PlistTreePanel.cpp/.h dockable three-column outline (Header + TreeView, custom
                       draw) and all in-place editing
  Docking.h            minimal Notepad++ docking SDK (tTbData, DWS_*/DMN_*)
  PluginDefinition.*   plugin name and menu commands
  NppPluginMain.cpp    Notepad++ notification dispatch
```