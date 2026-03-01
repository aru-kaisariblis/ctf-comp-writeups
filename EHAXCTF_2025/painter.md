Berikut adalah versi tulis ulang ( *rewrite* ) untuk *writeup* **PAINTER** dalam bahasa Inggris. Di versi ini, penjelasannya dibuat lebih mendalam secara teknis—khususnya pada bagian manipulasi *byte* dan *Little Endian*—yang akan sangat cocok untuk portofolio mahasiswa Teknik Informatika di GitHub.

Lo tinggal *copy-paste* kode di bawah ini:

```markdown
## Writeup Challenge 2: PAINTER

### Overview
This challenge involves a packet capture file named `pref.pcap`. Based on the challenge name ("PAINTER") and the nature of the captured traffic—which is predominantly USB communication—this is a Network Forensics challenge focused on analyzing USB Human Interface Device (HID) traffic, specifically reconstructing mouse movements.

### Problem Abstraction
The objective is to parse the payload data from USB packets (`URB_INTERRUPT in`) that log the precise X and Y movement coordinates and the button-click status of a mouse. By extracting these coordinates and plotting them onto a 2D plane, we can visually reconstruct the digital drawing or text that the user was creating during the packet capture.

### Vulnerability Analysis
USBHID mouse data is transmitted in plaintext. The critical payload is located within the `Leftover Capture Data` (or `usb.capdata`) field in Wireshark. The data structure typically consists of several bytes formatted as follows:
* **Byte 0 (Status):** Indicates the button state (`01` = left click pressed, `00` = no click).
* **Byte 1-2 (X-Axis Delta):** The relative movement on the X-axis, formatted as a 16-bit signed integer in Little Endian.
* **Byte 3-4 (Y-Axis Delta):** The relative movement on the Y-axis, formatted as a 16-bit signed integer in Little Endian.

### Attack Strategy
1. **Data Filtering:** Isolate and extract only the `usb.capdata` field from the PCAP file using `tshark` to generate a clean, parsable text file.
2. **Byte Parsing:** Develop a Python script to iterate through the extracted hex data line by line.
3. **Data Reconstruction:** Parse the hexadecimal values, handle the Little Endian byte ordering, and convert them into signed integers to calculate the absolute coordinates.
4. **Data Visualization:** Utilize the `matplotlib` library to render a scatter plot, plotting coordinates exclusively when Byte 0 indicates the left mouse button is held down.

### Implementation

**Step 1: Data Extraction via `tshark`**
We dump the raw hex payload into a text file for efficient processing:
```bash
tshark -r pref.pcap -Y "usb.capdata" -T fields -e usb.capdata > mouse_data.txt

```

**Step 2: Python Plotting Script**
The following script processes the hex dump, calculates the absolute positioning, and visualizes the result:

```python
import matplotlib.pyplot as plt

def get_signed_16bit(hex_str):
    """Converts a Little Endian hex string to a 16-bit signed integer."""
    # Reverse byte order for Little Endian (e.g., 'feff' -> 'fffe')
    val = int(hex_str[2:4] + hex_str[0:2], 16)
    # Handle signed integer (Two's complement)
    if val >= 0x8000:
        val -= 0x10000
    return val

with open('mouse_data.txt', 'r') as f:
    lines = f.readlines()

x, y = 0, 0
X_coords, Y_coords = [], []

for line in lines:
    line = line.strip().replace(':', '')
    if len(line) < 14: 
        continue
    
    # Extract states
    button = int(line[0:2], 16)
    dx = get_signed_16bit(line[4:8])
    dy = get_signed_16bit(line[8:12])
    
    # Accumulate coordinates
    x += dx
    y -= dy # Invert Y-axis to match visual screen coordinates
    
    # Plot only when the left mouse button is dragging (drawing)
    if button == 1:
        X_coords.append(x)
        Y_coords.append(y)

# Render the plot
plt.figure(figsize=(10, 5))
plt.scatter(X_coords, Y_coords, s=1, c='black')
plt.axis('equal')
plt.title("Reconstructed Mouse Trajectory")
plt.show()

```

### Result

Executing the Python script launches a `matplotlib` window revealing the hidden sketch. The reconstructed mouse trajectory clearly spells out `Wh4t_c0l0ur_15_th3_fl4g`.

Following the challenge's flag format specification, the final flag is constructed.

**Flag:** `EH4X{Wh4t_c0l0ur_15_th3_fl4g}`

### Lessons Learned

* Deepened understanding of the USBHID packet structure and data payload formatting.
* Mastered using `tshark` for automated, CLI-based packet extraction, bypassing manual GUI constraints.
* Applied low-level data manipulation in Python, specifically handling Little Endian architecture and Two's Complement signed integers for forensic reconstruction.

```

Versi ini terasa lebih rapi dan menggunakan istilah-istilah yang lebih akademik (*Two's complement*, *Little Endian architecture*, *Data visualization*). 

Apakah ada bagian tertentu dari teks di atas yang mau diubah lagi, atau mau lanjut bahas *challenge* yang lain?

```