# Routine Checks

## Overview
This challenge provides a packet capture file (challenge.pcap) containing network traffic. The challenge description mentions "routine system checks" and notes that while most messages are ordinary status updates, "one message stands out as... different." The challenge falls under the Network Forensics and Steganography categories, requiring participants to extract hidden payloads from the network stream and uncover nested secrets.

## Problem Abstraction
The main objective is to identify anomalous data exfiltration within the network traffic, extract the fragmented payload, repair the intentionally corrupted file header to reconstruct the original file, and finally perform steganography analysis to bypass a "rabbit hole" and retrieve the actual flag.

## Vulnerability Analysis
Initial analysis of the .pcap file in Wireshark reveals TCP traffic. By inspecting the payload of the TCP packets, a distinct hexadecimal pattern is found: 3F D8 FF E0 00 10 4A 46 49 46.
The ASCII representation JFIF strongly indicates a JPEG image file. However, a standard JPEG magic header should start with FF D8 FF E0. The first byte has been intentionally modified from FF to 3F to corrupt the file and prevent straightforward extraction. Furthermore, the reconstructed image presents a QR code containing a fake flag, indicating a secondary layer of concealment using Least Significant Bit (LSB) steganography.

## Attack Strategy

1. Open the challenge.pcap file using Wireshark.
2. Locate the TCP packet containing the JFIF string in its payload.
3. Use the Follow TCP Stream feature to reassemble the fragmented image data and export it as raw bytes.
4. Utilize a Hex Editor to repair the corrupted magic byte (3F -> FF).
5. Open the repaired image to reveal a QR code (Rabbit Hole).
6. Use Steghide to extract hidden data embedded within the image using an empty passphrase.

## Implementation

### Network Analysis & Extraction
In Wireshark, the TCP stream containing the JPEG data is identified. By right-clicking the packet and selecting Follow -> TCP Stream, the entire payload is aggregated.

![Screenshot: Tampilan Wireshark saat melakukan Follow TCP Stream dan mengubah opsi ke 'Raw'](path/to/wireshark_tcp_stream.png)

The display format is changed to Raw and the data is saved as flag_corrupted.jpg.

### Header Repair
Using a hex editor in the terminal, the corrupted first byte is fixed to restore the JPEG format.

```bash
hexeditor flag_corrupted.jpg
# Change the first byte from 3F to FF, then save as flag.jpg
```

![Screenshot: Tampilan Hexeditor menunjukkan perubahan byte 3F menjadi FF](path/to/hexeditor_fix.png)

### Steganography Extraction
Opening flag.jpg reveals a QR code. Scanning it yields apoorvctf{this_aint_it_brother}, which is a fake flag.

![Screenshot: Gambar QR Code dari file flag.jpg yang sudah diperbaiki](path/to/qrcode_image.png)

Assuming the real flag is hidden via LSB steganography, steghide is executed against the repaired image without a passphrase.

```bash
steghide extract -sf flag.jpg
# Enter passphrase: (leave blank and press Enter)
```

![Screenshot: Output terminal saat menjalankan perintah steghide dan berhasil mengekstrak file rahasia](path/to/steghide_output.png)

## Result

Steghide successfully extracts the hidden message embedded in the lowest bits of the image. The real flag is revealed.

**Flag:** `apoorvctf{b1ts_wh1sp3r_1n_th3_l0w3st_b1t}`

## Lessons Learned

* The importance of understanding file signatures (Magic Bytes) to manually identify and repair corrupted files during data exfiltration analysis.
* The capability of Wireshark's "Follow TCP Stream" to automatically reassemble fragmented payloads.
* Awareness of CTF "rabbit holes" (fake flags) and the necessity to apply further steganography techniques like LSB (Steghide) when encountering suspicious media files.
