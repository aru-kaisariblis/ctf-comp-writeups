Sip! Karena ini buat ditaruh di portofolio GitHub, gue bantu rapikan *formatting*-nya supaya konsisten dengan dua *writeup* sebelumnya. Gue juga nambahin *placeholder* gambar ilustrasi gelombang (*waveform*) supaya pembaca di GitHub bisa lebih gampang kebayang bentuk datanya.

Tinggal *copy-paste* teks di dalam kotak hitam ini ke file `README.md` lo:

```markdown
## Writeup Challenge 3: Power Analysis Forensics

### Overview
This challenge provides a CSV file containing power consumption traces recorded during the execution of a secret-dependent computation. The goal is to recover a hidden secret using **power side-channel analysis**, then compute the final flag in the format: `EHAX{SHA256(secret)}`.

The key hint given in the challenge is:
> *"Power reveals the secret."*

This indicates that the solution relies on analyzing **power leakage**, not cryptographic weaknesses.

### Problem Abstraction
The dataset consists of the following fields:
* `position` – index of the secret digit (0–5)
* `guess` – candidate digit (0–9)
* `trace_num` – trace identifier
* `sample` – time index
* `power_mW` – measured power consumption

Each `(position, guess)` pair produces a **power waveform** over time. The task is to determine which guess at each position corresponds to the **true secret digit** by analyzing these waveforms.



### Vulnerability Analysis
The system is vulnerable to **power side-channel leakage**. Although all guesses execute the same operation, the **correct guess causes different internal switching activity**, leading to:
* Higher instantaneous power consumption.
* A **distinct peak** during the cryptographic operation.

This leakage is **data-dependent** and occurs at a consistent time offset, making it observable after averaging multiple traces. The vulnerability lies in the lack of power masking and constant-power execution during the cryptographic routine.

### Attack Strategy
The attack follows a **Simple Power Analysis (SPA)** approach:
1. For each secret position, group the traces by `(guess, sample)`.
2. Compute the **mean waveform** for each guess to eliminate random noise.
3. Identify the **time of the cryptographic event** (visible as a distinct power spike).
4. Select the guess whose waveform shows the **highest peak at that event**.
5. Repeat for all 6 positions to reconstruct the full secret.
6. Hash the recovered secret using SHA-256 to construct the final flag.

**Key Principle:**
> The leaked information is found in the **waveform shape**, not in individual, unaligned CSV values.

### Implementation
The following Python script automates the trace averaging and peak detection process:

```python
import pandas as pd
import hashlib

def solve_power_analysis(file_path):
    df = pd.read_csv(file_path)
    secret_string = ""

    for pos in range(6):
        pos_data = df[df['position'] == pos]

        # Build mean waveform per guess to eliminate noise
        mean_traces = (
            pos_data
            .groupby(['guess', 'sample'])['power_mW']
            .mean()
            .unstack()
        )

        # Find peak power for each guess
        max_power_per_guess = mean_traces.max(axis=1)

        # Select the guess with the highest peak
        best_guess = max_power_per_guess.idxmax()
        secret_string += str(best_guess)

    # Hash the recovered 6-digit PIN
    sha256_hash = hashlib.sha256(secret_string.encode()).hexdigest()
    return f"EHAX{{{sha256_hash}}}"

if __name__ == "__main__":
    print(solve_power_analysis("power_traces.csv"))

```

*Note: This implementation is highly effective because it eliminates noise via averaging, preserves temporal alignment across samples, and correctly extracts the secret digits as a string before hashing.*

### Result

The analysis successfully recovers a **6-digit secret PIN** from the power waveforms. After hashing the recovered secret with SHA-256, the correct flag is obtained. The attack succeeds purely by exploiting physical leakage without breaking the underlying cryptography.

**Flag:** `EHAX{5bec84ad039e23fcd51d331e662e27be15542ca83fd8ef4d6c5e5a8ad614a54d}`

### Lessons Learned

* **Side-channel attacks target implementations, not algorithms.** Even a mathematically secure cipher can be broken if its physical execution leaks data.
* Raw maximum values are unreliable; **waveform analysis is essential.**
* Time alignment is critical in signal-based forensics.
* Averaging traces dramatically improves signal clarity and signal-to-noise ratio (SNR).
* Visual inspection of waveforms can often outperform naive automation when identifying points of interest.
* Power analysis is a practical, real-world threat when countermeasures (like power masking or constant-time code) are absent.

```