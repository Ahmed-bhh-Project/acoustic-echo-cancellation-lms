# 🔇 Acoustic Echo Cancellation with LMS Filter

> A MATLAB implementation of an **Acoustic Echo Cancellation (AEC)** system using the **LMS (Least Mean Squares)** adaptive filter to isolate clean near-end speech from an echo-corrupted dialogue signal.

---

## 📋 Table of Contents

- [Overview](#overview)
- [How It Works](#how-it-works)
- [LMS Algorithm](#lms-algorithm)
- [File Structure](#file-structure)
- [Getting Started](#getting-started)
- [Parameters](#parameters)
- [Signal Pipeline](#signal-pipeline)
- [Results](#results)
- [Technologies Used](#technologies-used)

---

## Overview

In real-time communication systems (video calls, speakerphones, VoIP), the far-end speaker's voice is played through a loudspeaker and picked up by the microphone alongside the near-end speaker's voice — this is **acoustic echo**. Without cancellation, the far-end speaker hears their own voice echoed back with a delay.

This project simulates this scenario and eliminates the echo using an **adaptive LMS filter** that continuously learns and subtracts the estimated echo from the mixed signal.

---

## How It Works

The system models the following communication scenario:

```
Far-end speaker (s1.wav)
        │
        ▼
  Acoustic Room (Rep.dat)        ← Impulse response of the room
        │
        ▼
  Filtered far-end signal        ← Simulated echo
        │
        + ← Near-end speaker (s2.wav)
        │
        ▼
  Mixed dialogue signal          ← What the microphone captures
        │
        ▼
  LMS Adaptive Filter            ← Estimates and removes the echo
        │
        ▼
  Clean near-end speech ✅
```

The **LMS filter** takes the far-end signal as its reference input and the mixed dialogue as the desired signal. By minimizing the error between its output and the desired signal, it adaptively estimates the room impulse response and cancels the echo component.

---

## LMS Algorithm

The `algolms` function implements the standard LMS update rule:

```matlab
function [w, y, e] = algolms(x, d, P, mu)
```

| Parameter | Description |
|---|---|
| `x` | Reference input signal (far-end / near-end speech) |
| `d` | Desired signal (mixed dialogue with echo) |
| `P` | Filter order (number of taps) |
| `mu` | Step size / learning rate |
| `w` | Output filter coefficients |
| `y` | Filter output (estimated echo) |
| `e` | Error signal (cleaned speech) |

At each time step `n`, the algorithm:

1. **Forms the input vector**: `x_vec = x(n:-1:n-P+1)` — the last `P` samples
2. **Computes the filter output**: `y(n) = w' * x_vec`
3. **Computes the error**: `e(n) = d(n) - y(n)`
4. **Updates the filter weights**: `w = w + μ · e(n) · x_vec`

This gradient descent update steers the filter weights toward the true acoustic channel impulse response, effectively learning to predict and subtract the echo.

---

## File Structure

```
acoustic-echo-cancellation-lms/
│
├── Algo_LMS.txt        # LMS adaptive filter function (MATLAB)
├── main.txt            # Main script — full processing pipeline (MATLAB)
├── s1.wav              # Far-end speaker audio signal
├── s2.wav              # Near-end speaker audio signal
├── Rep.dat             # Room acoustic impulse response
└── Présentation.pdf    # Project presentation slides
```

> **Note:** Rename `Algo_LMS.txt` → `algolms.m` and `main.txt` → `main.m` before running in MATLAB.

---

## Getting Started

### Prerequisites

- MATLAB (R2018b or later recommended)
- Signal Processing Toolbox (for `filter`)
- Audio Toolbox (for `audioread` / `sound`)

### Setup

1. **Clone the repository**

```bash
git clone https://github.com/Ahmed-bhh-Project/acoustic-echo-cancellation-lms.git
cd acoustic-echo-cancellation-lms
```

2. **Rename the script files**

```
Algo_LMS.txt  →  algolms.m
main.txt      →  main.m
```

3. **Open MATLAB** and set the working directory to the project folder.

4. **Run the main script**

```matlab
run('main.m')
```

---

## Parameters

The following parameters are defined at the top of `main.m` and can be tuned:

| Parameter | Value (default) | Description |
|---|---|---|
| `P` | `300` | Filter order — number of LMS taps |
| `mu` | `0.001` | Step size — controls convergence speed vs. stability |
| Playback gain | `×5` | Amplification applied to cleaned signal for playback |

**Tuning tips:**
- A **larger `P`** captures longer room impulse responses but increases computation.
- A **smaller `mu`** gives more stable convergence but slower adaptation. Values in `[0.0001, 0.01]` are typical.
- If the output sounds distorted, reduce `mu`. If convergence is too slow, increase it slightly.

---

## Signal Pipeline

```
1. Load audio signals
   ├── s1.wav  →  y_farspeech   (far-end speaker)
   └── s2.wav  →  y_nearspeech  (near-end speaker)

2. Simulate acoustic echo
   └── filter(Rep, 1, y_farspeech)  →  filtered_farspeech_signal

3. Mix signals (simulate microphone capture)
   └── y_nearspeech + filtered_farspeech_signal  →  dialogue_filtered

4. Apply LMS filter
   └── algolms(y_nearspeech, dialogue_filtered, 300, 0.001)
       ├── w          →  Learned filter coefficients
       ├── y_cleaned  →  Estimated echo (subtracted component)
       └── e          →  Clean near-end speech ✅

5. Visualize
   ├── Plot 1: Original near-end signal (s2.wav)
   └── Plot 2: Signal after echo cancellation
```

---

## Results

After running the script, MATLAB displays a figure with two subplots:

**Subplot 1 — Original near-end signal (`s2.wav`)**
The raw near-end speech before any processing.

**Subplot 2 — Signal after echo cancellation**
The LMS-filtered output where the far-end echo has been adaptively estimated and removed, leaving the near-end speech isolated.

The cleaned audio is also played back via `sound(y_cleaned * 5, Fs_nearspeech)`.

---

## Technologies Used

| Tool | Purpose |
|---|---|
| MATLAB | Signal processing & implementation |
| `audioread` | Load `.wav` audio files |
| `filter` | Apply room impulse response convolution |
| `find_peaks` *(scipy equivalent)* | R-peak detection (used in the companion ECG project) |
| LMS Algorithm | Adaptive echo estimation and cancellation |

---

*Implemented in MATLAB — Adaptive Signal Processing / Acoustic Echo Cancellation*
