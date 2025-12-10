## EIT CT Project – Torso Conductivity Library and EIT Simulation

This repository contains the source code, scripts and drivers used to build a library of realistic finite‑element meshes and conductivity maps from thoracic CT data and to simulate electrical impedance tomography (EIT) forward and inverse problems.

### Contents

The project contains both Python and MATLAB code.  Python is used for CT processing, mesh generation, conductivity assignment and EIT forward/inverse modelling via the [pyEIT](https://github.com/eitcom/pyEIT) library.  MATLAB scripts are provided for additional meshing, electrode placement and visualisation utilities.

```
.
├── Example_driver.m             # MATLAB script demonstrating mesh loading and simulation
├── Example_load_mesh.ipynb      # Jupyter notebook that loads pre‑computed meshes, visualises them and solves forward/inverse EIT problems
├── Example_meshing.ipynb        # Jupyter notebook that builds meshes from CT segmentations using pygalmesh
├── Driver_test_OOEIT.m          # MATLAB driver showing a test of the OOEIT solver (outer orthogonal EIT)
├── requirements.txt             # Python dependencies (except pygalmesh)
├── src/                         # Python modules for CT processing, meshing and EIT simulation
│   ├── CT_processing_functions.py   # CT mask extraction, hole filling and connected‑component helpers
│   ├── mesh_generating_functions.py # Generate 2‑D meshes from binary masks, scale to millimetres and assign conductivities
│   ├── mesh_post_processing.py      # Boundary smoothing, resampling, alignment and thin‑plate‑spline deformation
│   ├── EIT_sim.py                   # Complete electrode model (CEM) forward solver and helper functions
│   ├── visualise_*.py               # Utility scripts for plotting results
│   └── __init__.py
├── src_aatae/                    # Additional Python drivers for reconstruction/search experiments
│   ├── Driver_Reconstruct.py        # Example script that searches a library of pre‑computed meshes for similar EIT measurements
│   ├── Driver_for_matlab.py         # Wrapper for calling Python reconstruction from MATLAB
│   └── __init__.py
└── src_mat/                     # MATLAB functions used by the project
    ├── Sig_update.m                 # Update conductivity values by organ label
    ├── ThinPlateSpline2D.m          # 2‑D thin‑plate spline implementation
    ├── align_contours.m             # Align boundary contours by rotation and reversal
    ├── collectAllSlices.m           # Utility to collect all mesh slices for a subject
    ├── generateElectrodesSurfaces.m # Create electrode surfaces on a mesh boundary
    ├── resample_contour_by_arclength.m # Resample boundary contours by arc length
    ├── warpAndComputeMeasurements.m    # Warp meshes and compute EIT measurements (MATLAB version)
    └── ... (other helper functions)
```

### Getting Started

This code depends on Python ≥ 3.8, [conda](https://docs.conda.io/) for environment management and a few third‑party libraries.  A MATLAB installation is needed only if you wish to run the `.m` scripts.

#### 1. Create a conda environment

Create a new environment and install the base dependencies listed in `requirements.txt`:

```bash
conda create -n eit python=3.10
conda activate eit

# install Python dependencies
pip install -r requirements.txt

```

#### 2. Install *pygalmesh* 

Mesh generation in this project uses [pygalmesh](https://github.com/meshpro/pygalmesh), a Python wrapper around CGAL.  Installing pygalmesh requires a C++14 compiler and the CGAL/Eigen libraries.  On Linux or macOS this is straightforward; on Windows it is more involved.  You can follow the instructions from this StackOverflow answer for Windows users: [“How does one install pygalmesh which depends on Eigen on a Windows PC?”](https://stackoverflow.com/questions/61472028/how-does-one-install-pygalmesh-which-depends-on-eigen-on-a-windows-pc).  Alternatively, run the notebooks in a Linux/WSL environment where `conda install -c conda‑forge cgal cgal-cpp boost eigen` followed by `pip install pygalmesh` works out of the box.

If you do not wish to use pygalmesh, you can replace it with another mesher (e.g. [pygmsh](https://github.com/pygmsh/pygmsh) + Gmsh), but the example notebooks assume pygalmesh is available.

### Data Setup

The code expects a `Data_set/` directory containing the CT volumes and associated segmentation masks for each subject.  Each subject directory should contain `ct.nii.gz` (the CT volume) and a `segmentations/` folder with organ masks named as `<organ>.nii.gz`.

When running the notebooks, a library of meshes should  be in `Python_library/` and `meshes_mat/` (for MATLAB) Ensure you have enough disk space for storing hundreds of mesh files.

### Usage

After installation and data preparation:

1. **Generate meshes** – Run `Example_meshing.ipynb` to process CT slices, extract torso and organ masks, and generate 2‑D finite‑element meshes with electrode locations.  This notebook demonstrates how to tune mesh resolution (`h` parameter) and other options.

2. **Load and simulate** – Run `Example_load_mesh.ipynb` to load a  generated mesh from the library, visualise it, assign realistic conductivity values to organs, solve the EIT forward problem using CEM "solver" (see `EIT_sim.py`). It also demonstrates deformation of meshes between slices using thin‑plate splines.

3. **Matlab utilities** – Use `Example_driver.m` or `Driver_test_OOEIT.m` to call into the MATLAB functions.  The first one show how to load meshes saved from Python (as `.mat` files), generate electrode surfaces, smooth boundaries, and call MATLAB EIT solvers.The second one implement the solution by search (described in the report) and compare it with some other inverse problem solutions of OOEIT.

4. **Two useful scripts** – The `src_aatae/Driver_Reconstruct.py` script contains an example of searching through a library of mesh slices to find those whose simulated measurements best match given EIT data(same as `Driver_test_OOEIT.m`). The `Driver_for_matlab.py` script transform a  meshe from Python-readble file to Matlab files.


### Tips

- **Paths** – Many of the scripts contain path variables pointing to example locations (e.g. `"C:\\Program Files\\distmesh-master"` or `"C:\\Users\\..."`).  Adapt these paths to match your local environment.
- **Dataset** – You will need to supply your own CT NIfTI volumes and segmentation masks; these are not included due to size(only one is in the folder `Data_set` ).  Ensure they are organised under `Data_set/<case_id>/` as described earlier.
- **Electrode placement** – The number of electrodes and their distribution on the boundary is determined during mesh generation.  Use the functions in `mesh_post_processing.py` or the MATLAB `generateElectrodesSurfaces` function to customise electrode positions.
- **Pygalmesh on Windows** – Building pygalmesh on Windows can be difficult due to CGAL/Eigen dependencies.  Consider running the notebooks in a Linux or WSL2 environment, or consult the [StackOverflow answer](https://stackoverflow.com/questions/61472028/how-does-one-install-pygalmesh-which-depends-on-eigen-on-a-windows-pc) for guidance.(Usage of an LLM like chatGPT could be helpful as well)
- **Interactive visualization with Napari** -To inspect the CT scans and  segmented volumes interactively, you can use the Napari viewer:
`pip install napari[all]`
The installation will probably fail with an error like:(if not great)
`error: Microsoft Visual C++ 14.0 or greater is required`
You need to install Microsoft C++ Build Tools --> https://visualstudio.microsoft.com/visual-cpp-build-tools/ .