# 👑 Understanding QR Code Generation in C++ with `qrcodegen`

> A 20-minute walkthrough of how QR Code generation works under the hood using `qrcodegen.cpp`

---

## 📌 Introduction

- QR codes are 2D barcodes that encode data like text, URLs, etc.
- The `qrcodegen` C++ library by Project Nayuki is a compact, self-contained QR generator.
- This presentation explains the **flow** and **key parts** of the code.

---

## 🧠 Structure of the Library

The code is divided into:

- `qrcodegen.hpp`: Header file (declarations)
- `qrcodegen.cpp`: Source file (definitions)

Two core classes:

- `QrSegment`: Breaks your input into properly encoded chunks.
- `QrCode`: Converts those chunks into a scannable QR grid.

---

## 🔠 `QrSegment` – Encoding Input Data

The job of `QrSegment` is to:

- Break input into segments based on encoding type
- Convert characters to bitstreams

### Supported Modes

- `NUMERIC` — digits only (highly compressed)
- `ALPHANUMERIC` — letters + digits + symbols
- `BYTE` — any data (UTF-8, binary)
- `KANJI`, `ECI` — advanced/optional

---

## 🧠 Segment Creation Functions

Each encoding mode has a function:

```cpp
makeNumeric("12345");
makeAlphanumeric("HELLO");
makeBytes(dataBytes);
```

Each function builds:

- A mode indicator
- Character count
- Bitstream (using `BitBuffer`)

It uses the best mode based on the data.

---

## 🧼 Calculating Total Bits

Before QR generation, we compute how many bits are needed:

```cpp
QrSegment::getTotalBits(segments, version);
```

Checks if:

- Your data fits in a version
- Segment lengths are legal
- There are no overflows

Used to **select QR version** that fits your content.

---

## 🧱 `QrCode` – Building the QR

Now we switch to the `QrCode` class.

This does the **heavy lifting**:

- Lays out the grid
- Adds error correction
- Masks and finalizes the code

Core function:

```cpp
QrCode::encodeSegments(segments, QrCode::Ecc::LOW);
```

---

## 🧼 QR Version & Error Correction

The code tries multiple versions:

```cpp
for (version = min; version <= max; version++) {
    // Check if data fits
}
```

If there's room, it may **boost error correction level**.

ECC levels:

- LOW (7%)
- MEDIUM (15%)
- QUARTILE (25%)
- HIGH (30%)

---

## 🔲 Function Patterns

Before placing data, the QR grid is decorated with:

- Finder patterns (top-left, top-right, bottom-left)
- Alignment patterns
- Timing lines
- Format/version bits

Done using:

```cpp
drawFunctionPatterns();
```

This reserves space to avoid data overlapping it.

---

## 🧪 Error Correction with Reed–Solomon

The library computes ECC using:

```cpp
reedSolomonComputeRemainder();
```

It:

- Treats your data as a polynomial
- Divides it by a generator polynomial
- Returns the **remainder**, aka the ECC

Then:

```cpp
addEccAndInterleave();
```

Splits into blocks and **interleaves** the data + ECC.

---

## 🧵 Interleaving Blocks

QR requires data to be interleaved:

- Split into multiple blocks
- ECC is added to each
- Then we stitch them **column-by-column**

This increases resilience against localized damage.

---

## 🖌️ Drawing Data on the Grid

Final step:

```cpp
drawCodewords(codewords);
```

- Starts from bottom-right
- Moves in 2-column wide zigzags
- Avoids function patterns
- Places 1 bit at a time

---

## 🥸 Applying a Mask

QR masks randomize the data visually:

```cpp
applyMask(maskPattern);
```

Prevents ugly patterns like:

- Large white/black blocks
- Repeating shapes

There are 8 official mask patterns:

```
(x + y) % 2 == 0
y % 2 == 0
x % 3 == 0
... and more
```

Each mask is applied, scored, then undone.

---

## 💅 Penalty Score – Choosing the Best Mask

Each mask is evaluated using:

```cpp
getPenaltyScore();
```

Penalty is based on:

1. Long runs of same color
2. 2×2 blocks
3. False finder patterns
4. Unbalanced black/white

Lowest penalty = best mask

---

## 🎉 Final QR is Ready

Once the best mask is applied and format bits are drawn, the QR is **complete** and can be rendered or exported.

You can now:

- Convert it to ASCII
- Render as an image
- Scan with any QR reader

---

## 🧠 Summary

- `QrSegment` breaks down and encodes data
- `QrCode` builds the grid and handles ECC/masking
- Reed–Solomon math provides ECC
- Masking ensures scan-friendliness
- Penalty scores pick the best appearance

---

## 🔥 Bonus: Sample Usage

```cpp
auto qr = QrCode::encodeText("HELLO WORLD", QrCode::Ecc::LOW);
int size = qr.getSize();
for (int y = 0; y < size; y++) {
    for (int x = 0; x < size; x++)
        cout << (qr.getModule(x, y) ? "██" : "  ");
    cout << '\n';
}
```

This prints the QR code in your terminal using ASCII blocks.


