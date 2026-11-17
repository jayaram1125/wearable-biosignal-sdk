
# Blood Pressure Signal Processing SDK

An open-source Kotlin/Android signal-processing SDK for preparing wearable photoplethysmography (PPG) and motion-sensor data for research-oriented cuffless blood-pressure estimation.

The SDK implements the signal-processing pipeline selected through controlled experiments in the companion:

**`blood-pressure-prediction-benchmark`**

project.

Its primary responsibility is to transform raw wearable biosignals into standardized, quality-assessed, model-compatible tensors that can be passed to a separately deployed blood-pressure estimation model.

Repository location:

`wearable-biosignal-sdk/blood-pressure-signal-processing-sdk`

> **Project status:** Research and development  
> **Language:** Kotlin  
> **Target platform:** Android / Wear OS-compatible environments  
> **Purpose:** Research, experimentation, benchmarking, and software development  
> **Medical use:** This SDK is not a medical device and is not intended for diagnosis, treatment, medication decisions, or clinical blood-pressure monitoring.

---

# Overview

Cuffless blood-pressure estimation from wearable sensors is substantially more difficult than heart-rate estimation.

A wearable PPG sensor measures optical changes associated with peripheral blood-volume pulsations.

It does **not** directly measure arterial pressure.

Blood-pressure-related information may be reflected in characteristics of the PPG waveform, but those same characteristics can also be influenced by:

- arterial stiffness;
- vascular tone;
- temperature;
- autonomic state;
- medication;
- sensor placement;
- contact pressure;
- body position;
- skin and tissue characteristics;
- motion;
- peripheral perfusion.

Therefore:

```text
PPG waveform
     ↓
contains physiological information
     ↓
but is not a direct BP measurement
```

The purpose of this SDK is to provide a reusable and experimentally validated signal-processing layer for preparing PPG and related sensor data for downstream cuffless blood-pressure research.

---

# Research-Driven Development

The SDK does not assume in advance that a particular filter or signal representation is optimal.

The processing pipeline is derived from experiments performed in:

**`blood-pressure-prediction-benchmark`**

The benchmark evaluates combinations of:

- PPG preprocessing;
- filtering;
- waveform denoising;
- morphology preservation;
- first- and second-derivative representations;
- time-frequency representations;
- beat alignment;
- motion-artifact analysis;
- accelerometer information;
- signal-quality assessment;
- normalization;
- calibration strategies;
- machine-learning models;
- subject-independent validation;
- cross-dataset validation.

Candidate signal-processing approaches include:

- detrending;
- Butterworth band-pass filtering;
- Savitzky-Golay smoothing;
- wavelet denoising;
- PPG first derivative / VPG;
- PPG second derivative / APG;
- beat-aligned processing;
- motion-aware processing;
- signal-quality gating;
- CWT-based representations;
- STFT-based representations.

The final SDK pipeline will be selected based on measured results rather than assumption.

---

# Project Relationship

The overall project architecture is:

```text
Public BP datasets
        ↓
blood-pressure-prediction-benchmark
        ↓
Controlled signal-processing experiments
        ↓
Signal-representation experiments
        ↓
Model training / fine-tuning
        ↓
Subject-independent validation
        ↓
Cross-dataset validation
        ↓
Selected processing pipeline
        ↓
blood-pressure-signal-processing-sdk
        ↓
Model-ready tensors
        ↓
Selected BP estimation model
        ↓
Research BP estimate
```

The benchmark determines **what should be implemented**.

The SDK provides a reusable Kotlin implementation of the selected processing methodology.

---

# SDK Responsibility

The primary responsibility of `blood-pressure-signal-processing-sdk` is:

> **Convert raw wearable biosignals into validated, quality-assessed, model-compatible input representations for cuffless blood-pressure estimation research.**

The SDK is not responsible for establishing clinical BP accuracy.

The SDK is also designed to remain as independent as practical from any single machine-learning architecture.

---

# Intended Processing Architecture

```text
Raw PPG
   │
   ├────────────── Optional ACC / IMU
   │
   ▼
┌─────────────────────────────┐
│ Input Validation            │
│                             │
│ sampling rate               │
│ timestamps                  │
│ missing samples             │
│ clipping                    │
│ flatline detection          │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│ Synchronization / Resampling│
│                             │
│ align sensor streams        │
│ normalize sample timing     │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│ PPG Preprocessing           │
│                             │
│ detrending                  │
│ selected filtering          │
│ selected denoising          │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│ Motion / Quality Analysis   │
│                             │
│ accelerometer analysis      │
│ motion level                │
│ signal quality              │
│ reliability assessment      │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│ Morphology / Representation │
│                             │
│ PPG                         │
│ VPG                         │
│ APG                         │
│ CWT / STFT if selected      │
│ beat alignment if selected  │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│ Tensor Generation           │
│                             │
│ windowing                   │
│ channel construction        │
│ normalization               │
│ model contract              │
└──────────────┬──────────────┘
               │
               ▼
       Model Input Tensor
```

The exact implementation will reflect the experimentally selected processing pipeline.

---

# Design Goals

## 1. Morphology Preservation

Blood-pressure estimation may depend on subtle characteristics of the PPG waveform.

The SDK should therefore avoid signal processing that unnecessarily destroys features such as:

- pulse upstroke;
- pulse width;
- slope;
- relative peak morphology;
- reflected-wave characteristics;
- first-derivative features;
- second-derivative features.

Filtering should reduce noise while preserving BP-relevant waveform information.

---

## 2. Signal-Quality Awareness

The SDK should not assume that every PPG segment is suitable for BP estimation.

A core design principle is:

```text
PPG window
     ↓
Signal-quality assessment
     ↓
┌─────────────────┬──────────────────┐
│ Reliable        │ Unreliable       │
│                 │                  │
▼                 ▼
Generate tensor   Suppress / reject
```

Possible output states may include:

```text
GOOD
DEGRADED
UNRELIABLE
```

or a numerical quality score.

---

## 3. Motion Awareness

Where accelerometer or IMU data are available, the SDK may use them to characterize:

- movement magnitude;
- periodic motion;
- posture changes;
- high-motion windows;
- conditions where PPG morphology may be unreliable.

Motion information may be used for:

- filtering;
- feature generation;
- signal-quality assessment;
- reliability gating;
- model input.

The SDK should not assume that motion-corrupted PPG can always be recovered.

A valid result may be:

```text
Blood-pressure inference not recommended
because signal quality is insufficient.
```

---

## 4. Reproducibility

The Kotlin implementation should reproduce the reference preprocessing used in `blood-pressure-prediction-benchmark`.

For identical input:

```text
Python reference pipeline
          │
          ├── numerical comparison
          │
Kotlin SDK pipeline
```

outputs should remain within documented numerical tolerances.

---

## 5. Model Compatibility

The SDK should provide a stable model-input contract.

The processing pipeline used during deployment must match the preprocessing used during model training.

A model trained using:

```text
PPG
+
VPG
+
APG
```

must not later receive a runtime tensor containing only differently filtered PPG.

Training/deployment consistency is mandatory.

---

## 6. Edge Suitability

The selected implementation should eventually be practical for Android and wearable environments.

Evaluation should therefore consider:

- preprocessing latency;
- memory usage;
- temporary allocations;
- computational complexity;
- battery impact;
- tensor size.

---

# Intended Inputs

The SDK is expected to accept time-series inputs such as:

## PPG

```text
samples
timestamps
sampling frequency
channel / wavelength metadata where applicable
```

## Accelerometer

Where available:

```text
ACC-X
ACC-Y
ACC-Z
timestamps
sampling frequency
```

## Optional Metadata

Depending on the selected research methodology, configuration may eventually include:

```text
sensor location
device sampling rate
calibration mode
reference calibration BP
```

The SDK should not silently require demographic information unless the experimentally selected model explicitly depends on it.

---

# Intended Output

A processing request may conceptually return:

```kotlin
BloodPressureSignalResult(
    processedPpg = ...,
    tensor = ...,
    signalQuality = ...,
    motionLevel = ...,
    isReliableForInference = ...,
    metadata = ...
)
```

The exact API will be finalized after the reference processing pipeline is selected.

---

# Example Future Usage

The public API may eventually resemble:

```kotlin
val processor = BloodPressureSignalProcessor(
    configuration = BloodPressureProcessingConfig.default()
)

val result = processor.process(
    ppg = ppgSamples,
    accelerometerX = accX,
    accelerometerY = accY,
    accelerometerZ = accZ,
    samplingRate = 100
)

if (result.isReliableForInference) {

    val modelInput = result.tensor

    // Pass modelInput to the selected
    // blood-pressure estimation model.

} else {

    // Do not perform BP inference
    // from this signal window.
}
```

This example describes the intended architecture only. The final API may change based on research findings.

---

# Signal Representation

The companion benchmark evaluates multiple representations before one is selected for SDK implementation.

Possible representations include:

## Time-Domain PPG

```text
[PPG]
```

## PPG + First Derivative

```text
[PPG]
[VPG]
```

## PPG + First and Second Derivatives

```text
[PPG]
[VPG]
[APG]
```

## PPG + Motion

```text
[PPG]
[ACC-X]
[ACC-Y]
[ACC-Z]
```

## Morphology + Motion

```text
[PPG]
[VPG]
[APG]
[ACC magnitude]
```

## Time-Frequency Representation

Depending on benchmark results:

```text
CWT(PPG)
```

or:

```text
STFT(PPG)
```

The SDK will implement the representation selected through reproducible experiments.

---

# Calibration Modes

Cuffless BP research frequently distinguishes between calibration-free and calibrated estimation.

The SDK architecture should therefore avoid assuming that all models operate identically.

## Calibration-Free

```text
PPG
 ↓
processing
 ↓
tensor
 ↓
model
 ↓
SBP / DBP
```

No known BP value from the current user is supplied.

## Calibration-Based

```text
Reference cuff BP
        +
PPG
        ↓
personalized model context
        ↓
subsequent PPG
        ↓
BP estimate
```

If the selected benchmark model requires calibration, that requirement must be explicitly represented in the SDK configuration.

Example conceptually:

```kotlin
val calibration = BloodPressureCalibration(
    systolic = 122f,
    diastolic = 78f
)
```

Calibration-free and calibration-based results should never be presented as equivalent.

---

# Signal Quality and Reliability

Signal quality is a core component of the SDK.

Possible quality indicators investigated by the benchmark include:

- clipping;
- flatline detection;
- missing samples;
- pulse consistency;
- spectral quality;
- morphology consistency;
- beat-to-beat similarity;
- amplitude stability;
- accelerometer magnitude;
- motion intensity;
- sensor-signal discontinuities.

A possible reliability object may eventually resemble:

```kotlin
SignalReliability(
    qualityScore = 0.87f,
    motionScore = 0.20f,
    validPulseCount = 8,
    reliableForInference = true
)
```

The threshold values must come from validation experiments rather than arbitrary constants.

---

# Why Reliability Gating Matters

A BP model may produce a numerically plausible value even when its input PPG is corrupted.

For example:

```text
poor-quality PPG
      ↓
model
      ↓
124 / 79
```

The number may look reasonable despite being unsupported by the signal.

Therefore, the intended architecture is:

```text
PPG
 ↓
signal-quality assessment
 ↓
┌─────────────────────────────┐
│ Is signal suitable for BP?  │
└───────────┬─────────────────┘
            │
      ┌─────┴─────┐
      │           │
     YES          NO
      │           │
      ▼           ▼
generate       reject /
tensor         suppress
```

Reliability gating is therefore part of the processing pipeline rather than merely a UI feature.

---

# Activity and Motion Conditions

The companion benchmark investigates how PPG reliability changes under different physical conditions.

Potential categories include:

- seated rest;
- standing;
- walking;
- light activity;
- running;
- arm movement;
- high-motion conditions.

However, the project distinguishes between:

> **testing signal quality under motion**

and:

> **claiming accurate continuous blood-pressure ground truth during motion.**

Many available public activity datasets do not contain continuous high-quality BP reference measurements throughout exercise.

The SDK therefore focuses first on identifying whether the signal is sufficiently reliable for inference.

---

# Relationship to `blood-pressure-prediction-benchmark`

The companion benchmark is responsible for determining which processing pipeline should be implemented here.

Its research flow is:

```text
Public BP datasets
        ↓
signal-processing candidates
        ↓
signal representations
        ↓
model candidates
        ↓
subject-independent evaluation
        ↓
calibration evaluation
        ↓
external dataset validation
        ↓
motion / signal-quality analysis
        ↓
selected reference pipeline
```

The SDK should implement only the final validated reference pipeline, while experimental alternatives remain in the Python benchmark project.

This keeps the production-facing Kotlin SDK focused and maintainable.

---

# Algorithm Selection Criteria

The processing pipeline will not be selected solely by the lowest error observed on one dataset.

Selection will consider:

```text
subject-independent accuracy
        +
external generalization
        +
BP-range robustness
        +
morphology preservation
        +
signal-quality behavior
        +
motion robustness
        +
computational efficiency
```

A method that achieves slightly lower error on one clinical dataset but fails on wrist PPG may not be selected for the wearable SDK.

---

# Proposed Module Structure

`blood-pressure-signal-processing-sdk` is maintained as a dedicated module within:

**`wearable-biosignal-sdk`**

Repository structure:

```text
wearable-biosignal-sdk/
│
├── README.md
├── LICENSE
│
├── heart-rate-signal-processing-sdk/
│
├── blood-pressure-signal-processing-sdk/
│   │
│   ├── README.md
│   ├── build.gradle.kts
│   │
│   ├── src/
│   │   ├── main/
│   │   │   └── kotlin/
│   │   │
│   │   └── test/
│   │
│   ├── docs/
│   │   ├── architecture.md
│   │   ├── preprocessing.md
│   │   ├── morphology.md
│   │   ├── signal-quality.md
│   │   ├── calibration.md
│   │   ├── MODEL_CONTRACT.md
│   │   ├── validation.md
│   │   └── benchmark-results.md
│   │
│   └── samples/
│
└── shared/
```

---

# Internal SDK Architecture

The Kotlin implementation may be organized conceptually as:

```text
blood-pressure-signal-processing-sdk/
│
├── input/
│   ├── PpgSample
│   ├── AccelerometerSample
│   └── SignalWindow
│
├── validation/
│   ├── SampleValidator
│   ├── TimestampValidator
│   └── MissingDataDetector
│
├── preprocessing/
│   ├── detrending/
│   ├── resampling/
│   ├── normalization/
│   └── beat_detection/
│
├── filtering/
│   ├── bandpass/
│   ├── smoothing/
│   ├── wavelet/
│   └── motion/
│
├── morphology/
│   ├── PpgDerivative
│   ├── VelocityPpg
│   ├── AccelerationPpg
│   └── BeatAlignment
│
├── quality/
│   ├── PpgQualityEstimator
│   ├── MotionQualityEstimator
│   ├── MorphologyQualityEstimator
│   └── ReliabilityGate
│
├── representation/
│   ├── TimeDomainRepresentation
│   ├── MorphologyRepresentation
│   ├── CwtRepresentation
│   └── ModelInputTensor
│
├── calibration/
│   ├── BloodPressureCalibration
│   └── CalibrationConfig
│
├── config/
│   └── BloodPressureProcessingConfig
│
└── api/
    ├── BloodPressureSignalProcessor
    └── BloodPressureSignalResult
```

The final package organization will reflect the processing pipeline selected by the research benchmark.

Unused experimental algorithms should not automatically be carried into the stable SDK.

---

# Shared Components

The parent `wearable-biosignal-sdk` repository may contain a `shared` module for functionality genuinely common to heart-rate and blood-pressure processing.

Potential shared functionality includes:

- timestamp utilities;
- generic time-series structures;
- resampling;
- mathematical utilities;
- accelerometer structures;
- general signal-window abstractions.

BP-specific functionality should remain in:

**`blood-pressure-signal-processing-sdk`**

because BP processing may require morphology preservation, calibration, and reliability logic that is inappropriate for heart-rate processing.

---

# Development Roadmap

## Phase 1 — Benchmark Research

Complete experiments in:

**`blood-pressure-prediction-benchmark`**

including:

- preprocessing comparison;
- signal-representation comparison;
- model comparison;
- calibration analysis;
- subject-independent testing;
- external dataset validation;
- motion/signal-quality analysis.

---

## Phase 2 — Select Reference Processing Pipeline

Identify the processing configuration with the strongest practical combination of:

- generalization;
- reliability;
- morphology preservation;
- computational efficiency.

Freeze the Python reference implementation.

---

## Phase 3 — Define Model Contract

Document exactly what the final BP model expects.

For example:

```text
Window:
10 seconds

Input channels:
PPG
VPG
APG

Sampling frequency:
125 Hz

Tensor type:
float32

Normalization:
training-set normalization

Tensor shape:
[1, 3, 1250]
```

This example is illustrative only.

The actual contract will be defined from experimental results.

---

## Phase 4 — Kotlin Implementation

Port the selected Python preprocessing pipeline to:

**`blood-pressure-signal-processing-sdk`**

For identical input windows:

```text
Python reference output
           │
           ├── compare
           │
Kotlin SDK output
```

Results should remain within documented numerical tolerances.

---

## Phase 5 — Dataset Replay Testing

Replay samples from public research datasets through the Kotlin implementation.

Verify:

- signal processing;
- morphology outputs;
- quality scores;
- tensor values;
- rejection decisions.

---

## Phase 6 — Android Performance Testing

Measure:

- preprocessing latency;
- memory;
- allocations;
- tensor-generation latency;
- CPU utilization.

---

## Phase 7 — Public Release

Publish:

- source code;
- architecture;
- API documentation;
- reference tests;
- validation methodology;
- known limitations.

---

## Phase 8 — Maven Central Distribution

Publish a versioned reusable Android/Kotlin package.

Conceptually:

```kotlin
dependencies {
    implementation(
        "io.github.jayaram1125:blood-pressure-signal-processing-sdk:<version>"
    )
}
```

The final group ID, artifact ID, and versioning scheme will be defined during publication.

---

# Python / Kotlin Equivalence Testing

This is a major requirement.

Suppose the Python reference produces:

```text
Processed PPG:
[...]

VPG:
[...]

APG:
[...]

Tensor:
[...]

Quality:
0.88
```

The Kotlin implementation should produce numerically equivalent results within established tolerance.

Tests should cover:

- clean resting signals;
- noisy signals;
- motion-corrupted signals;
- missing samples;
- boundary conditions;
- low-quality windows.

This helps ensure that Android deployment does not silently change the algorithm validated in Python.

---

# Model Contract

A dedicated:

**`MODEL_CONTRACT.md`**

should document the interface between this SDK and the selected BP model.

It should specify:

- input sampling frequency;
- window length;
- stride;
- required channels;
- channel order;
- preprocessing steps;
- normalization;
- tensor shape;
- tensor type;
- calibration requirements;
- expected model outputs;
- units;
- reliability requirements.

For example:

```text
SDK
 ↓
ModelInputTensor
 ↓
selected BP model
 ↓
SBP / DBP
```

The signal-processing SDK should be independently versioned from the model, while maintaining compatibility metadata.

---

# Version Compatibility

A future model metadata record could include:

```json
{
  "modelVersion": "1.0.0",
  "requiredSdkVersion": "1.x",
  "processingPipeline": "bp-pipeline-v1",
  "calibrationRequired": false
}
```

This prevents a newer processing pipeline from being accidentally paired with a model trained using an older transformation.

---

# Testing Strategy

## Unit Tests

Test individual components:

- filters;
- derivatives;
- resampling;
- windowing;
- signal-quality calculations;
- calibration structures.

## Numerical Reference Tests

Compare outputs against Python reference arrays.

## Regression Tests

Known input should produce stable known output across SDK versions.

## Dataset Replay Tests

Feed selected public-dataset windows through the full SDK.

## Motion Tests

Verify quality/reliability behavior for motion-corrupted examples.

## Performance Tests

Measure runtime characteristics on representative Android hardware.

---

# Open-Source Goals

The project is intended to allow researchers and developers to:

- inspect the processing methodology;
- reproduce the Android implementation;
- compare Kotlin output with Python research output;
- integrate the signal-processing layer into experimental applications;
- test alternative BP models;
- contribute improvements;
- evaluate additional sensors;
- examine reliability behavior.

The goal is not merely to publish source files.

The goal is to provide a **documented, testable, reusable implementation** of an experimentally selected cuffless-BP signal-processing methodology.

---

# Public Dissemination

The project forms part of a broader open-source research-to-deployment pathway:

```text
blood-pressure-prediction-benchmark
        │
        │ research
        │ controlled experiments
        │ model evaluation
        │ processing selection
        ▼
blood-pressure-signal-processing-sdk
        │
        │ reusable Kotlin
        │ tensor generation
        │ signal-quality gating
        ▼
future wearable reference application
        │
        │ on-device model inference
        ▼
Researchers and developers
```

This allows third parties to inspect and reproduce the path from scientific experimentation to reusable mobile implementation.

---

# Research Transparency

The project will clearly distinguish:

## Published Research Findings

Results reported in external scientific literature.

## Reproduced Results

Results obtained by independently running published implementations.

## Original Benchmark Results

Results generated by `blood-pressure-prediction-benchmark`.

## Selected SDK Pipeline

Processing methodology chosen from benchmark results.

## Android Runtime Results

Performance measured directly from the Kotlin implementation.

No numerical accuracy, latency, battery, or clinical claim should be made without documented experimental evidence.

---

# Current Limitations

## PPG Does Not Directly Measure BP

The SDK prepares PPG-derived information for research-oriented BP inference.

It does not convert PPG into a directly measured pressure signal.

## Sensor Differences

PPG characteristics vary between devices because of differences in:

- wavelength;
- optical geometry;
- photodetectors;
- analog front ends;
- sampling rate;
- body location;
- contact pressure.

A pipeline validated on one sensor may not transfer unchanged to another.

## Dataset Differences

Many public BP datasets use clinical fingertip PPG rather than wrist-wearable PPG.

Cross-dataset and wrist validation remain essential.

## Motion

The SDK may determine that a signal is unsuitable for inference.

It should not guarantee BP prediction under every motion condition.

## Calibration

Some models may require subject-specific calibration.

Calibration requirements must be clearly documented.

## Clinical Validation

Research benchmark accuracy does not establish medical-device accuracy.

This SDK is not intended to replace validated cuff-based BP monitors.

---

# Related Projects

## blood-pressure-prediction-benchmark

Python research framework responsible for:

```text
public BP datasets
        ↓
signal-processing experiments
        ↓
signal-representation experiments
        ↓
model training / fine-tuning
        ↓
calibration experiments
        ↓
subject-independent testing
        ↓
cross-dataset validation
        ↓
motion / quality analysis
        ↓
final pipeline selection
```

## blood-pressure-signal-processing-sdk

This module.

Responsible for:

```text
raw PPG / motion signals
        ↓
validated processing
        ↓
quality assessment
        ↓
selected representation
        ↓
model-ready tensor
```

## wearable-biosignal-sdk

Parent repository containing reusable Android/Kotlin biosignal-processing modules, including:

```text
heart-rate-signal-processing-sdk
blood-pressure-signal-processing-sdk
```

---

# Long-Term Objective

The long-term objective of `blood-pressure-signal-processing-sdk` is to provide an open, reproducible, and computationally practical implementation of the signal-processing methodology selected through systematic cuffless blood-pressure research.

The intended development philosophy is:

> **Experiment first, validate on unseen subjects and independent datasets, preserve physiologically meaningful signal information, reject unreliable measurements when necessary, and only then translate the selected methodology into reusable open-source software.**

---

# License

This project is intended to be released under the:

**Apache License 2.0**

Third-party datasets, research implementations, pretrained models, and software dependencies remain subject to their respective licenses.

---

# Disclaimer

`blood-pressure-signal-processing-sdk` is provided for research, education, software development, and experimental use.

It is **not a medical device**, has not been clinically validated for diagnosis or treatment, and should not be used as a substitute for validated blood-pressure measurement equipment or professional medical advice.
