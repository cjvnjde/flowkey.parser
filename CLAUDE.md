# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Flowkey Notes Parser & Printer is a client-side web application that parses Flowkey HTML content containing sheet music images, automatically detects bar lines, splits the music into measures, and arranges them into printable rows. The entire application runs in the browser with no backend.

## Tech Stack

- **Pure HTML/CSS/JavaScript** - No build tools, frameworks, or dependencies
- **Canvas API** - For image processing and manipulation
- **GitHub Pages** - Static site deployment via `.github/workflows/static.yml`

## File Structure

- `index.html` - Main HTML structure with input controls and preview area
- `index.js` - Core application logic (~470 lines)
- `styles.css` - Styling including print-specific CSS media queries

## Running the Application

This is a static site. To run locally:
```bash
# Serve with any static file server, e.g.:
python3 -m http.server 8000
# Then open http://localhost:8000
```

Or simply open `index.html` directly in a browser (note: may have CORS issues with image loading).

## Core Architecture

### Data Flow Pipeline

The application follows a sequential processing pipeline:

1. **HTML Parsing** (`parseHTML()`) - Extracts `.sheet-image` elements from pasted HTML, parses inline styles to get image URLs, widths, positions
2. **Image Loading** (`loadAllImages()`) - Asynchronously loads all images with CORS enabled
3. **Image Combination** (`combineImages()`) - Stitches images horizontally into single canvas, normalizes vertical positioning
4. **Bar Line Detection** (`detectBarLines()`) - Uses vertical pixel analysis with dual scoring:
   - **Continuity score** (70% weight) - How tall the continuous dark streak is
   - **Density score** (30% weight) - Overall darkness of the vertical line
   - Requires both `scoreThreshold` (0.4) and `continuityThreshold` (0.5)
   - Filters by width (1-6px) and enforces minimum distance between bars (100px)
5. **Block Splitting** (`splitByBarLines()`) - Divides canvas at detected bar line positions
6. **Block Validation** (`validateAndMergeBlocks()`) - Critical quality control step:
   - Calculates median block width as reference
   - First pass: Merges blocks that are too narrow (< 60% of median) forward
   - Second pass: Merges very narrow blocks (< 30% of median, like double bar lines) backward with previous block
   - Splits blocks that are too wide (> 280% of median)
   - **First/Last block handling**:
     - First block: ALWAYS keeps original width (may contain clef/key signature)
     - Last block: ALWAYS keeps original width (may contain ending bars)
     - leftMargin = max(0, firstWidth - avgMiddleWidth)
     - rightMargin = max(0, lastWidth - avgMiddleWidth)
   - Width normalization: Middle blocks scaled to average width; first/last keep original size
7. **Row Arrangement** (`arrangeInRows()`) - Creates ONE canvas per row with built-in margins:
   - Determines margins: First row leftMargin=0, last row rightMargin=0
   - Creates single canvas with width = sum(block widths) + margins
   - Draws left margin as white space
   - Draws all blocks horizontally on the canvas
   - Draws right margin as white space
   - Returns array of row canvases (one image per row)
8. **Rendering** (`renderSheet()`) - Renders each row as a single image:
   - Each row is one `<img>` element displaying the merged canvas
   - No overflow issues - each row fits page width perfectly
   - Staff lines align because margins are built into the image
   - Simple, clean DOM structure

### Key Algorithms

**Bar Line Detection Algorithm** (index.js:105-197):
- Samples vertical columns from 20% to 80% of image height (staff area)
- Tracks longest continuous dark streak and total dark pixel density
- Combines metrics to identify genuine bar lines vs. noteheads or other elements
- Enforces minimum spacing to avoid duplicate detections

**Block Validation & Merging** (index.js:302-375):
- Uses median width (more robust than average) as reference
- Tolerance: 60%-140% of reference width is acceptable
- First pass: Too narrow → merge forward with next blocks
- Second pass: Very narrow (<30% median) → merge backward with previous block (handles double bar lines at end)
- Too wide (>280%) → split in half (likely missed bar line)
- **First/Last block special handling** (when >2 blocks total):
  - Calculates average width from middle blocks only (pure music width)
  - First block: ALWAYS keeps original width (clef visible)
  - Last block: ALWAYS keeps original width (endings visible)
  - leftMargin = max(0, firstWidth - avgWidth) - extra width for clef/key signature
  - rightMargin = max(0, lastWidth - avgWidth) - extra width for endings
- Width normalization: Only middle blocks scaled to targetWidth
- Returns `{blocks, leftMargin, rightMargin}` for row composition

**Row Composition** (index.js:377-422):
- Each row becomes a single merged canvas image
- Row width = sum(all block widths) + leftMargin + rightMargin
- Process per row:
  1. Create canvas with calculated width and max block height
  2. Fill with white background
  3. Draw leftMargin as white space (for rows 2+)
  4. Draw all blocks sequentially
  5. Draw rightMargin as white space (for rows except last)
- Result: One image per row with perfect alignment built-in

## Image Processing Details

- All processing uses HTML5 Canvas 2D context
- Images loaded with `crossOrigin = "anonymous"` for CORS
- Brightness threshold: 128 (RGB average < 128 = dark pixel)
- Minimum block width: 10px to filter noise

## UI Controls

- **HTML Input** - Paste area for Flowkey HTML containing `.sheet-image` divs
- **Blocks per Row** - Number of measures per line (default: 4)
- **Parse & Preview** - Triggers entire processing pipeline
- **Print** - Opens browser print dialog with optimized print CSS

## Print Layout

The print CSS (styles.css:186-220) ensures clean, compact output:
- Hides all UI controls and headers
- Each row is a single `<img>` element with `width: 100%; height: auto`
- `page-break-inside: avoid` prevents rows from splitting across pages
- `margin: 0; padding: 0` removes all spacing
- Each row image has margins built-in, so no CSS padding needed
- No overflow issues - images scale to fit page width
- Staff lines align perfectly because alignment is rendered into the images

## Deployment

- Auto-deploys to GitHub Pages on push to `main` branch
- Workflow: `.github/workflows/static.yml`
- Uploads entire repository root as static site

## Common Modifications

**Adjusting Bar Detection Sensitivity:**
- Modify `scoreThreshold` (line 157) - lower = more sensitive
- Modify `continuityThreshold` (line 158) - lower = detects shorter lines
- Modify `minBarDistance` (line 161) - controls duplicate detection

**Changing Block Width Tolerance:**
- Modify multipliers in `validateAndMergeBlocks()` (lines 237-238)
- Currently: 0.6x to 1.4x median width is acceptable
- Modify `veryNarrowThreshold` (line 241) to change threshold for backward merging
- Currently: blocks < 0.3x median width are merged backward

**Default Blocks Per Row:**
- Change `value="4"` in index.html:24

## Known Limitations

- Requires CORS-enabled image URLs (Flowkey images must allow cross-origin access)
- Bar line detection may fail on highly stylized or hand-written notation
- No undo/redo functionality - must re-paste and re-parse
- Width normalization may slightly stretch or compress measures, but ensures consistent alignment
