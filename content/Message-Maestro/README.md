# Message Maestro

Message Maestro is a modern, GPU-accelerated desktop application for viewing, tagging, and exporting conversations from various messaging platforms. It features a beautiful, responsive UI built with PyQt6, advanced message parsing, customizable tagging, and high-quality PDF export. The application is designed for both casual users and professionals who need to analyze, archive, or present chat data.

## Features

- **Multi-Platform Support:**
  - Import and view conversations from Kik Messenger, Snapchat, Twitter DMs, and Discord DMs (with extensible parser support).
- **Modern UI:**
  - GPU-accelerated scrolling and animated widgets for a smooth, visually appealing experience.
- **Tagging System:**
  - Tag messages with custom labels and colors for organization, review, or analysis.
- **PDF Export:**
  - Export entire conversations to beautifully formatted PDFs, including tag legends and color-coded message bubbles.
- **Media & Link Detection:**
  - Messages with media are displayed for Snapchat and Kik.
  - **URL Image Loading:** Automatically downloads and displays images, GIFs, and other media from URLs (Discord, Twitter, etc.) with intelligent caching and graceful error handling.
- **Keyboard Shortcuts:**
  - Efficient navigation and tagging with keyboard shortcuts (see TODO for planned enhancements).
- **Statistics Dashboard:**
  - Comprehensive message analytics with interactive charts and visualizations.
  - View message patterns, activity trends, response times, and conversation insights.
  - Export statistics reports to PDF and JSON formats.
- **Extensible Architecture:**
  - Easily add new message parsers for other platforms by implementing the `BaseParser` interface.

## Installation

1. **Clone the repository:**
   ```sh
   git clone <repo-url>
   cd Message-Maestro
   ```
2. **Install dependencies:**
   ```sh
   pip install -r requirements.txt
   ```

## Usage

Run the application:

```sh
python message_viewer.py
```

- Use the file dialog to open exported message files from supported platforms.
- Tag messages, search, and export conversations as PDF.

## Supported Platforms

- **Kik Messenger** (`.csv`)
- **Snapchat** (`.csv`)
- **Twitter DM** (`.txt`)
- **Discord DM** (`.txt`)

## Search Bars Explained

Message Maestro features two powerful and distinct search bars to help you quickly find conversations and messages:

### 1. Global Conversation Search (Sidebar Search Bar)

- **Location:** At the top of the left sidebar, always visible.
- **Purpose:** Search across all loaded conversations for keywords in conversation titles, participant names, or message content.
- **How it works:**
  - Type a keyword or phrase and press Enter (or click the search icon).
  - Results instantly filter the conversation list below, showing only conversations that match your search.
  - You can refine your search using radio buttons:
    - **All:** Search both conversation titles and message content.
    - **Titles:** Only search conversation titles and participant names.
    - **Content:** Only search within the text of messages.
  - Click the clear (✕) button to reset the search and show all conversations again.
- **Use case:** Quickly locate a conversation by a participant's name, group title, or a word/phrase mentioned anywhere in the chat history.

### 2. In-Conversation Search (Header Search Bar)

- **Location:** At the top of the main chat view, appears after you open a conversation and click the '🔍 Search' button in the header.
- **Purpose:** Search for specific words or phrases within the currently open conversation.
- **How it works:**
  - Type your search term and press Enter.
  - All matching messages are highlighted in the chat view.
  - Use the **Next ↓** and **↑ Prev** buttons to jump between matches.
  - The search bar displays the number of matches and your current position (e.g., "3 of 7").
  - Click the close (✕) button to exit in-conversation search and clear highlights.
- **Use case:** Find every instance of a word, phrase, or name within a single conversation, and quickly navigate between them for review or tagging.

**Tip:** The two search bars are independent; use the sidebar search to find the right conversation, then use the in-conversation search to drill down to the exact messages you need.

## Statistics and Analytics

Message Maestro includes a comprehensive statistics dashboard that provides deep insights into your conversation data with beautiful visualizations and analytics.

### Statistics Dashboard Features

- **Message Volume Analytics:**
  - Total message counts across all conversations
  - Messages per sender with interactive charts
  - Temporal analysis showing activity patterns by hour and day of week
  - Message length statistics and averages

- **Activity Patterns:**
  - Hourly activity heatmaps showing when conversations are most active
  - Day-of-week patterns to identify communication trends
  - Response time analysis to understand conversation dynamics
  - Most active participants and fastest responders

- **Interactive Visualizations:**
  - Bar charts for message counts and sender activity
  - Pie charts for proportional data analysis
  - Line charts for temporal trends and patterns
  - Custom dark-themed charts that match the application's modern UI

- **Data Export:**
  - Export comprehensive statistics reports to PDF format
  - JSON data export for further analysis in external tools
  - Formatted reports with charts and detailed breakdowns

### Accessing Analytics

1. **Open Statistics Dashboard:** Click the "Statistics" button in the main toolbar
2. **View Overall Stats:** Browse comprehensive statistics for all loaded conversations
3. **Export Reports:** Use the export buttons to save statistics and charts to PDF or JSON

## Directory Structure

- `message_viewer.py` — Main application (PyQt6 GUI)
- `parsers/` — Platform-specific message parsers
- `message_stats/` — Statistics and analytics modules
  - `stats_dashboard.py` — Main statistics dashboard interface
  - `stats_calculator.py` — Statistical analysis engine
  - `chart_widgets.py` — Custom chart visualizations
  - `stats_exporter.py` — Export functionality for reports
- `templates/` — Example export templates and sample data
- `requirements.txt` — Python dependencies

## Dependencies

- **Core Dependencies:**
  - PyQt6 — Modern GUI framework
  - reportlab — PDF generation and export
  - weasyprint — Advanced PDF styling
  - requests — HTTP library for downloading media from URLs

- **Statistics and Analytics:**
  - numpy — Numerical computations for statistics

Install all dependencies with `pip install -r requirements.txt`.

## Contributing

Contributions are welcome! To add support for a new platform, implement a new parser in `parsers/` by subclassing `BaseParser`.