# Toast

A high-performance native desktop application for visualizing call detail record (CDR) relationships. Built with C++ and Qt6 for maximum performance when handling thousands of phone numbers and connections.

## Features

- **Load and parse CDR data** from CSV files (600 to 18,000+ records)
- **Interactive graph visualization** with hundreds of phone number nodes
- **Force-directed layout algorithm** that prevents node overlap
- **Color-coded connection lines** for easy differentiation (dark gray text)
- **Double-click nodes** to select and highlight connections
- **Right double-click nodes** to add custom labels (people's names)
- **Customize node colors** with keyboard shortcut (press 'c' to cycle)
- **Connection details**: Shows call count, inbound/outbound breakdown, and average duration on edges
- **Save/Load projects** in `.TOAST` (binary archive format) including positions and colors
- **Export high-resolution images** at 300 DPI for printing/documentation
- **Community detection** automatically groups related phone numbers
- **About dialog** with Qt license compliance information

## Architecture

### Key Components

- **CDRParser**: Parses CSV files and builds connection graphs
- **PhoneNode**: Visual representation of phone numbers with custom labels
- **CallEdge**: Connections showing call statistics between numbers
- **ForceLayout**: Physics-based layout engine preventing overlap
- **GraphScene**: Manages the visualization and interactions
- **ToastArchive**: Binary file format for saving complete projects

### Why Qt/C++?

- Native performance for handling large datasets
- Efficient QGraphicsView for complex visualizations
- Direct memory control for graph algorithms
- No overhead from web technologies
- Professional desktop application experience

## Building

### Prerequisites

- CMake 3.16 or higher
- Qt 6.x (Core, Widgets, Gui modules)
- C++17 compatible compiler (MSVC, GCC, or Clang)

### Windows Build

1. Download Qt Online Installer from https://www.qt.io/download-qt-installer
2. Install Qt 6.10.x with **MinGW 64-bit** component (NOT MSVC)
3. Build:
```powershell
# Use the included build script (automatically deploys Qt dependencies)
.\build.bat "C:\Qt\6.10.1\mingw_64"

# Run the application
cd build
.\Toast.exe
```

And when quickly iterating:
1. ```cd .. ; .\build.bat "C:\Qt\6.10.1\mingw_64"```
2. ```cd build ; .\Toast.exe```

## Usage

### 1. Load CDR Data
Click **Load CSV** and select your CDR file. The CSV should have the format:
```csv
Target Number,Call Direction,From or To Number,Date,Start,End
+12177259600,Outbound,+16757881466,2024-11-08,16:45:00,16:59:18
```

### 2. Run Layout
Click **Run Layout** to automatically organize nodes using force-directed physics. The algorithm:
- Repels overlapping nodes
- Attracts connected nodes
- Prevents overlap with collision detection

Click **Stop Layout** when satisfied with the arrangement.

### 3. Navigate
- **Pan**: Click and drag the background
- **Zoom In/Out**: Use zoom buttons
- **Reset Zoom**: Return to default view
- **Select Node**: Double left-click to select/deselect and highlight connections
- **Cycle Node Color**: Press 'c' key while node is selected to change its color

#### Mouse Wheel / Middle Button Behaviors
- **Scroll wheel**: Scrolls vertically.
- **Scroll wheel + CTRL:** Zooms in/out (same behavior as the Zoom In/Zoom Out buttons up top).
- **Scroll wheel + SHIFT:** Scrolls horizontally.

These behaviors help quickly navigate large graphs: hold middle and Ctrl to zoom, or hold middle and Shift to pan left/right using the wheel.

### 4. Add Labels
Double **right-click** any phone number box to add a custom label (e.g., person's name). Double left-click selects/deselects the node and highlights its connections.

### 5. Save Project
Click **Save Project (.TOAST)** to save:
- All CDR records
- Custom labels
- Graph state

The `.TOAST` format uses binary serialization with a magic number header for verification.

### 6. Export Image
Click **Export Image (300 DPI)** to save a high-resolution PNG or JPEG. Large graphs may produce files of 20-50 MB or more.

## File Format: .TOAST

The TOAST (Telecommunications Operations Archive Storage) format stores:

```
Header:
- Magic Number: 0x544F4153 ("TOAS")
- Version: 1

Data Blocks:
1. CDR Records (count + serialized records)
2. Node Labels (count + key-value pairs)
```

All data is stored in Qt's QDataStream format for cross-platform compatibility.

## Performance

- Handles **18,000+ CDR records**
- Visualizes **hundreds of phone numbers** simultaneously
- Force layout stabilizes in **~500 iterations** (8-10 seconds)
- 300 DPI exports scale to **50,000x50,000 pixels** or more
- Efficient collision detection and overlap prevention

## Color Scheme

- **Node boxes**: Light blue background (#F0F5FA) with gray border
- **Connection lines**: Hash-based color generation for consistency and variety
- **All text**: Dark gray (#404040) for readability
- **Line thickness**: Scales with call count (1-6 pixels)
- Nodes can be assigned one of 22 colors using the 'c' key (cycle through palette):
    - Red #e6194B
    - Green #3cb44b
    - Yellow #ffe119
    - Blue #4363d8
    - Orange #f58231
    - Purple #911eb4
    - Cyan #42d4f4
    - Magenta #f032e6
    - Lime #bfef45
    - Pink #fabed4
    - Teal #469990
    - Lavender #dcbeff
    - Brown #9A6324
    - Beige #fffac8
    - Maroon #800000
    - Mint #aaffc3
    - Olive #808000
    - Apricot #ffd8b1
    - Navy #000075
    - Grey #a9a9a9
    - White #ffffff
    - Black #000000

## License

Copyright © 2026. All rights reserved.

## Qt License Compliance

This project links against Qt 6 (see `CMakeLists.txt` — `Qt6::Core`, `Qt6::Widgets`, `Qt6::Gui`). The included `build.bat` and the recommended Windows build process use the official Qt binary distribution and `windeployqt`, which deploys Qt as shared libraries (dynamic linking).