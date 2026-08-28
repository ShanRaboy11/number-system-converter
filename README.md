# Number System Converter

A browser-based number system converter for converting values among binary, octal, decimal, and hexadecimal representations. The application is implemented as a self-contained HTML file with embedded CSS and JavaScript.

## System Requirements

### Hardware

- Any computer, tablet, or phone capable of running a modern web browser.
- No special hardware or calculator is required.

### Software

- A modern browser with support for:
  - JavaScript ES2020 or later
  - `BigInt`
  - `String.prototype.startsWith`
  - DOM event handling
  - Clipboard API for copying results when permitted by the browser
- Examples of compatible browsers include current versions of Chrome, Edge, Firefox, and Safari.
- No server, database, package installation, or internet connection is required.
- To run the program, open `app.html` in a browser.

## Supported Number Systems

| Number system | Base | Accepted digits |
|---|---:|---|
| Binary | 2 | `0-1` |
| Octal | 8 | `0-7` |
| Decimal | 10 | `0-9` |
| Hexadecimal | 16 | `0-9`, `A-F`, or `a-f` |

A leading minus sign and one radix point (`.`) are accepted for every base. Examples of valid forms include `10.5`, `1010.1`, `2A.8`, `10.`, and `.5`. Leading and trailing spaces are removed before validation and conversion.

## Algorithm / Pseudocode

The application converts an input in two stages: the source value is converted to an exact rational value using two `BigInt` values, then that rational value is converted to each target base.

### Main conversion algorithm

```text
START

Create a minimum of three input rows.

When the user types a value or changes its selected base:
    Read the input text and selected source base.
    Remove leading and trailing whitespace.

    IF the input is empty:
        Clear the error message and all result fields.
        STOP processing this row.

    Validate the input against the selected base:
        Binary      -> optional '-' followed by digits 0 or 1
        Octal       -> optional '-' followed by digits 0 through 7
        Decimal     -> optional '-' followed by digits 0 through 9
        Hexadecimal -> optional '-' followed by digits 0 through 9 or A through F

    IF validation fails:
        Mark the input as invalid.
        Display the appropriate error message.
        Clear all result fields.
        STOP processing this row.

    Convert the valid source string to an exact rational value:
        Handle a leading '-' separately.
        Split the input at the radix point.
        Join the whole and fractional digits.
        Set numerator = 0.
        FOR each joined digit from left to right:
            Convert the digit to its numeric value.
            numerator = numerator * source base + digit value.
        Set denominator = source base ^ number of fractional digits.
        Apply the negative sign to the numerator if necessary.

    For each target base in [2, 8, 10, 16]:
        Convert the rational value to the target base:
            Separate the sign and use the absolute numerator.
            Convert numerator / denominator to the integer part
            using repeated division and remainders.
            Set remainder = numerator modulo denominator.
            WHILE remainder is not zero and fewer than 32 fraction digits exist:
                Multiply remainder by the target base.
                The quotient is the next fraction digit.
                Keep the new remainder after division.
            Append "..." when the fraction is still repeating after 32 digits.
            Restore the negative sign if necessary.
        Display the converted value.

END
```

### Input-row management algorithm

```text
When Add Input is clicked:
    Create a new input row.
    Add it to the page.
    Renumber every row.

When a Remove button is clicked:
    IF there are only three rows:
        Do nothing.
    ELSE:
        Remove the selected row.
        Renumber every row.

When Clear Values is clicked:
    Empty every input field.
    Clear every error and result field.

When a result is clicked:
    Expand the result to its full generated value.
    Copy the full generated value to the clipboard.
    Collapse it again if clicked a second time.
    Briefly display "Copied" when copying succeeds.
```

## Flowchart

```mermaid
    A([Start]) --> B[Enter value and select source base]
    B --> C{Valid input?}
    C -->|No| D[Show error and clear results]
    D --> B
    C -->|Yes| E[Convert to exact rational BigInt]
    E --> F[Convert to bases 2, 8, 10, and 16]
    F --> G[Show five-digit fractional previews]
    G --> H{Result clicked?}
    H -->|Yes| I[Show full value and copy it]
    H -->|No| B
    I --> B
```

The `Add Input`, `Remove`, `Clear Values`, and theme controls operate independently of the conversion path. Removing a row uses a brief slide-and-fade animation, and the application always keeps at least three rows.

## Program Implementation

### File structure

```text
number-system-converter/
├── app.html
└── README.md
```

### User interface

The interface is contained in `app.html` and includes:

- A responsive converter workspace.
- Student information fields for the name and course.
- A light/dark theme toggle.
- A subtle transparent grid background.
- Three input rows displayed initially.
- Base selector buttons: `BIN`, `OCT`, `DEC`, and `HEX`.
- Four result fields per row.
- `Add Input`, `Remove`, and `Clear Values` controls.

### Validation implementation

The `BASE_INFO` object defines the accepted pattern for each base:

```javascript
const BASE_INFO = {
    2:  { name: "Binary",      pattern: /^-?(?:[01]+(?:\.[01]*)?|\.[01]+)$/i },
    8:  { name: "Octal",       pattern: /^-?(?:[0-7]+(?:\.[0-7]*)?|\.[0-7]+)$/i },
    10: { name: "Decimal",     pattern: /^-?(?:[0-9]+(?:\.[0-9]*)?|\.[0-9]+)$/i },
    16: { name: "Hexadecimal", pattern: /^-?(?:[0-9A-Fa-f]+(?:\.[0-9A-Fa-f]*)?|\.[0-9A-Fa-f]+)$/i }
};
```

The `validateInput()` function rejects characters that are not legal for the selected base. Empty input is treated as a cleared row rather than an error.

### Conversion implementation

- `baseStringToRational(rawValue, base)` converts a string from its selected base into an exact `{ numerator, denominator }` rational value.
- `rationalToBaseString(rational, base)` converts the rational value using integer division and repeated multiplication of the remainder.
- `BigInt` is used instead of JavaScript `Number`, allowing exact integer and finite-fraction conversion beyond the normal safe integer limit.
- Results show at most five digits after the radix point in their normal collapsed state.
- Clicking a result expands it to the full generated value. Repeating output is limited to 32 digits and ends with `...`.
- The `DIGITS` string, `0123456789ABCDEF`, supplies output symbols for all supported bases.
- `updateRow()` coordinates validation, conversion, error display, and result rendering.

### Event handling

The program uses event delegation on the input container. This allows input, base-selection, result-copy, and remove actions to work for rows created after the initial page load.

## Test Cases

The following cases can be tested directly in a browser by entering a value, selecting its source base, and checking all four result fields.

| Test | Input | Source base | Expected binary | Expected octal | Expected decimal | Expected hexadecimal | Result |
|---:|---|---:|---|---|---:|---|---|
| 1 | `42` | 10 | `101010` | `52` | `42` | `2A` | Pass |
| 2 | `101010` | 2 | `101010` | `52` | `42` | `2A` | Pass |
| 3 | `52` | 8 | `101010` | `52` | `42` | `2A` | Pass |
| 4 | `2A` | 16 | `101010` | `52` | `42` | `2A` | Pass |
| 5 | `-15` | 10 | `-1111` | `-17` | `-15` | `-F` | Pass |
| 6 | `0` | 10 | `0` | `0` | `0` | `0` | Pass |
| 7 | `10.5` | 10 | `1010.1` | `12.4` | `10.5` | `A.8` | Pass; fractional input |
| 8 | `-2A.8` | 16 | `-101010.1` | `-52.4` | `-42.5` | `-2A.8` | Pass; negative fraction |
| 9 | `10201` | 2 | Blank | Blank | Blank | Blank | Error shown: invalid binary number; results are cleared |
| 10 | `1G` | 16 | Blank | Blank | Blank | Blank | Error shown: invalid hexadecimal number; results are cleared |

The invalid-input cases produce blank result fields, not the text `Not generated`.

The large-integer behavior is covered by the implementation because all arithmetic uses `BigInt`; it can also be checked by entering any integer larger than JavaScript's safe integer limit.

The main UI behavior is covered during normal testing: the page starts with three rows, `Add Input` adds a row, `Remove` animates and removes a row only when more than three exist, `Clear Values` clears all rows, and clicking a populated result expands and copies it.

## Sample Output

### Sample 1: Decimal input

Input:

```text
Source base: Decimal
Value: 42
```

Displayed results:

```text
Binary:      101010
Octal:       52
Decimal:     42
Hexadecimal: 2A
```

### Sample 2: Hexadecimal input

Input:

```text
Source base: Hexadecimal
Value: 2A
```

Displayed results:

```text
Binary:      101010
Octal:       52
Decimal:     42
Hexadecimal: 2A
```

### Sample 3: Invalid binary input

Input:

```text
Source base: Binary
Value: 10201
```

Displayed validation output:

```text
Invalid Binary number. Allowed digits: 0-1.
```

The result fields remain blank until the input is corrected.

### Sample 4: Fractional decimal input

Input:

```text
Source base: Decimal
Value: 10.5
```

Displayed results:

```text
Binary:      1010.1
Octal:       12.4
Decimal:     10.5
Hexadecimal: A.8
```

### Sample 5: Long fractional output

Input:

```text
Source base: Decimal
Value: 1.123456
```

Collapsed result preview:

```text
Decimal: 1.12345...
```

After clicking the Decimal result, the full finite value is displayed and copied:

```text
Decimal: 1.123456
```

For a repeating conversion, the expanded output can look like this:

```text
Binary: 1.00011001100110011001100110011001...
```

## How to Run

1. Open the project folder.
2. Open `app.html` in a modern web browser.
3. Select the source base for an input row.
4. Enter a valid value.
5. Read the four generated representations.
6. Click a result to copy it when needed.

## Limitations and Notes

- Fractional values are supported using exact rational arithmetic. Collapsed results show up to five fractional digits and use `...` when more digits exist.
- Clicking a result expands it to the full generated value, up to 32 fractional digits; repeating values end with `...`.
- Prefixes such as `0b`, `0o`, and `0x` are not accepted because the validator expects digits only, with an optional leading minus sign.
- Theme selection follows the browser's current color-scheme preference when the page loads; it is not persisted after the page is closed.
- Clipboard copying depends on browser permission and support for `navigator.clipboard`.
