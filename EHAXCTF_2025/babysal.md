## Babyserial Forensic

### Overview
This challenge provides a capture file from a logic analyzer named `babyserial.sal`. The challenge falls under the Hardware or Forensics category, where participants are required to analyze raw digital signals to extract hidden information.

### Problem Abstraction
The main objective is to find the active digital channel from the logic analyzer recording, identify the communication protocol used, decode the signal into a sequence of bytes (Hex/ASCII data), and reconstruct it into the original payload file containing the flag.

### Vulnerability Analysis
Given the challenge name `babyserial`, the vulnerability/attack vector centers around extracting unencrypted Asynchronous Serial (UART) communication. Initial analysis shows that data is transmitted on **Channel D0**. After decoding the signal, a set of Hexadecimal values is revealed, which, when translated to ASCII, forms the string `iVBORw0KGgo...`. In the cybersecurity realm, this string is widely recognized as the magic header of a **PNG** image file encoded in **Base64**.

### Attack Strategy
1. Open the `.sal` file using the **Saleae Logic 2** application.
2. Apply the **Async Serial** analyzer on Channel D0, which shows signal activity.
3. Export the decoded analyzer results into a CSV format.
4. Create an automation script to read the Hex values from the CSV, convert them into a Base64 string, decode the Base64, and save it as a `.png` file.

### Implementation
* **Signal Analysis:** Using Logic 2, the Async Serial Analyzer is set on input **D0**.
* **Extraction:** Data is exported using the *Export Analyzer Results* feature to `serial_data.csv`.
* **Scripting:** A Python script is used to parse the text:

```python
import re, base64

# Read CSV data and extract hex values
with open('serial_data.csv', 'r') as f:
    hex_values = re.findall(r'0x([0-9A-Fa-f]{2})', f.read())

# Convert Hex to ASCII (Base64 String)
base64_string = "".join([chr(int(h, 16)) for h in hex_values])

# Decode Base64 and save as PNG
with open('flag.png', 'wb') as img_file:
    img_file.write(base64.b64decode(base64_string))
```
### Result

A pink-colored image file is successfully extracted and can be opened perfectly. The image clearly displays the flag text.

![alt text](flag.png)

Flag: EH4X{baby_U4rt}
Lessons Learned

* Familiarization with hardware analysis tools like Saleae Logic 2.
* The importance of recognizing file signatures or magic bytes, especially their Base64 encoded versions (iVBORw... for PNG).
* The ability to automate the extraction and conversion of large amounts of data (>9000 lines) using Python scripting.