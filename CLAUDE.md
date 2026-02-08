# Worksheet Generator - Development Notes

**IMPORTANT**: When making changes to this project, always update this CLAUDE.md file to reflect:
- New worksheet types added
- Changes to architecture or rendering approach
- Updates to helper functions or algorithms
- UI/UX improvements
- Changes to default values or configuration

Keep this documentation current as you work!

---

## Overview
This is a browser-based math worksheet generator built as a single HTML file (`worksheet.html`). It generates printable worksheets with consistent, seeded random questions based on worksheet number.

## Architecture

### Key Design Principles
1. **Modular Question Generators**: Each worksheet type is defined in the `questionGenerators` object with `title`, `defaultQuestions`, and `generate(rng)` function
2. **Seeded Randomization**: Uses `SeededRandom` class to ensure the same worksheet number always generates the same questions
3. **Single File Application**: Everything (HTML, CSS, JavaScript) is contained in `worksheet.html` for easy distribution
4. **HTML/CSS Rendering**: Uses DOM manipulation and CSS for layout (formerly canvas-based)
5. **MathML Support**: Fractions are rendered using native MathML for professional appearance

### File Structure
- `worksheet.html` - The complete application (only file in the repo)
- `CLAUDE.md` - This file (development notes)

## Current Worksheet Types (9 total)

### Fractions Category

#### 1. Decimals to Fractions (40 questions)
- Uses `generateWeightedFraction(rng)` helper function
- Displays decimals like `19.4 ` (trailing space instead of `19.40` for alignment)
- Answer format: simplified fractions (e.g., `1/2`, `3/4`, `19 2/5`)
- Whole number part: 0-99

#### 2. Fractions to Decimals (24 questions)
- Uses same `generateWeightedFraction(rng)` helper
- Question format: MathML fractions (e.g., proper rendering of 1/2, 19¾)
- Answer format: decimals to 2 decimal places (e.g., `0.50`, `19.75`)
- Whole number part: 0-99
- Uses MathML for beautiful fraction rendering
- 26px spacing for taller MathML elements

#### 3. Simplify Fractions (30 questions)
- Weighted denominators: [2, 4, 5, 6, 8, 10, 12, 15, 20, 25, 30, 50, 100]
- Generates non-simplified fractions that require simplification
- 30% chance of improper fractions
- Answers include mixed numbers when appropriate (e.g., `2 1/3`)
- Falls back to composite denominators if prime selected
- MathML rendering

#### 4. Improper to Mixed Numbers (30 questions)
- Weighted denominators: [2, 3, 4, 5, 6, 8, 10, 12]
- Generates improper fractions (numerator > denominator)
- Whole parts range from 1-5
- Answer format: mixed numbers (e.g., `3 2/5`)
- MathML rendering

#### 5. Add & Subtract Fractions (20 questions)
- Common denominators: [2, 3, 4, 5, 6, 8, 10]
- 40% chance of mixed numbers in operands
- 50/50 split between addition and subtraction
- Automatically simplifies results
- Ensures subtraction results are positive
- MathML rendering with proper operators (+, -, =)
- 26px spacing

### Arithmetic Category

#### 6. Multiply & Divide by 10/100 (40 questions)
- Generates numbers with up to 2 decimal places
- Operations: ×10, ×100, ÷10, ÷100
- Includes both positive and negative numbers (30% negative)
- Results constrained to 0.01-999 range

#### 7. Times Tables (up to 12×12) (40 questions)
- Weighted distribution favoring harder multiplication facts
- Tier-based selection (see Times Tables Weighting below)
- Question format: `7 × 8 =`
- Answer: integer product

#### 8. Long Addition & Subtraction (12 questions)
- 1-3 digits before decimal, 0-2 decimal places
- Positive numbers only
- 50/50 split between addition and subtraction
- Ensures subtraction results are positive
- 120px spacing for generous working space
- Designed for column method practice

#### 9. Negative Numbers (+, -, ×, ÷) (30 questions)
- Operands with absolute value 1-20
- At least one operand is always negative (~33% first only, ~33% second only, ~33% both)
- All four operations: addition, subtraction, multiplication, division
- Division always produces integer results (divisor 2-10, dividend ≤ 20)
- Negative numbers displayed in parentheses: `(-5) + 3 =`

## Key Helper Functions

### `generateWeightedFraction(rng)`
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
Tier-based distribution:
- **Tier 1 (15%)**: Easy - 1, 2, 10, 11
- **Tier 2 (25%)**: Moderate - 3, 4, 5
- **Tier 3 (45%)**: Challenging - 6, 7, 8, 9 (most practice here!)
- **Tier 4 (15%)**: Advanced - 12

One operand uses weighted selection, the other is uniform 1-12. Order is randomly swapped.

### `createMathMLFraction(wholePart, numerator, denominator)`
Generates MathML markup for beautiful fraction rendering:
- Handles mixed numbers (whole part + fraction)
- Returns proper MathML xmlns structure
- Browser renders natively for crisp display

## UI Features

### Controls
- **Worksheet Type Selector**: Categorized dropdown with optgroups
  - **Fractions**: 5 types (conversions, simplify, improper, add/subtract)
  - **Arithmetic**: 4 types (×÷10/100, times tables, long add/subtract, negative numbers)
- **Start Page**: First worksheet number to generate
- **End Page**: Last worksheet number to generate (auto-updates if < start page)
- **Questions per Sheet**: Dynamically set based on worksheet type
  - Updates automatically when type changes via `updateDefaultQuestions()`

### Page Syncing
`syncEndPage()` function ensures end page ≥ start page automatically

### Buttons
- **Generate Worksheets**: Creates worksheets for specified page range
- **Print**: Opens browser print dialog (appears after generation)

## HTML/CSS Rendering

### `createWorksheet(worksheetId, questions, isAnswerKey, title, worksheetType)` function
- Creates DOM elements instead of drawing on canvas
- A4-sized pages (210mm × 297mm)
- Sets `data-worksheet-type` attribute for type-specific CSS
- Two-column layout using CSS `column-count: 2`
- Monospace font for questions to maintain alignment
- Modern sans-serif font for headers

### Layout
- Questions arranged in 2 columns via CSS columns
- Column gap: 25px
- Type-specific spacing via CSS selectors:
  - **Regular tasks** (multiply-divide, times-tables, decimal-to-fraction): 16px
  - **Fraction tasks** (fraction-to-decimal, simplify, improper-to-mixed, add-subtract): 26px
  - **Long addition/subtraction**: 120px (for working space)

### MathML Styling
- Base font size: 21px (for MathML container)
- Whole number: 23px (slightly larger)
- Numerator/denominator: 18px (close to regular text at 19px)
- 6px vertical margin around fraction line
- 4px margin-right after whole number for spacing

## Type-Specific Features

### Task-Dependent Defaults
Each generator has a `defaultQuestions` property:
- Most tasks: 30-40 questions
- Fraction-to-decimal: 24 (taller MathML)
- Add/subtract fractions: 20 (more complex)
- Long addition/subtraction: 12 (takes longer, needs space)

### Dynamic Spacing
CSS rules target `data-worksheet-type` attribute:
```css
.worksheet[data-worksheet-type="long-addition-subtraction"] .question-item {
    margin-bottom: 120px;
}
```

## How to Add a New Worksheet Type

1. **Add to categorized dropdown**:
```html
<optgroup label="Category Name">
    <option value="new-type">New Worksheet Type Name</option>
</optgroup>
```

2. **Add generator to `questionGenerators` object**:
```javascript
'new-type': {
    title: 'Display Title for Worksheet Header',
    defaultQuestions: 30,  // or appropriate number
    generate: function(rng) {
        // Your generation logic here
        // Use rng.next() for random numbers (returns 0-1)

        return {
            question: "question text or expression",
            answer: "answer (number or string)",
            isMathML: false  // set to true if question contains MathML
        };
    }
}
```

3. **Add type-specific spacing (if needed)**:
```css
.worksheet[data-worksheet-type="new-type"] .question-item {
    margin-bottom: 50px;
}
```

4. The rest is automatic - the system handles rendering, printing, defaults, etc.

## Technical Details

### SeededRandom Class
- Simple linear congruential generator
- Formula: `seed = (seed × 9301 + 49297) % 233280`
- Ensures deterministic question generation per worksheet number
- Each worksheet uses seed = `pageNumber × 12345`

### Question Format
All generators return objects with:
- `question`: String or MathML markup displayed to student
- `answer`: String or number for answer key
- `isMathML`: Boolean (optional, default false) - set true if question contains MathML

### Print Styling
- Hides controls when printing
- Forces A4 portrait layout
- Removes borders/shadows from worksheets
- Page break after each worksheet
- `@page { size: A4 portrait; }`

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

### Why HTML/CSS rendering (not canvas)?
- Better text rendering quality
- Native MathML support
- More maintainable code (~200 lines less than canvas)
- Accessibility (screen readers can read content)
- Selectable text if needed
- Easier to style and modify

### Why seeded randomization?
- Teachers can assign specific worksheet numbers
- Students can't "game" the system by regenerating
- Reproducible for answer keys
- Same worksheet number = same questions forever

### Why MathML for fractions?
- Professional, native browser rendering
- Crisp display at any zoom level
- Proper mathematical notation
- Better than plain text like "3/4"

## Future Enhancement Ideas
- Add answer key toggle (code structure already supports it, commented out in generateWorksheets)
- More worksheet types (percentages, ratios, order of operations, etc.)
- Configurable difficulty levels
- Custom question ranges per type
- Dark mode for on-screen viewing
- Multiplication/division with larger numbers
- Mixed operations worksheets

## Recent Major Changes

### 2025-01 (Latest)
- Refactored from canvas to HTML/CSS rendering
- Added MathML support for fractions
- Added 4 new worksheet types (simplify fractions, improper-to-mixed, add/subtract fractions, long addition/subtraction)
- Implemented task-dependent default question counts
- Added type-specific spacing system
- Organized dropdown with optgroup categories
- Removed redundant page footer
- Improved fraction rendering with proper sizing

### Key Benefits of Recent Changes
- **Cleaner code**: ~200 lines removed with canvas removal
- **Better fractions**: Native MathML rendering vs text
- **More variety**: 8 worksheet types vs original 4
- **Better organization**: Categorized UI with Fractions/Arithmetic groups
- **Appropriate pacing**: Each worksheet type takes similar time to complete
- **Working space**: Long addition/subtraction has generous spacing for column method
