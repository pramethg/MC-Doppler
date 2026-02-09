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
*   **Source Generation:**
    *   `makeTruncatedRatioSource(...)`: Generates the source structure. It defines the initial position and direction of photon packets to model a focused Gaussian beam with specific waist and divergence, truncated by the fiber NA.
*   **Media & Boundaries:**
    *   `cylinderSpatial(...)`: Takes the `media` structure with shape definitions and generates the implicit surface equations (signed distance functions) for the cylindrical (tube) and planar (plate) interfaces used for ray-tracing intersection tests.
    *   **Properties:** Sets absorption coefficients (`absorptionCoeff`) and refractive indices (`refractiveIndex`) for each layer (Air, PVC, Fluid, Glass).

## 3. Sample Specification
*   **Physical Properties:** Particle/Fluid Density, Temperature (`300 K`).
*   **Flow Configuration:**
    *   `flowRate` calculated from `15 mL/min`.
*   **Particle Types:** Arrays defining bead diameters (1µm, 1.5µm, 3µm) and their corresponding scattering coefficients (`mu_s`).

## 4. Simulation Loop (Per Sample)
Iterate through each sample configuration:

*   **Flow Dynamics:**
    *   **Velocity Field:** Defines `media.flowVelocity` as a function of position `.
    *   **Brownian Motion:** Calculates `speedStandardDeviation` and adds a random component to the Poiseuille flow vector.
*   **Optical Properties Update:**
    *   `simplePhaseFunction(...)`: Uses Mie theory to calculate the scattering phase function (angular probability distribution of scattering) and cross-sections for the specific particle diameter and refractive index.
    *   `mediaAnglePickFunction(...)`: Prepares the cumulative distribution function (CDF) for the phase function to allow efficient random sampling of scattering angles on the GPU.
*   **Photon Execution Loop:**
    *   **Execution:**
        *   `McDopplerSim(...)`: **(CORE FUNCTION)** The main GPU-accelerated Monte Carlo kernel. It propagates photon packets through the media, handling:
            *   Hop generation based on scattering/absorption coefficients.
            *   Boundary interactions (reflection/refraction).
            *   Scattering events (updating direction based on phase function).
            *   Doppler shifts (updating frequency based on flow velocity at interaction points).
        *   `gatherPhotonsGPU(...)`: Transfers the photon data from GPU memory back to the CPU.
    *   **Detection & Filtering:**
        *   `circularDetector(...)`: Filters the raw photon output. It checks which photons intersect the detector's spatial extent and fall within its acceptance angle (NA). It discards photons that miss the detector.
    *   **Data Aggregation:**
        *   `concatenatePhotonStruct(...)`: Appends the detected photons from the current batch to the cumulative `detectedPhotons` data structure.
*   **Data Saving:**
    *   Saves the `detectedPhotons` structure, simulation config (`sim`, `source`), and physical parameters to a `.mat` file.
