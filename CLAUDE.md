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
   - Width normalization: Scales all blocks to median width so they align perfectly in rows
7. **Row Arrangement** (`arrangeInRows()`) - Groups blocks into rows, normalizes heights within each row
8. **Rendering** (`renderSheet()`) - Creates DOM structure for preview and print with dynamic heights based on image aspect ratios

### Key Algorithms

**Bar Line Detection Algorithm** (index.js:105-197):
- Samples vertical columns from 20% to 80% of image height (staff area)
- Tracks longest continuous dark streak and total dark pixel density
- Combines metrics to identify genuine bar lines vs. noteheads or other elements
- Enforces minimum spacing to avoid duplicate detections

**Block Validation & Merging** (index.js:225-344):
- Uses median width (more robust than average) as reference
- Tolerance: 60%-140% of reference width is acceptable
- First pass: Too narrow → merge forward with next blocks
- Second pass: Very narrow (<30% median) → merge backward with previous block (handles double bar lines at end)
- Too wide (>280%) → split in half (likely missed bar line)
- Width normalization: All blocks are scaled to median width for perfect alignment in rows
- Prevents misaligned measures and handles end-of-sheet narrow segments

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

The print CSS (styles.css:186-252) ensures clean, compact output:
- Hides all UI controls and headers
- Removes all padding/margins from sheet container and rows
- Each row has `page-break-inside: avoid` to prevent splitting across pages
- `gap: 0` removes horizontal spacing between blocks in a row
- `margin: 0; padding: 0` on all elements removes all spacing
- `line-height: 0; font-size: 0` eliminates inline spacing issues
- Row height is dynamic based on image content (no fixed height)
- Segments use `flex: 1 1 0` for equal width distribution
- Images use `height: auto` to maintain aspect ratio naturally
- `vertical-align: top` prevents baseline alignment spacing
- Separator lines between segments are hidden in print

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
- Modify multipliers in `validateAndMergeBlocks()` (lines 238-239)
- Currently: 0.6x to 1.4x median width is acceptable
- Modify `veryNarrowThreshold` (line 242) to change threshold for backward merging
- Currently: blocks < 0.3x median width are merged backward

**Default Blocks Per Row:**
- Change `value="4"` in index.html:24

## Known Limitations

- Requires CORS-enabled image URLs (Flowkey images must allow cross-origin access)
- Bar line detection may fail on highly stylized or hand-written notation
- No undo/redo functionality - must re-paste and re-parse
- Width normalization may slightly stretch or compress measures, but ensures consistent alignment
