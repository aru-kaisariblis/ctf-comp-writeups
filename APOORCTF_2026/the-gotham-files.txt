# The Gotham Files

## Overview
This challenge falls under the Steganography category. We are provided with a single image file named challenge.png, which depicts a comic book panel featuring Batman, Supergirl, and Batgirl. The challenge description states: "A mysterious panel surfaced at this year's ComiCon. The artist left something behind," along with the author's name, n3twraith.

## Problem Abstraction

The primary objective is to extract a hidden string (the flag) embedded within the structural data of the PNG image—specifically within the Red channel values of the image's Color Palette (PLTE chunk)—based on hints discovered in the file's metadata.

## Vulnerability Analysis
Initial analysis of the file using standard enumeration tools revealed a crucial clue. Extracting the metadata showed a comment: "The Collector. Not all colors make it to the page. In Gotham, only the red light tells the truth." Furthermore, running zsteg revealed the presence of a chunk:1:PLTE, indicating this is an indexed color image rather than a standard RGB image. The hint heavily implied that the secret data was not hidden in standard pixel LSBs, but rather specifically within the "red" data of the "colors that make it to the page" (the palette).

## Attack Strategy

1.  **Metadata Enumeration:** Extract strings and metadata to look for initial clues or author intentions.
2.  **Standard Stego Checks:** Verify if standard LSB techniques on the Red channel yield any results (to rule out simple visual steganography).
3.  **Palette Data Extraction:** Write a custom script to extract the RGB palette from the PLTE chunk.
4.  **Data Decoding:** Isolate only the Red values from the extracted palette and convert those byte values into ASCII characters to reveal the flag.

## Implementation

### Phase 1: Metadata and Initial Analysis
The first step was to check the metadata using `exiftool`.

```bash
exiftool -b challenge.png
```

This revealed the critical hint about the "red light" and "not all colors". Next, `zsteg` was run to check for basic LSBs and file structure. `zsteg` confirmed the image used a PLTE chunk but outputted gibberish for the LSB payloads.

### Phase 2: Ruling out Visual LSB
To be thorough, I extracted the Red bitplanes (Plane 0, Plane 1, etc.) to see if the flag was drawn visually.

!Visual Noise

The result was purely visual noise, confirming the data was not hidden in the pixel grid itself, but structurally.

### Phase 3: Extracting the Red Palette (The Payload)
Based on the hints, I wrote a Python script using the Pillow library to extract the image's color palette. In a PNG palette, colors are stored sequentially as `[R, G, B, R, G, B...]`. The script extracts every 3rd value starting from index 0 (the Red values) and converts them to ASCII.

```python
from PIL import Image

# Open the image
img = Image.open('challenge.png')

# Extract the color palette data
palette = img.getpalette()

if palette:
    # Isolate HANYA Red values (index 0, 3, 6, 9, etc.)
    red_values = palette[0::3]

    # Convert byte values to ASCII text
    flag = ""
    for r in red_values:
        if 32 <= r <= 126: # Only printable characters
            flag += chr(r)

    print("[+] Red Palette Extraction:")
    print(flag)
else:
    print("[-] No color palette found.")
```

## Result

Running the Python script successfully parsed the PLTE chunk, isolated the red byte values, and decoded them into a readable string. The output directly revealed the flag hidden alongside some junk data.

!Flag Output

**Flag:** `apoorvctf{th3_c0m1cs_l13_1n_th3_PLTE}`

## Lessons Learned

*   **Read the Hints Carefully:** Steganography is often a guessing game without hints. The phrase "Not all colors make it to the page" was a brilliant pointer toward Indexed Colors / PLTE chunks rather than standard RGB pixels.
*   **Look Beyond Pixel LSBs:** Tools like `zsteg` and `stegsolve` are great for visual/pixel-based steganography, but CTF authors often hide data in the file structure itself (like the palette array). Knowing how to manipulate image data structures with Python (PIL) is an essential skill.
