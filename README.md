# Processing of Agilent Interferogram Data to Spectra in MATLAB
Author: Rasmus Hiltunen <rasmus.hiltunen@uef.fi>

Note: Tested only on Agilent Cary 620/670 microspectrometer output files in MATLAB R2023a.

# About
Simple code to load Agilent interferogram files and process them to absorption spectra. Works on single tile (.seq) and mosaic (.dmt) measurements.

The codes for loading the Agilent output files heavily relies on the work by Alex Henderson (<https://github.com/AlexHenderson/agilent-ir-formats>).

Innovation for processing the interferograms taken from multiple sources of literature.
1. Werner Herress & Joern Gronholz "Understand FT-IR Data Processing"

Special mention to the codes by Lars Yunker at the University of Victoria, Victoria, B.C., Canada (<https://github.com/larsyunker/FTIR-python-tools>).

# Using the code
The code requires the following (Main.m)
1. Path to folder which contains the measurement data (.bsp & .seq / .dmt files).
2. Path to folder which contains the background data (.bsp & .seq files).

The script (AgilentSpectraFromInterferograms.m) extracts information from the measurement .bsp file (e.g., user-selected wavenumber region, name of the background file) and calculates the absorption spectra.

The following processing is done for the interferograms (in order):
1. Interferograms set to zero mean (Preparation for zero-padding in step 4.)
2. Apodization (Suppress the formation of sidelobes by bringing the end-points smoothly to zero. Blackman-harris filter is used).
3. Ramp-windowing the centerburst (To awoid wiggles in spectra)
4. 2-Term zero padding (Length of interferograms increased to the next power of two, decreases spectral artefacts)
5. Rotating the interferograms (pulse to the right of the ZPD set to start, side of the pulse to the left of the ZPD set to the end. Results in Kramers–Kronig relations) 
6. Fast-Fourier Transform (FFT)
7. Mertz-Phase correction (Makes absorption spectra more photometrically accurate)
