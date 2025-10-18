# Worksheet Generator - Development Notes

## Overview
This is a browser-based math worksheet generator built as a single HTML file (`worksheet.html`). It generates printable worksheets with consistent, seeded random questions based on worksheet number.

## Architecture

### Key Design Principles
1. **Modular Question Generators**: Each worksheet type is defined in the `questionGenerators` object with a `title` and `generate(rng)` function
2. **Seeded Randomization**: Uses `SeededRandom` class to ensure the same worksheet number always generates the same questions
3. **Single File Application**: Everything (HTML, CSS, JavaScript) is contained in `worksheet.html` for easy distribution

### File Structure
- `worksheet.html` - The complete application (only file in the repo)
- `CLAUDE.md` - This file (development notes)

## Current Worksheet Types

### 1. Multiply & Divide by 10 and 100
- Original worksheet type
- Generates numbers with up to 2 decimal places
- Operations: ×10, ×100, ÷10, ÷100
- Includes both positive and negative numbers (30% negative)

### 2. Convert Decimals to Fractions
- **Location**: `questionGenerators['decimal-to-fraction']` (line ~307)
- Uses `generateWeightedFraction(rng)` helper function
- Displays decimals like `19.4 ` (trailing space instead of `19.40` for alignment)
- Answer format: simplified fractions (e.g., `1/2`, `3/4`, `19 2/5`)
- Whole number part: 0-99

### 3. Convert Fractions to Decimals
- **Location**: `questionGenerators['fraction-to-decimal']` (line ~344)
- Uses same `generateWeightedFraction(rng)` helper
- Question format: simplified fractions (e.g., `1/2`, `19 3/4`)
- Answer format: decimals to 2 decimal places (e.g., `0.50`, `19.75`)
- Whole number part: 0-99

### 4. Times Tables Practice (up to 12×12)
- **Location**: `questionGenerators['times-tables']` (line ~374)
- Weighted distribution favoring harder multiplication facts
- Question format: `7 × 8 =`
- Answer: integer product

## Key Helper Functions

### `generateWeightedFraction(rng)` (line ~232)
Generates fractions with pedagogically-sound distribution:

**95% of questions**: Weighted by denominator using `P(d) ∝ 1/(d^0.7)`
- Denominators: [2, 4, 5, 10, 20, 25, 50, 100]
- Alpha parameter: 0.7 (controls decay rate)
- Numerators are coprime to denominator (avoids redundant fractions like 2/4)

**5% of questions**: Arbitrary fractions
- Denominator: 2-100
- Numerator: 1 to (denominator-1)
- Provides edge cases like 27/100

### Weighting Examples
With α=0.7, approximate probabilities:
- Denominator 2: ~25%
- Denominator 4: ~10%
- Denominator 5: ~8%
- Denominator 10: ~5%
- Denominator 20: ~3%
- Denominator 25: ~3%
- Denominator 50: ~2%
- Denominator 100: ~1%

### Times Tables Weighting
Tier-based distribution (line ~383):
- **Tier 1 (15%)**: Easy - 1, 2, 10, 11
- **Tier 2 (25%)**: Moderate - 3, 4, 5
- **Tier 3 (45%)**: Challenging - 6, 7, 8, 9 (most practice here!)
- **Tier 4 (15%)**: Advanced - 12

One operand uses weighted selection, the other is uniform 1-12. Order is randomly swapped.

## UI Features

### Controls
- **Worksheet Type Selector**: Dropdown to choose question type (line ~174)
- **Start Page**: First worksheet number to generate
- **End Page**: Last worksheet number to generate (auto-updates if < start page)
- **Questions per Sheet**: 10-40 questions (default: 40)

### Page Syncing (line ~542)
`syncEndPage()` function ensures end page ≥ start page automatically

### Buttons
- **Generate Worksheets**: Creates worksheets for specified page range
- **Print**: Opens browser print dialog (appears after generation)
- ~~Save as PDF~~ (removed - users can use browser print-to-PDF)

## Canvas Rendering

### `drawWorksheet()` function (line ~419)
- Canvas size: 816×1056 px (A4 proportions at 96 DPI)
- Uses 2× DPI scaling for crisp rendering
- Two-column layout with dynamic row height
- Monospace font for questions to maintain alignment
- Modern sans-serif font for headers

### Layout
- Questions arranged in 2 columns
- Row height: 38px
- Column width: 360px
- Questions per column: ceil(total/2)

## How to Add a New Worksheet Type

1. **Add to dropdown** (line ~174):
```html
<option value="new-type">New Worksheet Type Name</option>
```

2. **Add generator to `questionGenerators` object** (before line ~417):
```javascript
'new-type': {
    title: 'Display Title for Worksheet Header',
    generate: function(rng) {
        // Your generation logic here
        // Use rng.next() for random numbers (returns 0-1)

        return {
            question: "question text or expression",
            answer: "answer (number or string)"
        };
    }
}
```

3. The rest is automatic - the system will handle rendering, printing, etc.

## Technical Details

### SeededRandom Class (line ~210)
- Simple linear congruential generator
- Formula: `seed = (seed × 9301 + 49297) % 233280`
- Ensures deterministic question generation per worksheet number
- Each worksheet uses seed = `pageNumber × 12345`

### Question Format
All generators return objects with:
- `question`: String displayed to student (e.g., `"7 × 8 ="`, `"19.4 "`, `"3/4"`)
- `answer`: String or number for answer key (e.g., `56`, `"1/2"`, `"0.75"`)

### Print Styling (line ~140)
- Hides controls when printing
- Forces A4 portrait layout
- Removes borders/shadows from canvas
- Page break after each worksheet

## Testing

To test locally:
```bash
python3 -m http.server 8000
# Open http://127.0.0.1:8000/worksheet.html
```

## Design Considerations

### Why single HTML file?
- Easy to share/distribute
- No build process needed
- Works offline
- Can be opened directly in browser

### Why canvas rendering?
- Consistent rendering across browsers
- Print-friendly
- Pixel-perfect layout control
- Can export to image/PDF if needed

### Why seeded randomization?
- Teachers can assign specific worksheet numbers
- Students can't "game" the system by regenerating
- Reproducible for answer keys
- Same worksheet number = same questions forever

## Future Enhancement Ideas
- Add answer key toggle (code structure already supports it, see commented lines ~534-536)
- More worksheet types (division, addition, subtraction, percentages, etc.)
- Configurable difficulty levels
- Custom question ranges per type
- Export individual worksheets as PNG/PDF
- Dark mode for on-screen viewing
