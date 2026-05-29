Run the program on Google Colab: https://colab.research.google.com/drive/1deYnvR8JGWiC0skYQ4PFQECQakCOSWsl#scrollTo=AXMbEuyFpxzu

========================================================================
Ocaya-Yakuphanoğlu (OY) Analysis & Diode I-V Characterization
An advanced, open-source Python tool for automated parameter extraction and series resistance (Rs) compensation in Metal-Oxide-Semiconductor (MOS) and Metal-Semiconductor (MS) Schottky devices. This repository implements the Ocaya-Yakuphanoğlu (OY) Method alongside classic diagnostic 
models (Cheung & Cheung and Norde) to analyze experimental current-voltage (I-V) characteristics and correct for severe bias-induced distortions.

------------------------------------------------------------------------
1. THEORETICAL OVERVIEW & THE OY METHOD
------------------------------------------------------------------------

In practical Schottky structures, high forward bias currents induce significant potential drops across the neutral bulk region and contact interfaces. This effect is mathematically represented as an operational series resistance (Rs), transforming the ideal Thermionic Emission (TE) 
transport equation into a non-linearly coupled system:

    I = I0 * exp( q * (V - I * Rs) / (n * k * T) )

Where:
* I0 is the reverse saturation current: I0 = A* * A * T^2 * exp( -q * Phi_Bp / (k * T) )
* Phi_Bp is the intrinsic hole barrier height (eV)
* A* is the effective Richardson constant (32 A/cm^2 K^2 for p-type Si)
* A is the active device cross-sectional area (cm^2)
* n is the diode ideality factor
* k is Boltzmann's constant
* q is the elementary charge
* T is the absolute temperature (K)

The Paradigm Shift: Dynamic vs. Constant Resistance
---------------------------------------------------
Traditional extraction frameworks like Cheung and Norde treat Rs as a fixed ohmic constant. However, as demonstrated in the accompanying manuscript (Ocaya et al., 2026), Rs is intrinsically bias-dependent and represents the instantaneous slope of the experimental curve (dV/dI). 

The apparent resistance decays exponentially under forward bias due to:
1. Surface Accumulation: Voltage-induced carrier generation elevates local interface conductivity.
2. Conduction Geometry Widening: The lateral expansion of the accumulation layer dramatically reduces the effective structural spreading resistance.
3. Barrier Thinning: High fields promote thermionic-field emission and tunneling-assisted transport.

The OY method isolates the intrinsic device metrics at the near zero-bias limit, evaluating physical parameters free from transport non-idealities and field-induced distortions.

------------------------------------------------------------------------
2. REPOSITORY FEATURES
------------------------------------------------------------------------

* Dynamic Rs Compensation: Automatically calculates instantaneous Rs(V) = dV/dI and reconstructs the true, un-distorted junction voltage (Vj = V - I * Rs).
* Batch Processing Architecture: Scan, parse, and evaluate large experimental directories filled with multi-column raw CSV datasets simultaneously.
* Multi-Model Benchmark: Integrates Cheung functions (H(I) and dV/dln(I)) and Norde formulations (F(V)) for multi-variant performance tracking.
* High-Fidelity Automated Graphics: Exports high-resolution standalone and combined multi-dataset tracking plots (ln(Rs) vs. V) suitable for direct journal publication.
* Structured Data Pipeline: Compiles processed parameters directly into polished spreadsheets (IV_summary.xlsx) alongside raw data arrays.

------------------------------------------------------------------------
3. SCRIPT ARCHITECTURE (ocaya.py)
------------------------------------------------------------------------

The standalone Python module is tailored for quick deployments across automated execution servers or local development environments (e.g., Google Colab, Jupyter Notebooks).

Core Parameter Settings
-----------------------
Modify the initialization blocks inside the source code to fit your exact test bench environments:

    T = 300.0          # Operational Temperature (K)
    A_star = 32        # Material-specific Richardson Constant (A/cm^2 K^2)
    A = 7.854e-3       # Active contact area geometry (cm^2)
    iterations = 15    # Numerical iteration convergence threshold
    current_threshold = 1e-10  # Instrument noise filter floor (A)

Numerical Execution Block
-------------------------
def process_iv(df, T, A_star, A, iterations, current_threshold):
    # Implements the numerical differentiation and regression pipeline
    # 1. Computes instantaneous central gradients for dV/dI
    # 2. Reconstructs true baseline junction arrays (Vj)
    # 3. Performs dynamic log-space fits to isolate I0, n, and Phi_B
    ...

------------------------------------------------------------------------
4. TECHNICAL PREREQUISITES & SETUP
------------------------------------------------------------------------

Ensure you have a modern Python 3.8+ environment configured. Install required analytical and performance packages directly via pip:

    pip install numpy pandas matplotlib openpyxl tqdm

Input Data Specifications
-------------------------
Your experimental files must be formatted as raw, headerless comma-separated value (.csv) files containing two clean columns:
* Column 0: Total Measured Current (I, Amperes)
* Column 1: Applied Bias Voltage (V, Volts)

------------------------------------------------------------------------
5. STEP-BY-STEP EXECUTION GUIDE
------------------------------------------------------------------------

1. Local Notebook or Server Deployment:
   Execute the script pipeline in your designated environment:
   
       python ocaya.py

2. File Ingest Strategy:
   When running on Cloud Notebook environments (e.g., Google Colab), an interactive prompt will open requesting selection of your target data matrices. You can multi-select files (e.g., sample1.csv, sample2.csv).

3. Automated Deliverable Extraction:
   Upon computation cycles completing, the script dynamically writes the following asset structural dependencies into your working workspace:
   * *_adjusted_IV.csv: Contains structured arrays mapping [Voltage, Current, Adjusted Voltage Vj, Series Resistance Rs].
   * *_logRs.csv: Contains isolated log-transformed linear parameters [Voltage, ln(Rs)] for custom plotting.
   * *_logRs.png: High-resolution standalone scatter verification plot.
   * IV_summary.xlsx: Master summary containing grouped compilation parameters across your batch.

------------------------------------------------------------------------
6. STANDARD COMPARATIVE METRIC PROFILE
------------------------------------------------------------------------

Extracted parameters using the core engine layout on a standard benchmark device (Al/p-Si/2OD-TIFDKT/Al Schottky Diode under 100 mW/cm^2 illumination) show the clear performance alignment of the OY method:

+-------------------------+----------------------+--------------------+-----------------------+-----------------------+----------------------------------+
| Extraction Methodology  | Operational Function | Saturation Cur. I0 | Ideality Factor (n)   | Barrier Height Phi_Bp | Series Resistance Rs (Ohm)       |
+-------------------------+----------------------+--------------------+-----------------------+-----------------------+----------------------------------+
| Cheung Method           | dV/dln(I)            | N/A                | 12.96                 | N/A                   | 3.38 x 10^3                      |
|                         | H(I)                 | N/A                | N/A                   | 0.672 eV              | 3.92 x 10^3                      |
+-------------------------+----------------------+--------------------+-----------------------+-----------------------+----------------------------------+
| Norde Method            | F(V)                 | N/A                | N/A                   | N/A                   | 3.05 x 10^4 -> 4.5 x 10^3        |
+-------------------------+----------------------+--------------------+-----------------------+-----------------------+----------------------------------+
| OY Method (This Code)   | Rs-Corrected Bias    | 2.578 x 10^-8 A    | 12.83                 | 0.673 eV              | 6.62 x 10^3 -> 2.93 x 10^3       |
+-------------------------+----------------------+--------------------+-----------------------+-----------------------+----------------------------------+

* Note: The Norde minimum (F(V)) can frequently be obscured or mathematically undiscernible under specialized illumination profiles, whereas the OY Method resolves robust physical constraints uninterrupted across full operational bias ranges.

------------------------------------------------------------------------
7. LICENSE & CITATION
------------------------------------------------------------------------

This pipeline framework is open-source and released under the Creative Commons Attribution 4.0 International License (CC BY 4.0). You are permitted to share, adapt, and build upon these materials for educational or commercial purposes, provided appropriate attribution is maintained.

Academic Citation References:
----------------------------
If you leverage this software code or the underlying OY physical principles in your research, academic publications, or thesis records, please formally cite the core foundational papers:

1. Ocaya, R. O., Afassinou, K., & Yakuphanoğlu, F. (2026). A new perspective on series resistance compensation: Accurate I-V Characterization of MOS/MS Devices via Ocaya-Yakuphanoğlu Analysis. University of the Free State / Firat University Manuscript.
2. Ocaya, R. O., & Yakuphanoğlu, F. (2021). Ocaya-Yakuphanoğlu method for series resistance extraction and compensation of Schottky diode I-V characteristics. Measurement, 186, 110105. https://doi.org/10.1016/j.measurement.2021.110105
========================================================================
README.txt
Displaying README.txt.
