# MC-Doppler Simulation: Polystyrene Particles in Glass Tube

## Overview
This script (`Doppler_PolystyreneParticles_GlassTube_glassPlate.m`) simulates the Doppler broadening of light scattered by polystyrene microspheres flowing through a glass tube. It employs a Monte Carlo method to model photon migration and interaction within a complex geometry involving a flow tube and a glass plate.

## Key Features
*   **Geometry:** 
    *   Glass tube (Soda lime glass) containing the fluid.
    *   Glass plate (Fused Silica) positioned in front of the detector.
    *   Surrounding media: Air, PVC.
*   **Light Source:** 
    *   Simulates an SM600 fibre (Thorlabs) at 633 nm.
    *   Modelled as a truncated Gaussian spot.
*   **Flow Dynamics:** 
    *   **Poiseuille Flow:** Parabolic velocity profile characteristic of laminar flow in a tube.
    *   **Brownian Motion:** Random thermal motion added to the flow velocity based on particle size and temperature.
*   **Detection:** 
    *   Simulates a DET10A2 detector (Thorlabs).
    *   Configurable position, orientation, and Numerical Aperture (NA).
*   **GPU Acceleration:** Utilizes CUDA-enabled GPUs for parallel processing of photon packets.

## Dependencies
*   **MC-Doppler:** The core Monte Carlo simulation library.
*   **MatScat:** (Likely required) A MATLAB library for Mie scattering calculations (`simplePhaseFunction` calls).
*   **GPU:** CUDA drivers installed and configured for MATLAB.

## Configuration
The script initializes with several configurable parameters:
*   **Geometry:** Tube radii (`tubeOuterRadius`, `tubeInnerRadius`), plate thickness, and positions.
*   **Optics:** Refractive indices for glass and fluid.
*   **Particles:** Iterates through multiple sample types defined by `beadDiameters` and scattering coefficients (`mu_s`).
*   **Physics:** Flow rate, temperature, and fluid/particle densities.

## Output
For each sample type, the script generates a `.mat` file containing:
*   `sim`: Simulation parameters.
*   `source`: Source definition.
*   `detectedPhotons`: A structure containing details of all photons that reached the detector (position, direction, path length, etc.).
*   `flowrate_mL_per_min`, `detectorAxialOffset`: Key experimental variables.
*   `commitNumber`: Git hash of the code version used.

**Output Filename Format:**
`simResults_glassTube_[Diameter]micronParticles_mu_s_[ScatteringCoeff]perMM_uniformFlowIndexMatched.mat`
