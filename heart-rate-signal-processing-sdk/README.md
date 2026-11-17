# Heart Rate Signal Processing SDK

An open-source signal-processing SDK for reliable heart-rate estimation from wearable photoplethysmography (PPG) signals.

The SDK is intended to provide a reusable implementation of a signal-processing pipeline selected through systematic experiments on publicly available wearable PPG datasets.

The project focuses particularly on improving the quality of PPG signals under real-world conditions such as walking, running, exercise, wrist movement, and other activities that introduce motion artifacts.

> **Project status:** Research and development  
> **Intended use:** Research, experimentation, and software development  
> **Medical use:** This software is not a medical device and is not intended for diagnosis, treatment, or clinical decision-making.

---

# Overview

Wearable devices commonly use photoplethysmography (PPG) to estimate heart rate.

A PPG sensor illuminates the skin using an LED and measures changes in reflected light caused by changes in blood volume associated with each cardiac cycle.

Under relatively stable conditions, heart rate can often be estimated successfully from the periodicity of the PPG waveform.

However, wearable PPG signals can become substantially degraded when the user is moving.

Common causes include:

- walking;
- running;
- exercise;
- arm movement;
- changes in sensor-to-skin contact;
- wrist flexion;
- ambient-light interference;
- changes in peripheral perfusion;
- periodic motion that overlaps with the frequency of the cardiac signal.

These factors can cause a heart-rate algorithm to identify motion-related components instead of the true physiological pulse.

The purpose of this SDK is to provide a reusable signal-processing layer for wearable applications that need to prepare raw PPG and motion-sensor data for downstream heart-rate estimation.

---

# Research-Driven Development

The signal-processing pipeline used by this SDK is not selected in advance.

It is derived from the companion open-source research project:

**`wearable-biosignal-benchmark`**

The benchmark project systematically evaluates different combinations of:

- PPG preprocessing;
- filtering;
- motion-artifact mitigation;
- accelerometer-assisted processing;
- signal-quality assessment;
- tensor generation;
- heart-rate estimation models;
- activity conditions;
- subject-independent validation;
- cross-dataset validation.

Candidate processing methods include:

- Butterworth band-pass filtering;
- Savitzky-Golay smoothing;
- wavelet denoising;
- LMS adaptive filtering;
- NLMS adaptive filtering;
- accelerometer-assisted motion cancellation;
- motion-aware spectral processing;
- signal-quality gating.

The best-performing pipeline will be selected based on measured performance rather than assumption.

Selection criteria include:

- heart-rate estimation error;
- robustness during physical activity;
- robustness across subjects;
- performance on independent datasets;
- preprocessing latency;
- computational requirements;
- suitability for mobile and wearable deployment.

---

# Processing Architecture

The intended SDK architecture is:

```text
Raw PPG
   │
   ├──────────────┐
   │              │
   │        Accelerometer
   │        X / Y / Z
   │              │
   └──────┬───────┘
          │
          ▼
┌────────────────────────────┐
│ Input Validation           │
│                            │
│ Sampling-rate validation   │
│ Missing-value detection    │
│ Timestamp validation       │
└─────────────┬──────────────┘
              │
              ▼
┌────────────────────────────┐
│ PPG Preprocessing          │
│                            │
│ Detrending                 │
│ Filtering                  │
│ Normalization              │
└─────────────┬──────────────┘
              │
              ▼
┌────────────────────────────┐
│ Motion Artifact Processing │
│                            │
│ Accelerometer analysis     │
│ Motion estimation          │
│ Artifact reduction         │
└─────────────┬──────────────┘
              │
              ▼
┌────────────────────────────┐
│ Signal Quality Assessment  │
│                            │
│ Signal quality score       │
│ Motion level               │
│ Reliability assessment     │
└─────────────┬──────────────┘
              │
              ▼
┌────────────────────────────┐
│ Tensor Generation          │
│                            │
│ Windowing                  │
│ Channel construction       │
│ Model-ready representation │
└─────────────┬──────────────┘
              │
              ▼
       Model Input Tensor
```

The SDK is primarily responsible for producing a reliable, standardized signal representation for downstream heart-rate estimation.

---

# Design Goals

## Reliable PPG preprocessing

Provide a consistent pipeline for preparing wearable PPG signals before model inference.

## Motion-aware processing

Use accelerometer information where appropriate to identify or reduce motion-related contamination.

## Signal-quality awareness

Avoid treating every PPG window as equally reliable.

The SDK should be capable of reporting information such as:

```text
GOOD
DEGRADED
UNRELIABLE
```

or a numerical confidence/quality score.

## Reproducibility

The production implementation should correspond to processing methods evaluated in the `wearable-biosignal-benchmark` research repository.

## Mobile suitability

The selected processing pipeline should be computationally practical for Android and wearable environments.

## Model independence

The signal-processing layer should not be unnecessarily coupled to one specific machine-learning architecture.

Where practical, downstream applications should be able to use the processed representation with different heart-rate models.

---

# Intended Inputs

The SDK is expected to support synchronized time-series data including:

### PPG

```text
PPG samples
sampling frequency
timestamps
```

### Accelerometer

```text
ACC-X
ACC-Y
ACC-Z
sampling frequency
timestamps
```

Depending on the final experimentally selected architecture, accelerometer input may be optional or required.

---

# Intended Output

A processing operation may return a structure conceptually similar to:

```kotlin
HeartRateSignalResult(
    processedPpg = ...,
    tensor = ...,
    signalQuality = ...,
    motionLevel = ...,
    isReliable = ...,
    metadata = ...
)
```

The exact API will evolve as the research implementation matures.

---

# Example Future Usage

The intended API may eventually resemble:

```kotlin
val processor = HeartRateSignalProcessor(
    configuration = HeartRateProcessingConfig.default()
)

val result = processor.process(
    ppg = ppgSamples,
    accelerometerX = accX,
    accelerometerY = accY,
    accelerometerZ = accZ,
    samplingRate = 64
)

if (result.isReliable) {
    val tensor = result.tensor

    // Pass tensor to a heart-rate estimation model.
}
```

This example represents the intended design direction and is not a commitment to the final API.

---

# Signal Quality

One of the core goals of the SDK is to distinguish usable physiological signal from heavily contaminated signal.

Potential quality indicators evaluated by the research project include:

- PPG amplitude stability;
- spectral concentration;
- pulse periodicity;
- peak consistency;
- signal-to-noise indicators;
- accelerometer magnitude;
- motion intensity;
- PPG/accelerometer spectral overlap;
- signal discontinuities;
- missing samples.

Rather than forcing downstream inference for every signal window, the SDK may allow applications to reject or flag unreliable windows.

For example:

```text
PPG Window
     │
     ▼
Signal Quality
     │
 ┌───┴────┐
 │        │
 ▼        ▼
Reliable   Poor Quality
 │        │
 ▼        ▼
Inference  Reject / Defer
```

---

# Activity Robustness

The underlying research evaluates the processing pipeline under different physical activities.

Examples include:

- sitting;
- standing;
- walking;
- stairs;
- cycling;
- jogging;
- running;
- vigorous arm movement;
- mixed daily activity.

The objective is not merely to optimize performance under resting conditions.

The selected pipeline should provide a practical balance between accuracy and reliability across different motion conditions.

---

# Benchmark Relationship

The companion `wearable-biosignal-benchmark` project evaluates preprocessing methods and models using publicly available datasets containing combinations of:

- wrist PPG;
- accelerometer;
- activity labels;
- ECG-derived reference heart rate.

The research methodology includes:

```text
Dataset
   ↓
Preprocessing candidate
   ↓
Tensor representation
   ↓
Heart-rate model
   ↓
Prediction
   ↓
ECG ground truth
   ↓
Error analysis
```

The benchmark reports performance by activity rather than relying only on a single aggregate metric.

Metrics may include:

- Mean Absolute Error (MAE);
- Root Mean Squared Error (RMSE);
- percentage of predictions within ±5 BPM;
- percentage within ±10 BPM;
- correlation;
- signal rejection rate;
- preprocessing latency;
- inference latency;
- model size;
- memory requirements.

---

# Algorithm Selection

The final SDK pipeline will only be selected after benchmark experiments are completed.

The decision will consider four major dimensions:

```text
                Accuracy
                   │
                   │
Robustness ───── Selection ───── Efficiency
                   │
                   │
              Generalization
```

A method will not automatically be selected simply because it produces the lowest error on one dataset.

The preferred solution should also demonstrate:

- reasonable performance for unseen subjects;
- robustness during movement;
- acceptable performance on an independent dataset;
- practical computational requirements.

---

# Proposed Module Structure

The Heart Rate Signal Processing SDK is maintained as a dedicated module inside the parent **`wearable-biosignal-sdk`** repository.

Repository:

`https://github.com/jayaram1125/wearable-biosignal-sdk`

Heart-rate SDK module:

`https://github.com/jayaram1125/wearable-biosignal-sdk/tree/main/heart-rate-signal-processing-sdk`

The intended repository structure is:

```text
wearable-biosignal-sdk/
│
├── README.md
├── LICENSE
│
├── heart-rate-signal-processing-sdk/
│   │
│   ├── README.md
│   │
│   ├── build.gradle.kts
│   │
│   └── src/
│   │       ├── main/
│   │       │   └── kotlin/
│   │       │       └── .../
│   │       │
│   │       └── test/
│   │
│   ├── docs/
│   │   ├── architecture.md
│   │   ├── preprocessing.md
│   │   ├── signal-quality.md
│   │   ├── validation.md
│   │   └── benchmark-results.md
│   │
│   └── samples/
│
├── blood-pressure-signal-processing-sdk/
│
├── shared/
│
└── docs/
```

Within `heart-rate-signal-processing-sdk`, the Kotlin implementation is expected to be organized conceptually around the following components:

```text
heart-rate-signal-processing-sdk/
│
├── input/
│   ├── PpgSample
│   ├── AccelerometerSample
│   └── SignalWindow
│
├── preprocessing/
│   ├── detrending/
│   ├── normalization/
│   └── resampling/
│
├── filtering/
│   ├── bandpass/
│   ├── adaptive/
│   └── motion/
│
├── quality/
│   ├── SignalQualityEstimator
│   ├── MotionQualityEstimator
│   └── ReliabilityAssessment
│
├── tensor/
│   ├── WindowGenerator
│   ├── TensorGenerator
│   └── ModelInput
│
├── config/
│   └── HeartRateProcessingConfig
│
└── api/
    ├── HeartRateSignalProcessor
    └── HeartRateSignalResult
```

This structure is intentionally modular.

The major processing stages remain independently testable so that the Kotlin implementation can be validated against the corresponding Python reference implementation generated by the `wearable-biosignal-benchmark` project.

The `shared` directory at the parent repository level may later contain functionality that is genuinely reusable across multiple biosignal SDKs, such as:

- common time-series data structures;
- mathematical utilities;
- generic resampling utilities;
- common validation interfaces;
- benchmarking utilities.

Heart-rate-specific algorithms should remain inside `heart-rate-signal-processing-sdk`.

Similarly, future blood-pressure-specific processing should remain inside its own module instead of being coupled to the heart-rate implementation.

This allows the parent repository to support multiple independently testable biosignal-processing SDKs while preserving shared infrastructure where technically appropriate.

---

# Development Roadmap

## Phase 1 — Research

Complete comparative experiments in `wearable-biosignal-benchmark`.

Evaluate:

- preprocessing algorithms;
- motion-artifact techniques;
- tensor representations;
- ML models;
- subject-independent performance;
- activity-specific performance;
- cross-dataset generalization.

## Phase 2 — Reference Pipeline

Select the strongest practical signal-processing pipeline.

Create a canonical Python reference implementation.

Freeze expected processing outputs for validation.

## Phase 3 — Kotlin Implementation

Implement the selected pipeline in:

**`wearable-biosignal-sdk/heart-rate-signal-processing-sdk`**

The Kotlin implementation should be tested against the Python reference implementation.

For identical input windows:

```text
Python output
      │
      ├── compare numerically
      │
Kotlin output
```

Differences should remain within defined numerical tolerances.

## Phase 4 — Android SDK

Expose a stable public API.

Add:

- unit tests;
- integration tests;
- examples;
- documentation;
- performance benchmarks.

## Phase 5 — Open-Source Release

Maintain the SDK publicly within the `wearable-biosignal-sdk` GitHub repository under an open-source license.

## Phase 6 — Package Distribution

Publish the `heart-rate-signal-processing-sdk` as a versioned Android/Kotlin artifact through Maven Central so developers can integrate it using standard Gradle dependency management.

Conceptually:

```kotlin
dependencies {
    implementation("...:heart-rate-signal-processing-sdk:<version>")
}
```

The final Maven group ID, artifact ID, and versioning scheme will be defined before the first stable release.

---

# Validation Strategy

The Android implementation should not be considered complete merely because it compiles.

Validation should include:

### Numerical equivalence

Compare Kotlin processing output against the Python reference implementation.

### Dataset replay

Replay selected windows from public research datasets through the Kotlin SDK.

### Activity-specific testing

Verify representative windows covering:

- rest;
- walking;
- running;
- high-motion conditions.

### Performance testing

Measure:

- execution latency;
- CPU utilization;
- memory usage;
- allocation behavior;
- potential impact on battery usage.

### Regression testing

Known input signals should produce stable expected outputs across SDK releases.

---

# Open-Source Goals

The project is intended to allow third-party researchers and developers to:

- inspect the signal-processing implementation;
- reproduce processing behavior;
- integrate the processing pipeline into Android applications;
- test alternative models;
- compare results;
- report issues;
- propose improvements;
- contribute algorithms;
- extend support to additional sensors or datasets.

The goal is not only to publish source code, but to provide a reusable and testable implementation.

---

# Contributing

Contributions will be welcome after the initial architecture stabilizes.

Potential contributions may include:

- additional signal-quality metrics;
- performance optimizations;
- new preprocessing implementations;
- benchmark datasets;
- numerical-validation tests;
- Android-device performance measurements;
- documentation improvements;
- bug fixes.

A dedicated `CONTRIBUTING.md` will define development and testing requirements.

---

# License

This project is intended to be released under the:

**Apache License 2.0**

The license permits use, modification, and distribution subject to its terms and includes an explicit patent-license provision.

Third-party datasets, models, libraries, or research implementations used during development remain subject to their respective licenses.

---

# Research Transparency

This project will distinguish clearly between:

### Published research findings

Results reproduced from prior peer-reviewed studies.

### Reproduced results

Results obtained by running published implementations.

### Original benchmark results

Results generated through the `wearable-biosignal-benchmark` experiments.

### Production SDK decisions

Processing algorithms selected for implementation based on benchmark findings.

No accuracy value will be claimed unless supported by an actual experiment using documented data, methodology, and evaluation metrics.

---

# Limitations

PPG heart-rate estimation depends on:

- optical sensor hardware;
- sensor placement;
- sampling frequency;
- LED wavelength;
- photodetector characteristics;
- device fit;
- subject physiology;
- motion;
- environmental conditions.

Therefore, performance demonstrated using one public dataset or wearable sensor does not guarantee identical performance on another device.

The SDK should be considered a reusable research and engineering implementation rather than a clinically validated heart-rate monitor.

---

# Related Projects

## wearable-biosignal-benchmark

Python research framework used to evaluate PPG signal processing and heart-rate estimation models.

Research flow:

```text
Public datasets
      ↓
Python experiments
      ↓
Signal-processing comparison
      ↓
Model comparison
      ↓
Activity-specific evaluation
      ↓
Subject-independent evaluation
      ↓
Cross-dataset validation
      ↓
Selected reference pipeline
```

## wearable-biosignal-sdk

Parent Android/Kotlin repository containing reusable physiological-signal processing SDK modules.

Current/planned structure:

```text
wearable-biosignal-sdk/
│
├── heart-rate-signal-processing-sdk/
├── blood-pressure-signal-processing-sdk/
└── shared/
```

The Heart Rate Signal Processing SDK is therefore developed and released as a component of the broader `wearable-biosignal-sdk` project.

---

# Project Objective

The long-term objective of the Heart Rate Signal Processing SDK is to provide an open, reproducible, and computationally practical implementation of a signal-processing pipeline for preparing motion-corrupted wearable PPG signals for reliable heart-rate estimation.

The project follows the principle:

> **Experiment first, measure objectively, select the most reliable practical method, and then translate the validated methodology into reusable open-source software.**
