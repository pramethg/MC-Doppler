# MC-Doppler Simulation Pipeline

## 1. Initialization
*   **Define Constants:** Number of photons (`5e6`), phase function angles (`1e3`).
*   **Git Hash:** Store current commit hash for reproducibility.
*   **Geometry Setup:**
    *   Tube: Outer/Inner radii (`0.675mm`/`0.475mm`), Refractive Index (`1.5216`).
    *   Plate: Thickness (`100µm`), Z-position (`1.8mm`), RI (`1.5151`).
*   **Source Setup:**
    *   Wavelength: `633 nm`.
    *   Fibre: SM600 parameters, focus position calculation.
*   **Detector Setup:**
    *   Rotation, Axial Offset (`2.5mm`), Dimension (`0.5mm`), NA (`1`).

## 2. Build Simulation Structures
*   **Source:** Generate truncated Gaussian beam profile (`makeTruncatedRatioSource`).
*   **Sim:** Define simulation boundary (cylinder), GPU flag.
*   **Media:**
    *   Define layers: Air, PVC, Polystyrene Suspension, Glass Plate.
    *   Set optical properties (Absorption, RI).
    *   Generate spatial boundaries (`cylinderSpatial`).

## 3. Sample Specification
*   **Physical Properties:** Particle Density (`1050 kg/m³`), Temp (`300 K`), Fluid Density.
*   **Flow:** Set flow rate (`15 mL/min`).
*   **Particle Types:** Define array of bead diameters (1µm, 1.5µm, 3µm) and scattering coefficients (`mu_s`).

## 4. Simulation Loop (Per Sample)
Iterate through each sample configuration:

*   **Calculate Flow Dynamics:**
    *   Compute effective mass and Brownian motion speed standard deviation.
    *   Define velocity field: Poiseuille flow + Brownian motion component.
*   **Update Media Properties:**
    *   Calculate Phase Function using Mie theory (`simplePhaseFunction`).
    *   Update scattering coefficient (`mu_s`).
*   **Photon Execution Loop:**
    *   Initialize photon counter.
    *   **While** detected photons < limit:
        *   Run `McDopplerSim` on GPU.
        *   Filter photons with `circularDetector` (remove non-detected ones).
        *   Concatenate valid photons to `detectedPhotons` struct.
*   **Data Saver:**
    *   Construct filename with sample details.
    *   Save `.mat` file with simulation results.
