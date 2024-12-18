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

**1. Interferograms set to zero mean**
  - Preparation for step number 4.

2. Apodization (Suppress the formation of sidelobes by bringing the end-points smoothly to zero. Blackman-harris filter is used).
 - Interferogram in windowed around the centerburst (N/2 points on either side) using a box-window. The box-window has the highest amplitude resolution which Agilent also uses.
 - Supported apodization functions include: 3-term Blackman-Harris, 4-term Blackman-Harris, Happ-Genzel, Flat-top.

3. Ramp-windowing the centerburst. Gronholz & Herres, _Understanding FT-IR data processing. Part 2: Details of the spectrum calculation_, 2007.
 - The centerburst in windowed with a ramp function (0 at -N/2, linear rise to 1 at N/2).
 - The ramp can be decomposed to and odd part and an even part. The even part prevents the -N/2 to N/2 range around the centerburst from being counted twice, which can lead to unrealistically high absorption intensities.
 - Agilent does not seem to include this in their spectral calculations. This can be seen as visible absorption interfaces between adjacent tiles measured using the 'Mosaic Scan' option. Inclusion here lessens this scaling problem, but does not completely remove it.

4. 2-Term zero padding
  - Length of the interferograms in increased to the next power of two. Acts as an interpolants and decreases spectral artefacts.
  - Agilent 620/670 saves interferograms of length 2056. Thus, the code pads the interferograms to a length of 4096.
    
5. Rotating the interferograms around the centerburst
  - Pulse on right side of the centerburst moved to the start of the interferogram. Pulse on the right side of the centerburst moved to the end of the interferogram.
  - "Shifts the data about the maximum value to reference the function as close to zero phase as possible", Griffiths & Haseth, _Fourier Transform Infrared Spectroscopy_, 2nd edition, 2007.

6. Fast-Fourier Transform (FFT)
  - The processed interferograms are Fourier Transformed to absorbtion spectra. At this point they are not phase corrected.

7. Phase correction
  - Extracting the amplitude spectrum from the complex output of the FFT operation.
  - Mertz phase correction algorithm is utilized.
