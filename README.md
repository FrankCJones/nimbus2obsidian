# nimbus2obsidian
Convert notes exported from Nimbus (FuseBase) with HTML files in zip archives to a directory structure of MD files you can paste into an Obsidian vault.

# Obsidian Vault Note Reorganizer

This Python script is designed to help users reorganize notes exported from Nimbus Note or other note-taking applications into a format compatible with Obsidian, a popular Markdown-based note-taking app. The script processes ZIP archives containing notes, converts HTML files to Markdown, and reorganizes the directory structure to make the notes more accessible within Obsidian.

## Features

- **Extract ZIP Archives:** Automatically extracts notes and assets from ZIP archives.
- **HTML to Markdown Conversion:** Converts `note.html` files to `note.md` using `html2text`.
- **Preserves Directory Structure:** Maintains the original project and subproject folder hierarchy while moving note files up one level within their respective folders.
- **Asset Management:** Moves associated asset folders up one level and resolves any filename conflicts.
- **Customizable File Handling:** Includes an extensive list of supported file extensions for asset management, excluding unwanted file types like fonts, HTML, CSS, and archives.

## Prerequisites

Before using this script, ensure you have the following installed:

- Python 3.x
- Required Python libraries: `html2text`, `beautifulsoup4`

You can install the necessary libraries using pip:

```bash
pip install html2text beautifulsoup4
