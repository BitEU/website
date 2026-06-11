# **FENDER \- Forensic Extraction of Navigational Data & Event Records**

FENDER is a powerful tool for extracting GPS location data from vehicle telematics binary files. It supports multiple vehicle manufacturers and provides an easy-to-use interface for forensic investigators and researchers.

## Feedback
Your input helps make FENDER better! Please share bugs, feature requests, or suggestions by [creating a GitHub Issue](https://github.com/BitEU/FENDER/issues/new). Include the diagnostic details in your test file, as well as your logs/fender.log file.

I would also appreciate any assistance with analyzing QNX systems, as all of my attempts have failed thusfar. If you have experience in this, please contact me and we can work on adding it to FENDER!

## **Table of Contents**

* [Feedback](#feedback)
* [Simple Guide](#simple-guide)  
  * [Quick Start](#quick-start)  
    * [Windows Users](#windows-users)  
    * [Python Users](#python-users)  
  * [Supported Vehicles](#supported-vehicles)  
  * [Features](#features)  
  * [Command-Line Interface (CLI)](#command-line-interface-cli)
  * [System Requirements](#system-requirements)  
* [Advanced Guide](#advanced-guide)  
  * [Architecture Overview](#architecture-overview)  
    * [Core Components](#core-components)  
  * [Module Structure](#module-structure)
  * [Technical Details](#technical-details)  
    * [Plugin Architecture](#plugin-architecture)  
  * [Decoder Specifications](#decoder-specifications)  
    * [OnStar Decoder](#onstar-decoder)  
    * [Toyota Decoder](#toyota-decoder)  
    * [Honda Decoder](#honda-decoder)
    * [Honda TLHOBINN0D1 Decoder](#honda-tlhobinn0d1-decoder)
    * [Mercedes-Benz Decoder](#mercedes-benz-decoder)
    * [BMW NBT-HDD Decoder](#bmw-nbt-hdd-decoder)
    * [Stellantis Decoder](#stellantis-decoder)
    * [Denso Decoder](#denso-decoder)
    * [Kia Dealer Mode Decoder](#kia-dealer-mode-decoder)
  * [Installation from Source](#installation-from-source)  
    * [Windows](#windows)  
    * [Linux/macOS](#linuxmacos)  
  * [Building Executables](#building-executables)  
    * [Windows Executable with PyInstaller](#windows-executable-with-pyinstaller)  
  * [Decoder Development](#decoder-development)  
    * [Key Methods to Implement](#key-methods-to-implement)  
  * [Data Formats](#data-formats)  
    * [GPSEntry Structure](#gpsentry-structure)  
    * [XLSX Output Format](#xlsx-output-format)  
  * [Troubleshooting](#troubleshooting)  
    * [Common Issues](#common-issues)  
* [Todo](#todo)  
* [Scoreboard](#scoreboard)
* [Contributing](#contributing)  
* [Credits](#credits)

## **Simple Guide**

### **Quick Start**

#### **Windows Users**

1. Download the latest release from the releases page.  
2. Double-click FENDER.exe to run the application.  
3. Select your decoder type from the left panel.  
4. Drag and drop your binary file or click to browse.  
5. Click "Process File" to extract GPS data.  
6. Results will be saved as an XLSX file in the same directory.

#### **Python Users**

```bash
# Install dependencies  
pip install -r requirements.txt

# Run the GUI  
python main.py

# Run in CLI mode  
python main.py --cli
```

### **Supported Vehicles**

* **OnStar Gen 10+** \- Extracts GPS data from OnStar NAND dumps (.CE0 files)  
* **Toyota TL19** \- Extracts GPS data from Toyota infotainment systems (.CE0 files)  
* **Honda Telematics** \- Extracts GPS data from Honda Android eMMC images (.USER files)
* **Honda TLHOBINN0D1** \- Extracts GPS data from Honda telematics binary files (.CE0, .bin, .001 files)
* **Mercedes-Benz** \- Extracts GPS data directly from a raw Mercedes NTG5*2 head-unit image (.bin/.img/.dd/.001/.raw)
* **BMW NBT-HDD** \- Extracts GPS data from a raw BMW NBT-HDD image (.bin/.img/.dd/.001/.raw) **or** a manually-uploaded `trails.sqlite` / `trips.sqlite` (.sqlite/.db). Per-fix timestamps are reconstructed from the Path BLOB's TLV stream so each row carries its own UTC time (not just the trail BeginTime).
* **Stellantis** \- Extracts GPS data directly from a raw Stellantis vehicle eMMC/HDD image (.bin/.img/.dd/.001/.raw)
* **Denso DNNS087** \- Extracts GPS, speed, and bluetooth data from Denso and Acura Android eMMC images (.001 files)
* **Kia Dealer Mode** \- Extracts GPS data from Kia head unit dealer-mode log bundles (.tar.gz files)

### **Features**

* 🚗 Multi-manufacturer support with modular decoder architecture  
* 📍 Extracts latitude, longitude, and timestamps  
* 📊 Exports data to XLSX, JSON, and KML formats for analysis  
* 🖱️ Drag-and-drop file support  
* 💻 Both GUI and command-line interfaces  
* 🔌 Plugin architecture for easy decoder additions

### **Command-Line Interface (CLI)**

FENDER ships with an interactive CLI driven by `python main.py --cli` (or `FENDER.exe --cli` on the Windows release). Use it when:

* You're working on a headless system / over SSH where the GUI isn't an option
* You want to script extractions across multiple cases by piping stdin (see "Scripting" below)
* You're benchmarking a decoder change and want a quick repeatable harness

#### **Interactive walkthrough**

```bash
python main.py --cli
```

The CLI walks you through six prompts in order:

1. **Decoder** — numbered list (Acura Denso DNNS087 = 1, BMW NBT-HDD = 2, Honda TLHOBINN0D1 = 3, Honda Telematics = 4, Kia Dealer Mode = 5, Mercedes NTG5*2 = 6, OnStar Gen 10+ = 7, Stellantis Vehicles = 8, Toyota TL19 = 9). Order is alphabetical by decoder name, so it shifts when decoders are added.
2. **Export format** — `1` = XLSX, `2` = JSON, `3` = KML.
3. **Examiner name** — optional; embedded in the XLSX/JSON metadata sheet.
4. **Case number** — optional; embedded in the XLSX/JSON metadata sheet.
5. **Input path** — either:
   * A **raw image file** (`.bin`, `.img`, `.dd`, `.001`, `.raw`) for Stellantis / BMW NBT-HDD / Mercedes NTG5*2 — the decoder mounts every QNX6 partition inside and pulls the relevant log files or SQLite DBs (`/nav/trails.sqlite`, `/nav/trips.sqlite`, etc.) directly from the image. No need to know which partition holds them.
   * A **single binary file** for the other decoders (`.CE0`, `.USER`, `.bin`, etc.).
   * A **`.tar.gz` bundle** for Kia Dealer Mode.
6. **Filter duplicate entries?** — `y` collapses entries that agree on lat/lon (4-decimal precision), timestamp, and source.

Output filename is auto-generated as `<input-basename>_<decoder>_<YYYYMMDD_HHMMSS>.<ext>` next to the input file.

#### **Scripting**

The prompts read from stdin one line at a time, so you can pipe a here-doc to drive a non-interactive run. Example — extract Stellantis GPS from a Jeep eMMC image to XLSX with no duplicate filtering:

```bash
printf '8\n1\nYour Name\nCASE-1234\n/path/to/CFL-24-0170_EVD1_EMMC_ROM1.bin\nn\n' | python main.py --cli
```

The same shape works for BMW (decoder `2`) and Mercedes (decoder `6`) with their respective raw image paths.

#### **Note on Stellantis / BMW / Mercedes input**

These three decoders used to accept folders (Stellantis) or loose `.sqlite` files (BMW / Mercedes). They **no longer do** — they now accept only raw images and locate the relevant files inside the image's QNX6 partitions themselves. If you have a previously-extracted partition tree, run the QNX6 partitioner in reverse (or re-image the device) before feeding it to FENDER. The motivation is forensic provenance: the decoder reads bytes directly from the image and reports the internal POSIX path (e.g. `/persistentLogs/AASXMTC/Log1` or `/nav/trails.sqlite`) as the source rather than a host-filesystem path that's outside the chain of custody.

### **System Requirements**

* Windows 10/11 (for .exe release)  
* Python 3.8+ (for source code)  
* 4GB RAM minimum  
* 500MB free disk space

## **Advanced Guide**

### **Architecture Overview**

FENDER uses a modular plugin architecture that allows for easy addition of new decoder types:

```
FENDER/  
├── main.py                # Main application entry point
├── src/                   # Source code modules
│   ├── core/             # Core components
│   │   └── base_decoder.py    # Abstract base class  
│   ├── gui/              # GUI components  
│   │   └── main_window.py     # Main GUI application
│   ├── cli/              # CLI components
│   │   └── cli_interface.py   # Command-line interface
│   └── utils/            # Utility modules
│       ├── file_operations.py
│       └── system_info.py
├── decoders/             # Decoder plugins directory  
│   ├── __init__.py  
│   ├── onstar_decoder.py  
│   ├── toyota_decoder.py  
│   ├── honda_decoder.py  
│   ├── mercedes_decoder.py  
│   ├── bmw_decoder.py
│   ├── denso_decoder.py    
│   ├── stellantis_decoder.py  
│   └── kia_decoder.py
└── requirements.txt
```

#### **Core Components**

1. **DecoderRegistry**: Auto-discovers and manages available decoders  
2. **BaseDecoder**: Abstract class defining the decoder interface  
3. **VehicleGPSDecoder**: Main GUI application using tkinter  
4. **GPSEntry**: Standard data structure for GPS points

### **Module Structure**

This section provides detailed information about the modular architecture and components:

#### **1. `main.py`**
- **Purpose**: Main entry point for the application
- **Contents**: 
  - Logging setup
  - Command line argument parsing
  - Application initialization
  - Import and execution of GUI or CLI modes

#### **2. `main_window.py`**
- **Purpose**: GUI components and user interface
- **Contents**:
  - `VehicleGPSDecoder` class - Main GUI application
  - `CustomRadiobutton` and `CustomToggleButton` classes - Custom UI widgets
  - GUI setup, styling, event handling
  - File processing workflow for GUI mode
  - Drag-and-drop functionality
  - Progress reporting and error handling

#### **3. `cli_interface.py`**
- **Purpose**: Command-line interface logic
- **Contents**:
  - `DecoderRegistry` class - Manages available decoders
  - `run_cli()` function - Main CLI workflow
  - User interaction for decoder/format selection
  - CLI-specific processing and output
  - Helper functions for CLI operation

#### **4. `file_operations.py`**
- **Purpose**: File handling and export operations
- **Contents**:
  - File validation and security functions
  - Export format writers (Excel, JSON, KML)
  - Secure file operations (temp files, copying, etc.)
  - Duplicate entry filtering
  - File path sanitization and validation

#### **5. `system_info.py`**
- **Purpose**: System information gathering
- **Contents**:
  - Hardware and OS information collection
  - Decoder integrity verification
  - Network connectivity checks
  - Permission validation
  - Extraction metadata generation

### **Technical Details**

#### **Plugin Architecture**

The application automatically discovers decoders at runtime:

1. Scans the `decoders/` directory for `*_decoder.py` files  
2. Imports modules and finds classes inheriting from BaseDecoder  
3. Registers decoders in the registry  
4. Makes them available in the GUI/CLI

### **Decoder Specifications**

#### **OnStar Decoder**

File Format: OnStar NAND dumps (.CE0 files)  
Data Location: GPS data stored as text within binary  
Extraction Method: Pattern matching for GPS keywords  
Key patterns:

- `gps_tow=` - GPS time of week (milliseconds)  
- `gps_week=` - GPS week number  
- `lat=` - Latitude in hex format  
- `lon=` - Longitude in hex format  
- `utc_year=`, `utc_month=`, etc. - UTC timestamp components

**Coordinate Format**:

* Stored as 16-byte hex strings  
* Decoded as little-endian doubles  
* Divided by 10,000,000 for decimal degrees

#### **Toyota Decoder**

File Format: Toyota TL19 NAND dumps (.CE0 files)  
Data Location: Structured binary format with markers  
Extraction Method: Binary pattern matching with offsets  
Key markers:

- `loc.position` - Base location marker  
- Various longitude markers (e.g., `ong6`, `ongi5`)  
- Latitude marker: `latitud,`  
- Multiple timestamp markers

**Data Structure**:

* Fixed offsets from markers  
* Timestamps stored as Unix milliseconds  
* Coordinates as ASCII strings in binary

#### **Honda Decoder**

File Format: Honda Android eMMC images (.USER files)  
Data Location: SQLite database in Android userdata partition  
Extraction Method: Filesystem extraction using pytsk3  
Process:

1. Find userdata partition (GPT or ext4)  
2. Extract filesystem using TSK  
3. Locate crm.db in Honda telematics app data  
4. Query eco\_logs table for GPS data

**Database Schema**:

- `start_pos_lat`, `start_pos_lon` - Starting coordinates  
- `finish_pos_lat`, `finish_pos_lon` - Ending coordinates  
- `start_pos_time`, `finish_pos_time` - Timestamps

#### **Mercedes-Benz Decoder**

File Format: Raw Mercedes NTG5*2 head-unit image (.bin/.img/.dd/.001/.raw)
Data Location: `/nav/trails.sqlite` and `/nav/trips.sqlite` inside any QNX6 partition of the image
Extraction Method: QNX6 partition mount → SQLite extraction → Trails/Trips schema detection → binary Path BLOB decode with Bounding-box filtering
Process:

1. Mount every QNX6 partition discovered in the raw image (the same image typically contains 5+ QNX6 partitions; we don't require the user to know which one holds `/nav/`)
2. Locate `/nav/trails.sqlite` and `/nav/trips.sqlite` via the union-view file source; extract each to a private temp directory
3. For each extracted DB, probe the schema: Mercedes' `trails.sqlite` carries a `Trails` table with `BeginTime`/`EndTime`/`Path`/`Bounding`; Mercedes' `trips.sqlite` carries a `Trips` table with `DepartureTime`/`ArrivalTime`/`Path`/`Bounding` (and sometimes a stub `Trails` table that the decoder safely skips)
4. Decode the segmented Path BLOB (event_id 1 = GPS coordinates, lon/lat/elev as 32-bit ints); a 24-byte Bounding BLOB filters out marker-byte false positives
5. Merge results from both DBs, dedupe, and emit rows tagged with their `SourceTable` (`/nav/trails.sqlite:Trails`, `/nav/trips.sqlite:Trips`)

**Data Format**:

* Coordinates encoded as int32 (`value * 180 / 2^31`); unsigned-to-signed conversion applied per Mercedes convention
* Begin/Departure and End/Arrival times stored as Unix epoch
* `Bounding` BLOB: 4-byte magic (`01 01 01 00`) + SW (lon, lat as i32, 4 pad bytes) + NE (lon, lat as i32, 4 pad bytes) = 24 bytes

#### **Stellantis Decoder**

File Format: Raw Stellantis vehicle eMMC/HDD image (.bin/.img/.dd/.001/.raw)
Data Location: Log files at fixed paths under various QNX6 partitions inside the image (e.g. `/persistentLogs/AASXMTC/Log*`)
Extraction Method: QNX6 partition mount → glob across union view → text-mode line scan with multi-pattern matching
Key patterns:

- `SAL_SDARS_FUEL` - Navigation destination coordinates
- `NW_SOS` - Emergency call position data
- `SAL_KONA_NAVI` - Navigation system coordinates
- `GetCurrentLocAddressResponse` - Location service responses
- `JSR179InterfaceImpl` - Low-level positioning with speed data
- `NaviTelematicsDataRequest` - Telematics position reports

**Extraction Process**:

1. Mount every QNX6 partition in the raw image (typically 15+ partitions on a Stellantis dump)
2. Glob for known log-file patterns (`**/pas_debug.log.*`, `**/persistentLogs/...`, `**/Logs/**/*.log*`, etc.) across all partitions; the decoder doesn't need to know which partition holds the logs
3. Stream each matching file directly from the QNX6 image without writing it to disk; parse lines with the regex set above
4. Extract timestamps from log entries, validate coordinates, and convert to standard format
5. Sort entries chronologically; `Source_File` column carries the internal POSIX path inside the image (e.g. `/persistentLogs/AASXMTC/Log1`)

**Data Format**:

- Coordinates stored as decimal degrees in text format  
- Timestamps in various formats (`MM/DD/YYYY HH:MM:SS.mmm` or `YYYY.MM.DD HH:MM:SS,mmm`)  
- Some patterns include additional data like speed and heading

#### **Denso Decoder**

File Format: Denso and Acura Android eMMC images (.001, .bin, .CE0 files)  
Data Location: JSON-formatted telemetry data embedded in binary files  
Extraction Method: Binary pattern matching with JSON parsing  
Key patterns:

- `Navigation.Location` - GPS coordinates with accuracy and velocity data  
- `Frame.VehicleSpeed` - Vehicle speed in kilometers per hour  
- `Phone.BluetoothConnection` - Bluetooth device connection events  

**Extraction Process**:

1. Scan binary file for JSON telemetry record boundaries  
2. Extract records using regex pattern matching for timestamp and tag fields  
3. Parse JSON-formatted data from binary context  
4. Categorize data by event type (location, speed, bluetooth)  
5. Convert timestamps from ISO format to Unix epoch and UTC  
6. Validate GPS coordinates and filter invalid entries  

**JSON Record Structure**:

- `timestamp` - ISO format timestamp (e.g., `2023-12-15T14:30:25.123Z`)  
- `tag` - Event type identifier (`Navigation.Location`, `Frame.VehicleSpeed`, `Phone.BluetoothConnection`)  
- `value` - Event-specific data payload containing coordinates, speed, or device information  

**GPS Data Format**:

- Coordinates stored as decimal degrees in JSON `coordinate` object  
- Additional fields: `accuracy`, `speed`, `bearing`, `fixTime`  
- Speed data includes `kilometersPerHour` field  
- Bluetooth data includes `deviceName`, `deviceId`, `deviceAddress`, and connection `state`  

**Output Structure**:

The decoder creates separate data categories for comprehensive analysis:
- Location data with GPS coordinates and navigation details  
- Speed data with vehicle velocity measurements  
- Bluetooth data with device connection logs and states  

#### **BMW NBT-HDD Decoder**

File Format: Raw BMW NBT-HDD image (.bin/.img/.dd/.001/.raw) **or** a manually-supplied `trails.sqlite` / `trips.sqlite` (.sqlite/.db)
Data Location: `/nav/trails.sqlite` and `/nav/trips.sqlite` inside any QNX6 partition of the image; for direct uploads the file is processed in place
Extraction Method: QNX6 partition mount (image inputs only) → SQLite extraction → schema detection → TLV-stream decoding of the Path BLOB with Bounding-box filtering and per-fix timestamp reconstruction
Process:

1. Mount every QNX6 partition in the raw image; the decoder probes each for `/nav/` rather than requiring the user to identify the right partition (commonly `p2` but varies by case). If the input is a `.sqlite`/`.db`, the QNX6 mount is skipped and the file is read directly — useful for working with DBs already pulled by another tool while keeping the same decode pipeline.
2. Extract `/nav/trails.sqlite` and `/nav/trips.sqlite` to a private temp directory; both are processed
3. Schema detection: `trails.sqlite` carries `Trails(BeginCoordinatedUniversalTime, EndCoordinatedUniversalTime, Path, Bounding, Length, DurationTime, ...)`; `trips.sqlite` carries the same shape under `Trips(Departure*, Arrival*, ...)`
4. Walk the Path BLOB as a d-monotone TLV stream. Each record begins with a 1-byte tag and a 4-byte cumulative-distance `d` (centimetres). GPS markers (`0x1e` = begin sample, `0x1d` = end sample) carry a 4-byte longitude + 4-byte latitude as signed int32. Time-anchor records (`0x02`) carry a 4-byte `v_ms` value — milliseconds since the trail's first GPS lock. Other tags (`0x07`, `0x15`, `0x16`, `0x17`, `0x1f`, `0x20`, `0x22`, `0x24`, `0x72`) are sized from a known table; unknown tags are resynced byte-by-byte using d-monotonicity as the validity constraint
5. For each GPS event, look up the co-located `0x02` time anchor and compute `fix_unix = BeginCoordinatedUniversalTime + (v_ms − v0_ms) / 1000`. Parking is detected when multiple `0x02` records share a `d` but `v_ms` jumps by more than 30 seconds — the decoder emits both a pre-park and a wake-up fix so the parked interval is preserved
6. Merge entries from both DBs, dedupe, and emit rows tagged with their `SourceTable` (`/nav/trails.sqlite:Trails`, `/nav/trips.sqlite:Trips`) and a per-fix `FixTime_UTC` column

**Database Schemas**:

- `TrailId` / `TripId` - Unique identifier per trail or trip
- `BeginCoordinatedUniversalTime` / `DepartureCoordinatedUniversalTime` - Unix epoch start
- `EndCoordinatedUniversalTime` / `ArrivalCoordinatedUniversalTime` - Unix epoch end
- `Length` - Total trail distance in centimetres (used to cap the TLV parser's `d` range)
- `DurationTime` - Trail duration in milliseconds (used to reject desync'd `0x02` anchors whose v_ms exceeds the real trip length)
- `Path` - Binary TLV stream of GPS samples, time anchors, and other event records
- `Bounding` - 28-byte SW/NE bounding box used as a geographic filter against marker-byte false positives

**Path Binary Format**:

* Header begins with `0a 01 01 00 01 00`. The byte at offset 0x16 discriminates between the short (0x24 → 45-byte header) and long (0x22 → 65-byte header) variants observed in the wild
* GPS coordinates encoded as signed 32-bit integers (no unsigned-to-signed cast, unlike Mercedes); formula: `decoded_value = encoded_value * 180 / 2147483647`
* `d` is cumulative distance in centimetres (matches the row's `Length` column at end-of-trail) and is strictly non-decreasing across the stream
* `v_ms` in `0x02` records is milliseconds since first GPS lock (matches `DurationTime` at end-of-trail). Berla iVE's per-fix timestamps are reproduced to within ~1 second for 99.7%+ of fixes across 1000+ validated trails

**Validating against Berla iVE**:

The decoder's accuracy was developed against Berla iVE's `journey-*.txt` exports as ground truth. Across 1000+ trails from a real case dataset, drift vs. Berla measured: p50 = 0.5 s, p90 = 1.3 s, p99 = 2.1 s, p99.9 = 7.6 s. The remaining tail is dominated by Berla emitting synthetic heartbeat fixes during long parks that aren't physically present in the Path BLOB.

#### **Honda TLHOBINN0D1 Decoder**

File Format: Honda telematics binary files (.CE0, .bin, .001 files)  
Data Location: Text-formatted GPS records embedded in binary files  
Extraction Method: Pattern matching for TIMESTAMP markers with field extraction  
Key patterns:

- `TIMESTAMP:` - Unix epoch timestamp in milliseconds  
- `LATITUDE:` - GPS latitude in decimal degrees  
- `LONG*` - GPS longitude in decimal degrees (with special character handling)  
- `ALTITUDE:` - Altitude data  
- `SPEED:` - Speed in meters per second  
- `BEARING:` - Heading/bearing information  

**Extraction Process**:

1. Scan binary file for TIMESTAMP markers using regex pattern matching  
2. Extract text blocks between consecutive TIMESTAMP markers  
3. Parse GPS data fields using flexible regex patterns  
4. Handle special characters in field names (e.g., `LONG¬` instead of `LONGITUDE`)  
5. Validate coordinates and filter invalid entries (null island, out of range)  
6. Convert Unix epoch milliseconds to UTC timestamps  

**Data Fields Extracted**:

- **Basic GPS**: Latitude, longitude, altitude, accuracy  
- **Motion**: Speed, bearing  
- **Quality**: Fix type, HDOP, VDOP, PDOP, number of satellites  
- **GNSS Systems**: GPS_SVC, GLONASS, GALILEO constellation indicators  
- **Time Components**: Year, month, day, hour, minute, GPS week, GPS TOW  
- **Metadata**: Source identifier, geodetic system, hex offset location  

**Record Format**:

- Records begin with `TIMESTAMP:` followed by Unix epoch in milliseconds  
- Each field follows the pattern `FIELD_NAME:VALUE`  
- Special characters may appear in field names due to binary corruption  
- Multiple regex patterns used to handle field name variations  
- Records are variable length, separated by next TIMESTAMP marker  

**Output Structure**:

The decoder provides comprehensive GPS telemetry with 26 columns including:
- Hex offset for binary file reference  
- Unix epoch and formatted UTC timestamp  
- Full GPS coordinates and motion data  
- Signal quality and satellite constellation information  
- Alternative timestamp components for verification  

#### **Kia Dealer Mode Decoder**

File Format: Kia head unit dealer-mode log bundles (.tar.gz files)  
Data Location: AUTOSAR DLT binaries and plain-text service dumps inside the tarball  
Extraction Method: Streaming tarball read with chunked binary regex scanning (no on-disk extraction)  
Process:

1. Open the .tar.gz with Python's `tarfile` module in streaming mode  
2. Route each member by path: `customlog/servicedump-*.log` and `dltlog/log_*.dlt`  
3. For DLT logs, slide a chunked buffer with overlap to find ASCII GPS payloads embedded in DLT verbose-mode messages  
4. For each match, walk backwards to the nearest DLT storage header (`DLT\x01` magic) to recover a per-message UTC timestamp  
5. Validate coordinates and reject the half-zero "no fix" sentinel Kia emits during cold start  

**Key Patterns**:

- `LocationService: gps <lon>, <lat>, <alt>, <speed>, <heading>, ...` - Primary GNSS fix stream from the Trimble receiver  
- `onLocationMapMatchingInfoChanged: lon=, lat=, alt=, handing=` - Map-matched position output  
- `CompassWidget ... lati:<int>, longi:<int>` - Integer fixed-point fixes (1e-6 degrees)  
- `longitude :`, `laptitude :`, `altitude :`, `time :` - LocationService snapshot inside servicedump  

**Timestamp Recovery**:

- Per-fix UTC timestamps are decoded from the 16-byte DLT storage header preceding each match (4-byte magic + LE uint32 seconds + LE uint32 microseconds + 4-byte ECU id)  
- Pre-2000 epochs are rejected to filter out RTC-not-yet-synced boots  
- Falls back to the filename's embedded `YYYYMMDD-HHMMSS` if no header is in range  

**Output Structure**:

The decoder produces 13 columns including:
- Latitude, longitude, and UTC timestamp  
- Source tag (`DLT.LocationService`, `DLT.MapMatching`, `DLT.CompassWidget`, `ServiceDump.LocationService`)  
- Altitude, speed, heading, satellite count, HDOP, PDOP  
- Timestamp source (DLT storage header / DLT filename / log line) for provenance  
- Member path and member offset (hex) inside the tarball for forensic citation  

### **Installation from Source**

#### **Windows**

```bash
# Clone repository  
git clone https://github.com/BitEU/fender.git  
cd fender

# Install dependencies  
pip install -r requirements.txt

# Run application  
python main.py
```

#### **Linux/macOS**

```bash
# Clone repository  
git clone https://github.com/BitEU/fender.git  
cd fender

# Install dependencies  
pip install -r requirements.txt

# Install system dependencies for pytsk3  
# Ubuntu/Debian:  
sudo apt-get install libtsk-dev

# macOS:  
brew install sleuthkit

# Install pytsk3  
pip install pytsk3

# Run application  
python main.py
```

### **Building Executables**

#### **Windows Executable with PyInstaller**

```bash
# Run build.bat
build.bat

# Output will be in dist/main_.exe
```

### **Decoder Development**

See the Development Tutorial for detailed instructions on creating new decoders.

#### **Key Methods to Implement**

1. `get_name()` - Decoder display name  
2. `get_supported_extensions()` - File extensions list  
3. `extract_gps_data()` - Main extraction logic  
4. `get_xlsx_headers()` - Column headers for output  
5. `format_entry_for_xlsx()` - Format GPS data for Excel

### **Data Formats**

#### **GPSEntry Structure**

```python
@dataclass  
class GPSEntry:  
    lat: float              # Latitude in decimal degrees  
    long: float             # Longitude in decimal degrees  
    timestamp: str          # ISO format timestamp  
    extra_data: Dict[str, Any]  # Decoder-specific metadata
```

#### **XLSX Output Format**

Each decoder can define custom columns, but typically includes:

* Latitude/Longitude coordinates  
* Timestamp information (per-fix where the source format carries it — e.g. BMW NBT-HDD emits a `FixTime_UTC` column alongside the trail-level Begin/EndTime)
* Decoder-specific metadata  
* Hex representations (for debugging and data verification)

### **Troubleshooting**

#### **Common Issues**

**"No decoders found" error**

- Ensure `decoders/` directory exists  
- Check that decoder files end with `_decoder.py`  
- Verify Python path includes the application directory

**Honda decoder not working**

- Install `pytsk3` library  
- Ensure you have a valid Android eMMC image  
- Check that the image contains a userdata partition

**Large file processing**

- Files over 4GB may require 64-bit Python  
- Ensure sufficient RAM (8GB+ recommended for large files)  
- Consider using CLI mode for better performance

**Windows Defender warnings**

- Add exception for `FENDER.exe`  
- Or build from source yourself


## **Todo**

* Add QNX support
* Plotting points on an interactive map
* Include more data than just timestamps and geolocation
* Batch processing
* Implement anomoly detection to flag any rows that arent in line eith the rest of the data
* Make this program compliant with leading guidelines (ISO 27037? NIST 800-86?)
    * NIST 800-86
        * Need more contextual reporting (offsets, file paths, etc)
* Improve unit testing
* Input sanitization
* Memory efficient processing
* Make test files publically available

## **Scoreboard**

This is the current tally of vehicles supported by FENDER. This contains the most common vehicles in the United States.

<br>

Fully supported vehicles:

* Toyota Group
  * Toyota
  * Lexus
* Honda Motor Co
  * Honda
  * Acura
* General Motors
  * GMC
  * Chevrolet
  * Buick
  * Cadillac

<br>

Vehicles with partial (parse from file/folder, not disk image) support:

* Hyundai Group
  * Kia (dealer-mode USB log bundle only)

<br>

Currently unsupported:

* Nissan Motor Corp
  * Nissan
  * Infiniti
* Hyundai Group
  * Hyundai
* Ford Motor Company
  * Ford
  * Lincoln
* Volkswagen Group
  * Volkswagen
  * Porsche
  * Audi
* Volvo
* Tesla

## **Contributing**

1. Fork the repository  
2. Create a feature branch  
3. Add your decoder to `decoders/`  
4. Include test files for validation. If you are unable to publically supply them due to sensitive content or an ongoing investigation, please contact sschiavone@westchesterda.net.
5. Submit a pull request

## **Credits**

1. This project includes images created by [Iconoir](https://iconoir.com/). Copyright 2025 Iconoir
