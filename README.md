# Flowkey Notes Parser & Printer

A client-side web application that automatically parses Flowkey sheet music images, detects measures, and arranges them into perfectly aligned, printable rows. No backend required - everything runs in your browser.

## Features

- **Automatic Bar Line Detection** - Intelligently identifies measure boundaries using advanced image processing
- **Smart Block Validation** - Merges or splits measures to ensure consistent spacing
- **Clef Preservation** - Displays clef and key signature on every row for easy reading
- **Perfect Alignment** - Staff lines align seamlessly across all rows
- **Print-Optimized** - Clean, professional output with no UI elements
- **100% Client-Side** - No data sent to servers, works offline once loaded

## Live Demo

Visit the live application: [https://cjvnjde.github.io/flowkey.parser/](https://cjvnjde.github.io/flowkey.parser/)

*(Note: Replace with your actual GitHub Pages URL)*

## Quick Start

### Option 1: Use Online (Recommended)

Simply visit the [live demo](https://cjvnjde.github.io/flowkey.parser/) and start using it immediately.

### Option 2: Run Locally

```bash
# Clone the repository
git clone https://github.com/cjvnjde/flowkey.parser.git
cd flowkey.parser

# Serve locally (choose one method)
npx live-server
# OR
python -m http.server 8000
# OR simply open index.html in your browser
```

Then open `http://localhost:8080` (or the appropriate port) in your browser.

## How to Use

### Step 1: Get Flowkey HTML Content

1. Open a Flowkey lesson in your browser
2. Open browser DevTools (F12 or right-click → Inspect)
3. Find the sheet music elements (look for `.sheet-image` divs)
4. Right-click on the parent container and select "Copy → Copy outerHTML"

**Example of what the HTML should look like:**
```html
<div class="sheet-image" style="background-image: url('https://...'); width: 500px; left: 0px;">...</div>
<div class="sheet-image" style="background-image: url('https://...'); width: 300px; left: 500px;">...</div>
```

### Step 2: Paste HTML

1. Paste the copied HTML into the text area in the application
2. The HTML should contain multiple `<div>` elements with `sheet-image` class

### Step 3: Configure Layout

Set the **Blocks per Row** value (default: 4):
- **4 blocks** - Good for most practice pieces
- **2 blocks** - Better for beginners or complex passages
- **6 blocks** - Compact view for simpler pieces

### Step 4: Parse & Preview

Click the **Parse & Preview** button. The application will:
1. Extract all sheet music images from the HTML
2. Load and combine the images
3. Detect bar lines automatically
4. Split into individual measures
5. Validate and normalize block widths
6. Arrange into rows with proper spacing
7. Add clef and key signature to each row

**You'll see:**
- Processing status messages
- Preview of the arranged sheet music
- Each row as a perfectly aligned image
- Clef and key signature on every row (except the first which has it in the music)

### Step 5: Print

Click the **Print** button to open the print dialog. The output will:
- Show only the sheet music (all UI hidden)
- Display perfectly aligned staff lines
- Maintain consistent measure widths
- Include clef on every row for easy reading
- Fit cleanly to page width

**Print Settings:**
- Recommended: Portrait orientation
- Paper size: Letter or A4
- Margins: Default or minimal

## Usage Examples

### Example 1: Basic 4-Bar Layout

**Input:** Flowkey HTML with 16 measures of music

**Configuration:**
- Blocks per Row: 4

**Result:**
- 4 rows of music
- Each row shows 4 complete measures
- Clef visible on every row
- Perfect staff line alignment

### Example 2: Beginner-Friendly 2-Bar Layout

**Input:** Complex piece with intricate rhythms

**Configuration:**
- Blocks per Row: 2

**Result:**
- More rows with fewer measures each
- Easier to read and practice
- Less page turning needed
- Each row still perfectly aligned

### Example 3: Compact 6-Bar Layout

**Input:** Simple melody with repetitive patterns

**Configuration:**
- Blocks per Row: 6

**Result:**
- Fewer total pages
- More measures visible at once
- Great for getting an overview
- Still maintains readability

## Technical Details

### How It Works

1. **HTML Parsing** - Extracts image URLs and positioning from inline styles
2. **Image Loading** - Loads all images with CORS support
3. **Image Combination** - Stitches images horizontally into single canvas
4. **Bar Line Detection** - Uses dual scoring algorithm:
   - Continuity score (70%): Height of continuous dark streak
   - Density score (30%): Overall darkness of vertical line
   - Filters by width (1-6px) and enforces minimum spacing (100px)
5. **Block Validation** - Ensures consistent measure widths:
   - Merges blocks that are too narrow
   - Splits blocks that are too wide
   - Preserves first block width (contains clef)
   - Preserves last block width (contains endings)
6. **Row Composition** - Creates one merged canvas per row:
   - Draws clef image on left (rows 2+)
   - Draws all measures sequentially
   - Adds proper margins
7. **Rendering** - Displays each row as a single image

### Technology Stack

- **Pure HTML/CSS/JavaScript** - No frameworks or build tools
- **Canvas API** - For image processing
- **GitHub Pages** - Static site deployment

## Troubleshooting

### Images Not Loading

**Issue:** "Failed to load images" error

**Solution:**
- Ensure the Flowkey images are accessible (not behind authentication)
- Check browser console for CORS errors
- Try opening the images directly in a new tab to verify URLs

### Bar Lines Not Detected

**Issue:** Missing or extra bar line detections

**Solution:**
- The detection algorithm works best with standard notation
- Hand-written or highly stylized music may not work
- Try adjusting detection thresholds in the code if needed

### Blocks Too Wide or Narrow

**Issue:** Measures appear stretched or compressed

**Solution:**
- This is expected behavior to maintain alignment
- The algorithm normalizes widths for consistent spacing
- First and last measures preserve original width for clef/endings

### Print Layout Issues

**Issue:** Music doesn't fit page properly

**Solution:**
- Use Print Preview to check layout
- Try different blocks per row settings
- Ensure printer settings use minimal margins
- Use portrait orientation

## Development

### File Structure

```
flowkey.parser/
├── index.html          # Complete application (HTML + CSS + JS)
├── .github/
│   └── workflows/
│       └── static.yml  # GitHub Pages deployment
├── CLAUDE.md           # Developer documentation
└── README.md           # This file
```

### Making Changes

The entire application is contained in `index.html`:
- Lines 1-30: HTML structure and UI controls
- Lines 31-200: CSS styling (including print styles)
- Lines 201-670: JavaScript application logic

### Deployment

Automatically deploys to GitHub Pages on push to `main` branch via GitHub Actions.

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

### Common Modifications

**Adjust bar detection sensitivity:**
```javascript
// Line ~157 in index.html
const scoreThreshold = 0.4;        // Lower = more sensitive
const continuityThreshold = 0.5;   // Lower = detects shorter lines
```

**Change block width tolerance:**
```javascript
// Line ~302 in index.html
const toleranceMin = medianWidth * 0.6;  // Minimum acceptable width
const toleranceMax = medianWidth * 1.4;  // Maximum acceptable width
```

## License

This project is open source and available for personal and educational use.

## Acknowledgments

- Designed for use with [Flowkey](https://www.flowkey.com/) piano learning platform
- Uses HTML5 Canvas API for image processing
- Deployed via GitHub Pages

## Support

If you encounter issues or have suggestions, please [open an issue](https://github.com/cjvnjde/flowkey.parser/issues) on GitHub.

---

**Note:** This tool is for personal use and educational purposes. Please respect Flowkey's terms of service and copyright when using their content.
