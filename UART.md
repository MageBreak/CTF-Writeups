# NullCon CTF - UART Signal Decoding Challenge

**Challenge Name:** UART  
**Category:** Misc  
**File Provided:** `uart.sr` (Sigrok session file)  
**Objective:** Decode the signal transmission to recover the flag.  

---

## Challenge Overview

The challenge provides a `.sr` file, which is a **Sigrok session file** containing captured logic analyzer data.

Our task is to analyze the recorded digital signal and decode the UART transmission to extract the hidden flag.

---

# Phase 1: Environment Setup

Since the challenge provides a Sigrok capture file, we use the **Sigrok suite of tools**.

For this analysis:

- OS: Kali Linux (via WSL)
- GUI Tool: `PulseView`
- CLI Tool: `sigrok-cli`

---

## 1. Installation

To analyze the logic capture, install the required packages:

```bash
sudo apt update
sudo apt install pulseview sigrok-cli
```

---

## 2. Loading the Data

Navigate to the directory containing the challenge file and launch PulseView:

```bash
pulseview uart.sr &
```

This opens the graphical waveform viewer with the recorded signal.

---

# Phase 2: Signal Analysis

Upon opening the file, you should see:

- A single digital channel labeled `uart.ch1`

---

## Observing the Signal

### Idle State

The signal remains **HIGH** when no transmission occurs.

This is standard for UART (Universal Asynchronous Receiver/Transmitter), where:

- Idle = HIGH
- Start Bit = LOW

---

### Identifying Data Frames

You will observe periodic dips to LOW:

- These are **Start Bits**
- Each start bit marks the beginning of a UART frame

Each frame structure typically follows:

```
Start Bit (LOW)
8 Data Bits
Optional Parity Bit
Stop Bit (HIGH)
```

---

## Measuring Bit Width (Determining Baud Rate)

To decode properly, we must determine the transmission speed (baud rate).

1. Click the **Cursor tool** (blue flag icon in PulseView)
2. Measure the width of the narrowest pulse (one bit duration)

Typical results:

- If bit width ≈ **8.6 µs** → Baud rate ≈ **115200**
- If bit width ≈ **104 µs** → Baud rate ≈ **9600**

Most CTF UART challenges use **115200 baud**.

---

# Phase 3: Decoding the Protocol

Once the baud rate is known, we apply a protocol decoder.

---

## 1. Adding the UART Decoder

1. Click **"Add Protocol Decoder"** (yellow/green plus icon)
2. Search for: `UART`
3. Select it

---

## 2. Decoder Configuration

Click on the newly added UART decoder label and configure:

- **RX:** `uart.ch1`
- **Data Bits:** `8`
- **Parity:** None (unless specified)
- **Stop Bits:** 1
- **Baud Rate:** 115200 (or your measured rate)
- **Data Format:** ASCII

Setting the format to ASCII allows readable characters instead of hexadecimal values.

---

# Phase 4: Flag Extraction

Once configured correctly:

- PulseView displays decoded characters beneath the waveform
- The decoded text appears in the UART decoder row

---

## Steps to Retrieve the Flag

1. **Zoom out** using the mouse wheel to view the full transmission
2. Observe the decoded ASCII output
3. Look for a recognizable flag pattern such as:
   - `flag{...}`
   - `CTF{...}`
   - `ENO{...}`

---

## Result

The UART transmission reveals the "valuable" data referenced in the challenge description.

The decoded ASCII output contains the hidden flag embedded directly within the transmitted serial data.

---

# Summary

This challenge demonstrates:

- Basic UART protocol analysis
- Determining baud rate via pulse width measurement
- Using PulseView protocol decoders
- Converting raw digital waveforms into readable ASCII data

It serves as a practical introduction to hardware-level signal decoding and serial communication analysis in CTF environments.
