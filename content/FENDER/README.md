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
* **Mercedes-Benz** \- Extracts GPS data from Mercedes-Benz database files (.db files)
* **BMW NBT-HDD** \- Extracts GPS data from BMW NBT-HDD folder structure (folder)
* **Stellantis** \- Extracts GPS data from a folder holding Stellantis log files (folder)
* **Denso DNNS087** \- Extracts GPS, speed, and bluetooth data from Denso and Acura Android eMMC images (.001 files)
* **Kia Dealer Mode** \- Extracts GPS data from Kia head unit dealer-mode log bundles (.tar.gz files)

### **Features**

* 🚗 Multi-manufacturer support with modular decoder architecture  
* 📍 Extracts latitude, longitude, and timestamps  
* 📊 Exports data to XLSX, JSON, and KML formats for analysis  
* 🖱️ Drag-and-drop file support  
* 💻 Both GUI and command-line interfaces  
* 🔌 Plugin architecture for easy decoder additions

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

File Format: Mercedes-Benz log files and folders  
Data Location: Text-based log files in various locations  
Extraction Method: Folder-based recursive search and pattern matching  
Process:

1. Recursively scan folders for relevant log files  
2. Parse log files for GPS coordinate patterns  
3. Extract timestamp and location data using regex patterns  
4. Convert coordinate formats to standard decimal degrees  

**Key Log Patterns**:

* Navigation system location logs  
* Telematics service position reports  
* COMAND system diagnostic logs  
* Mercedes ME app synchronization data  

**Data Format**:

* Coordinates stored in various formats (decimal degrees, DMS)  
* Timestamps typically in ISO format or local system time  
* Additional vehicle status information often available in context  

#### **Stellantis Decoder**

File Format: Log files in persistent storage folders  
Data Location: Debug logs, service logs, and navigation logs  
Extraction Method: Recursive file search with multi-pattern matching  
Key patterns:

- `SAL_SDARS_FUEL` - Navigation destination coordinates  
- `NW_SOS` - Emergency call position data  
- `SAL_KONA_NAVI` - Navigation system coordinates  
- `GetCurrentLocAddressResponse` - Location service responses  
- `JSR179InterfaceImpl` - Low-level positioning with speed data  
- `NaviTelematicsDataRequest` - Telematics position reports

**Extraction Process**:

1. Scan folder structure for log files matching predefined patterns  
2. Parse files using regex patterns to extract GPS coordinates  
3. Extract timestamps from log entries  
4. Validate coordinates and convert to standard format  
5. Sort entries chronologically  

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

File Format: BMW NBT-HDD folder structure with SQLite database  
Data Location: `trails.sqlite` database in `NBT-HDD/p2/nav/` directory  
Extraction Method: Database extraction and binary path decoding  
Process:

1. Search for NBT-HDD folder structure in dropped folder/path  
2. Locate `trails.sqlite` database at expected path: `NBT-HDD/p2/nav/trails.sqlite`  
3. Query `Trails` table for navigation trail records  
4. Decode binary path data to extract GPS coordinates  
5. Convert timestamps from Unix format to UTC  

**Database Schema**:

- `TrailId` - Unique identifier for each navigation trail  
- `BeginTime`, `EndTime` - Unix timestamps for trail start/end  
- `Path` - Binary blob containing encoded GPS coordinates and events  

**Path Binary Format**:

* Similar to Mercedes-Benz NTG5*2 format  
* GPS coordinates encoded as 32-bit integers  
* Formula: `decoded_value = encoded_value * 180 / 2147483647`  
* Multiple GPS events per trail with elevation data  

**Expected Folder Structure**:

Users should drag and drop the `NBT-HDD` folder, which contains:
```
NBT-HDD/
├── p2/
│   └── nav/
│       └── trails.sqlite
└── [other system folders]
```

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
* Timestamp information  
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

* Mercedes-Benz
* BMW Group
  * BMW
  * Mini
* Stellantis
  * Chrystler
  * Dodge
  * Ram
  * Jeep
  * Fiat
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
4. Include test files for validation. If you are unable to publically supply them due to sensitive content or an ongoing investigation, please contact sschiavone@pace.edu.
5. Submit a pull request

## **Credits**

1. This project includes images created by [Iconoir](https://iconoir.com/). Copyright 2025 Iconoir
