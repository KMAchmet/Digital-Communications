# Digital Communications Lab: Quantization, DPCM and PAM Transmission

A collection of MATLAB Live Scripts (`.mlx`) covering three topics from a digital communications course:

| Part | File | Topic |
|------|------|-------|
| A1 | `A1_code.mlx` | Uniform and Lloyd-Max quantization of a speech signal |
| A2 | `A2_code.mlx` | Differential Pulse Code Modulation (DPCM) with a linear predictor |
| B | `merosB.mlx` | Baseband-to-passband PAM transmission over an AWGN channel (BER and SER) |

The scripts contain the code, inline comments, and the generated plots and printed results. The section headings inside the live scripts are written in Greek; this README describes them in English.

---

## Table of Contents

- [Requirements](#requirements)
- [Repository Structure](#repository-structure)
- [Part A1: Uniform and Lloyd-Max Quantization](#part-a1-uniform-and-lloyd-max-quantization)
- [Part A2: DPCM](#part-a2-dpcm)
- [Part B: PAM Transmission over an AWGN Channel](#part-b-pam-transmission-over-an-awgn-channel)
- [How to Run](#how-to-run)

---

## Requirements

- MATLAB with Live Script support (a recent release is recommended, because the scripts define local functions inside the live script)
- **Communications Toolbox** for Part B (`randsrc`, `pammod`, `pamdemod`, `awgn`, `bi2de`, `de2bi`)
- Input data that is loaded by the scripts but not stored in the `.mlx` files:
  - `speech.wav`, a speech recording used in Part A1
  - `source.mat`, containing the signal `t` used in Part A2
  - A `functions` folder with the helper functions used in Part A2: `Rx`, `iquantizer`, `idpcm_enco` and `idpcm_deco`

## Repository Structure

```
.
├── README.md
├── A1_code.mlx
├── A2_code.mlx
├── merosB.mlx
├── speech.wav          (input for A1)
├── source.mat          (input for A2)
├── functions/          (helper functions for A2)
└── sources/            (data folder referenced by A2)
```

Adjust the tree to match your actual repository.

---

## Part A1: Uniform and Lloyd-Max Quantization

**File:** `A1_code.mlx`

The speech signal is loaded with `audioread`, normalised to the range [-1, 1], and quantized with N bits using two different quantizers.

### Functions

- **`my_quantizer(x, N, min_value, max_value)`**: a uniform quantizer. The range is split into `2^N` equal intervals of width `delta`, the reconstruction levels are the interval midpoints, and each sample is mapped to the midpoint of its interval.
- **`Lloyd_Max(x, N, min_value, max_value, Kmax, epsilon)`**: an iterative Lloyd-Max quantizer. It starts from the uniform centers and then repeats:
  1. Assign every sample to its nearest center (this is equivalent to using the midpoints between centers as decision thresholds).
  2. Compute the distortion `D`, the mean squared error between the signal and its quantized version.
  3. Stop if the change in distortion is smaller than `epsilon` (1e-6), or if `Kmax` iterations are reached.
  4. Otherwise move each center to the mean of the samples assigned to it.

### Exercise 1

Quantizes the signal with N = 4 bits using both quantizers (`epsilon = 1e-6`, `Kmax = 100`) and plots each quantized waveform over the original signal.

### Exercise 2

| Sub-question | What it does |
|--------------|--------------|
| 1 | Runs Lloyd-Max for N = 2, 4 and 8 bits with the number of iterations ranging from 1 to 50, and plots the SQNR against the number of iterations. This shows how quickly the algorithm converges. |
| 2 | Computes and compares the SQNR of the uniform and Lloyd-Max quantizers for N = 2, 4, 8 bits and shows the result as a bar chart. |
| 3 | Plays the original, uniform-quantized and Lloyd-Max-quantized signals for each N, and plots the three waveforms together for comparison. |
| 4 | Computes and compares the mean squared error (MSE) of both quantizers for N = 2, 4, 8 bits. |

SQNR is computed as `10 * log10(mean(y.^2) / mean((y - xq).^2))`.

### Results

| N (bits) | SQNR Uniform (dB) | SQNR Lloyd-Max (dB) | MSE Uniform | MSE Lloyd-Max |
|----------|-------------------|---------------------|-------------|---------------|
| 2 | -2.48 | 7.19 | 0.035736 | 0.003850 |
| 4 | 11.17 | 17.59 | 0.001542 | 0.000351 |
| 8 | 35.97 | 36.03 | 0.000005 | 0.000005 |

Lloyd-Max gives a clear advantage at low bit depths because its reconstruction levels adapt to the amplitude distribution of the speech signal, which is concentrated near zero. At 8 bits the quantization steps are so fine that both quantizers perform almost identically.

The file also contains clipping variants of the two functions (`my_quantizer2` and `Lloyd_Max2`) that are defined but not used in the experiments.

---

## Part A2: DPCM

**File:** `A2_code.mlx`

Implementation and evaluation of a DPCM coder and decoder for the signal `t` stored in `source.mat`. The script notes that the source signal does not sound pleasant or intelligible when played, so it is analysed through plots.

### Processing steps

1. **Load and plot the source** signal `t`.
2. **Compute the stochastic quantities** for a predictor of order `p = 5`: the autocorrelation matrix `R` and the cross-correlation vector `r`, using `Rx(p, t)`.
3. **Compute the predictor coefficients** by solving the normal equations, `a = R \ r`.
4. **Quantize the coefficients** with an 8-bit quantizer over the range [-2, 2] using `iquantizer`, so that the coefficients can be transmitted with limited precision.

Example values from the script for `p = 5`:

| | a1 | a2 | a3 | a4 | a5 |
|---|----|----|----|----|----|
| Exact `a` | 1.2852 | -1.5856 | 0.9901 | -0.5424 | -0.0287 |
| Quantized `a` | 1.2988 | -1.5723 | 0.9980 | -0.5332 | -0.0410 |

### Exercises

1. **Encoding and decoding.** With N = 4 bits and a quantizer range of [-3.5, 3.5] (16 reconstruction levels), `idpcm_enco(t, p, N, min_value, max_value)` returns the encoded symbols, the quantizer centers, the quantized predictor coefficients and the signal `y` produced by the encoder. `idpcm_deco(encoded, a, centers)` reconstructs the signal, which is plotted against the original. The encoder computes `R`, `r` and the quantizer centers internally so that the experiments in the next exercises can be repeated easily with different parameters.
2. **Effect of `p` and `N` on the prediction error.** For `p` in {5, 10} and `N` in {1, 2, 3}, the original signal and the encoder output `y` are plotted for each combination.
3. **Effect of `p` and `N` on the mean squared error.** For `p` from 5 to 10 and `N` from 1 to 3, the signal is encoded and decoded and the MSE between the original and the reconstruction is computed. The results are shown in a 3D bar chart (`bar3`) with N, p and MSE on the axes.
4. **Signal reconstruction.** For `p` in {5, 10} and `N` in {1, 2, 3}, the reconstructed signals are plotted, followed by a side-by-side comparison of the original and a reconstructed signal.

At the end of the script, `rmpath` removes the `./functions` and `./sources` folders from the MATLAB path.

---

## Part B: PAM Transmission over an AWGN Channel

**File:** `merosB.mlx`

A complete simulation chain for M-PAM transmission, including a carrier-modulated waveform, an AWGN channel and a correlation receiver.

### System parameters

| Parameter | Value |
|-----------|-------|
| Symbol period `Tsymbol` | 4 microseconds |
| Carrier frequency `fc` | 2.5 MHz |
| Sampling period `Tsample` | 0.1 microseconds (40 samples per symbol) |
| Pulse | Rectangular, amplitude `sqrt(2 / Tsymbol)` |
| Number of bits `Lb` | 20232 |
| SNR range | 0 to 20 dB in steps of 2 dB |

### Functions

| Function | Role |
|----------|------|
| `source(Lb)` | Generates `Lb` random bits |
| `mapper(symbols_order, signal, M)` | Groups the bits into words of `log2(M)` bits and maps them to PAM symbols with `pammod`, using binary or Gray mapping |
| `modulator(symbols)` | Multiplies each symbol by the pulse and the carrier `cos(2*pi*fc*t)` to build the transmitted waveform |
| `AWGN(sm, SNR)` | Adds white Gaussian noise at the given SNR using `awgn` with the `'measured'` option |
| `demodulator(received)` | Correlates each symbol interval with the carrier and pulse and integrates, producing one decision variable per symbol |
| `demapper(symbols_order, symbols, M)` | Detects the symbols with `pamdemod` and converts them back to bits |

### Exercises

- **Exercise 2, bit error rate (BER).** Estimates the BER for 2-PAM with binary mapping, 8-PAM with binary mapping, and 8-PAM with Gray mapping across the SNR range, and plots the curves on a semilogarithmic scale. Gray mapping is expected to give a lower BER than binary mapping for 8-PAM, because neighbouring symbols differ in only one bit.
- **Exercise 3, symbol error rate (SER).** Estimates the SER for 2-PAM and 8-PAM with binary mapping. The detected bits are mapped back to symbols and compared with the transmitted symbols.

---

## How to Run

1. Clone the repository and open MATLAB in the repository folder.
2. Place the required input files (`speech.wav`, `source.mat`) and the `functions` folder in the working directory, or add them to the MATLAB path.
3. Open the desired `.mlx` file in the MATLAB Editor and click **Run**, or run it section by section.
4. Part A1, sub-question 3 plays audio, so make sure your speakers or headphones are connected. The script pauses while each signal plays.
5. Part B requires the Communications Toolbox.

Each live script already includes saved outputs, so you can also read the results directly in MATLAB without re-running the code.
