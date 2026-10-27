# Identifiability-Aware Inverse Design of Chalcogenide Photonic Crystal Fibers

**Target-specific explainable learning for inverse design of photonic
crystal fibers (PCFs)**

This research investigates which structural parameters of a chalcogenide
photonic crystal fiber (PCF) can be recovered from its simulated optical
response. Rather than assuming that every geometric parameter is
identifiable, the study evaluates four discrete design parameters
separately and uses target-specific Gradient SHAP feature selection to
reduce the spectral input.

## Overview

In conventional PCF design, a geometry is specified first and its
optical response is computed using a numerical solver. Inverse design
reverses this process: given a desired optical response, the objective
is to identify a geometry that can produce it.

However, different geometries can produce similar responses. This
project studies that identifiability challenge using COMSOL-simulated
Ge-As-Se chalcogenide PCFs and an explainable, decoupled neural-network
framework.

### Research questions

-   Which structural parameters can be recovered reliably from the
    optical response?
-   Does selecting wavelengths separately for each target improve
    inverse-design performance?
-   How well do predicted geometries reproduce a target response when
    re-simulated in COMSOL?

## Main contributions

-   **Target-specific feature selection:** Gradient SHAP ranks
    wavelengths independently for each structural target.
-   **Decoupled learning:** Four independent single-head neural networks
    predict the four design parameters without shared weights,
    gradients, or optimizer state.
-   **Structure-wise evaluation:** Five-fold `GroupKFold`
    cross-validation keeps each structure intact, avoiding splits
    between wavelength rows from the same geometry.
-   **Explainability cross-check:** Gradient SHAP rankings are compared
    with LIME.
-   **Numerical validation:** The ten best predicted geometries are
    re-simulated in COMSOL.
-   **Identifiability analysis:** Results are interpreted alongside the
    simulated response curves and label-permutation tests.

## PCF family and dataset

The study uses **54 unique PCF structures** from a full-factorial design
grid. Each structure is represented by six optical-response channels,
each sampled at 88 wavelengths between **0.64 µm and 1.51 µm**, with a
0.01 µm spacing.

  Dataset property                Description
  ------------------------------- --------------------------------------------------------
  Base material                   Ge-As-Se chalcogenide glass
  Simulation software             COMSOL Multiphysics
  Number of structures            54
  Optical-response channels       6
  Wavelength points per channel   88
  Features per structure          528
  Design targets                  4 discrete structural parameters
  Evaluation                      Five-fold structure-wise `GroupKFold` cross-validation

### Optical-response inputs

The six wavelength-dependent channels are:

1.  Real part of the effective refractive index
2.  Imaginary part of the effective refractive index
3.  Effective mode area
4.  Chromatic dispersion
5.  Nonlinear coefficient
6.  Confinement loss

The six 88-point curves are concatenated into a 528-feature vector. Each
sample represents one complete structure, not one wavelength row.

### Structural targets

  Target                                     Values considered
  ------------------------------------------ ----------------------------------
  Air-hole diameter-to-pitch ratio (`d/Λ`)   0.65, 0.70
  Air-hole lattice shape                     Hexagonal, heptagonal, octagonal
  Number of air-hole rings                   6, 7, 8
  Anomaly position                           2nd, 3rd, 4th ring

The simulated family uses a pitch of 1.7 µm and a specified anomaly-hole
diameter. The conclusions therefore apply to this design family and
should not automatically be generalized to other materials, pitches, or
geometries.

## Methodology

The workflow consists of the following stages:

1.  **Prepare the optical-response matrix:** Combine the six response
    channels into a 528-dimensional vector for each of the 54
    structures.
2.  **Train a preliminary target-specific network:** Fit an independent
    single-head network for each structural target using the full
    spectrum.
3.  **Rank wavelengths with Gradient SHAP:** Aggregate feature
    attributions across response channels and structures to obtain a
    wavelength-importance ranking for each target.
4.  **Select wavelengths:** Retain the top third of the 88
    wavelengths---29 wavelengths per target, corresponding to 174 input
    features.
5.  **Train independent networks:** Use separate fully connected neural
    networks for ratio, shape, ring count, and anomaly position.
6.  **Evaluate:** Compare against six baseline configurations using
    structure-wise five-fold cross-validation.
7.  **Check explanations and identifiability:** Compare Gradient SHAP
    with LIME, run label-permutation tests for the difficult targets,
    and examine simulated response curves.
8.  **Validate predicted geometries:** Re-simulate the ten best
    candidate geometries in COMSOL and compare their responses with the
    target.

### Neural-network configuration

Each independent network uses two hidden layers of 128 and 64 units,
followed by a softmax output layer. The model is trained with Adam, a
learning rate of `1e-3`, weight decay of `1e-4`, cross-entropy loss, a
maximum of 300 epochs, and early stopping. Training structures are
augmented with Gaussian-noise copies; validation structures are not
augmented.
<img width="1309" height="783" alt="Picture1" src="https://github.com/user-attachments/assets/f3316acf-46b7-4e85-a95d-f2046082ca2f" />

<img width="703" height="412" alt="Picture8" src="https://github.com/user-attachments/assets/7d528b00-548a-46ce-977a-3acc4fa97192" />
<img width="722" height="412" alt="Picture7" src="https://github.com/user-attachments/assets/4bfb66d5-016e-4f72-a033-fcf354cd5058" />
<img width="685" height="353" alt="Picture6" src="https://github.com/user-attachments/assets/31a04a1e-557c-471a-a2e2-f76a839bce7d" />
<img width="680" height="417" alt="Picture5" src="https://github.com/user-attachments/assets/54b50c8c-0fae-45e5-a74c-6cbe3154b232" />
<img width="680" height="400" alt="Picture4" src="https://github.com/user-attachments/assets/1d568e93-c480-4141-a9af-98b0e45ac59c" />

## Key results

The proposed configuration achieved a mean accuracy of **0.769** across
the four targets, compared with **0.617--0.657** for the six baseline
configurations reported in the paper.

  ------------------------------------------------------------------------
  Structural target           Accuracy with proposed Interpretation
                                              method 
  --------------------- ---------------------------- ---------------------
  Diameter-to-pitch                            0.982 Reliably recovered in
  ratio                                              this dataset

  Lattice shape                                1.000 Reliably recovered in
                                                     this dataset

  Number of rings                              0.562 Improvement is
                                                     preliminary and
                                                     statistically
                                                     unconfirmed

  Anomaly position                             0.533 Improvement is
                                                     preliminary and
                                                     statistically
                                                     unconfirmed

  **Mean across                            **0.769** Aggregate accuracy
  targets**                                          across four targets
  ------------------------------------------------------------------------

### Spectral reduction

Target-specific selection reduced the input from 88 to 29
wavelengths---a **67% reduction**---while retaining an accuracy of 0.982
for the ratio target and 1.000 for the shape target.

### COMSOL re-simulation

For the ten highest-ranked predicted geometries, the paper reports:

-   Mean nonlinear-coefficient error: **0.15%**
-   Mean real effective-index error: **0.02%**
-   Mean dispersion error: **48.4%**

The large dispersion error matters: matching the effective index closely
does not guarantee that its second derivative, the dispersion, will also
match closely.

### Statistical caution

Label-permutation tests for ring count and anomaly position returned **p
= 0.077** for each target, at the floor imposed by the 12 permutations
used. The apparent improvements for these two targets therefore do
**not** meet the conventional 0.05 significance threshold and should not
be described as confirmed recovery.

## Main takeaway

The results suggest that identifiability depends on how strongly a
geometric parameter affects the guided optical mode and the resulting
response. In this simulated PCF family, the diameter-to-pitch ratio and
lattice shape leave clearer signatures in the optical response than ring
count and anomaly position.

The framework is intended to make inverse-design claims more careful: it
distinguishes parameters that the available response can recover from
parameters for which the evidence remains weak.

## Limitations and future work

-   The dataset contains 54 structures from one design family, with a
    fixed material system and pitch.
-   All responses are simulation-based; no fabricated fiber was
    measured.
-   The results do not include fabrication tolerances or experimental
    measurement uncertainty.
-   Ring-count and anomaly-position improvements remain statistically
    unconfirmed.
-   Dispersion matching remains weaker than matching the effective index
    and nonlinear coefficient.

Future work should expand the design grid across materials and pitches,
explore anomaly positions closer to the core, and include response
channels that are more sensitive to the outer cladding. Experimental
fabrication and measurement are needed before a predicted geometry can
be considered validated in practice.

