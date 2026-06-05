# Tests Documentation

This directory contains the test suite for the Repressilator Analysis package. The tests validate all major components of the analysis pipeline against ground truth data.

## Test Structure

```
tests/
├── README.md                        # This file
├── calibration_test.py             # Calibration conversion tests
├── fluorescence_extractor_test.py  # Segmentation, tracking, and extraction tests
├── ode_inference_test.py           # ODE simulation and parameter inference tests
├── pipeline_test.py                # (Integration tests)
├── test_utils.py                   # Utility functions for testing
└── testdata/                       # Ground truth test data
    ├── F_vs_amount.txt             # Fluorescence and position ground truth
    └── protein_numbers/            # Synthetic ODE simulation data
        ├── simulation_00x.txt      # Simulated protein time series
        └── parameters/
            └── params_00x.json     # True parameters for simulation
```

## Running Tests

### Run all tests:
```bash
pytest tests/
```

### Run specific test file:
```bash
pytest tests/calibration_test.py
pytest tests/fluorescence_extractor_test.py
pytest tests/ode_inference_test.py
```

### Run with verbose output:
```bash
pytest tests/ -v
```

### Run with print statements visible:
```bash
pytest tests/ -s
```

## Test Files

### 1. `calibration_test.py`

**Purpose**: Validate the calibration system for converting pixel intensities to protein molecule counts.

**Test**: `test_calibration()`
- Loads both protein calibration files from `docs/` 
- Reads ground truth pixel intensities and expected molecule counts from `testdata/F_vs_amount.txt`
- Converts pixel intensities using `ProteinCalibration.pixel_intensities_to_molecules()`
- **Validation criteria**:
  - No negative molecule counts
  - Mean absolute error < 170 molecules

**Key features tested**:
- Loading calibration data with header rows
- Interpolation of calibration curves
- Pixel-to-arbitrary-unit conversion (factor: 1e7)
- Mass-to-molecule conversion using molecular weights
- Handling of both nuclear and cytosolic proteins

### 2. `fluorescence_extractor_test.py`

**Purpose**: Validate cell segmentation, tracking across timepoints, and fluorescence extraction.

**Test**: `test_track_cells()`
- Loads time-series images (intensity and phase contrast)
- Segments cells and tracks them across all timepoints
- Validates cell positions and fluorescence extraction against ground truth

**Test data**: `testdata/F_vs_amount.txt`
- Contains ground truth for 80 cells per timepoint
- Columns:
  - `[0, 3]`: Nuclear and cytosolic fluorescence intensities
  - `[6]`: Cell indices
  - `[7]`: Cell ID flags
  - `[8:]`: Cell centroid positions (y, x)

**Validation criteria per timepoint**:

1. **Cell count**: Must detect exactly 80 cells
2. **Position accuracy** (tracking):
   - Uses Hungarian algorithm to match tracked cells to ground truth positions
   - At first timepoint (t=0): All cells must have position error < 7 pixels
   - Subsequent timepoints: Cell IDs must remain consistent with initial mapping
   - Position error threshold: 7 pixels
3. **Fluorescence extraction**:
   - Extracts nuclear and cytosolic intensities
   - Uses Hungarian algorithm to match fluorescence values
   - Validates both nuclear (`n_intensity`) and cytosolic (`c_intensity`)
   - Mean error threshold: < 5 intensity units
   - Standard deviation threshold: < 6 intensity units

### 3. `ode_inference_test.py`

**Purpose**: Validate the ODE model simulation and parameter inference.

**Test 1**: `test_repressillator_simulation_class()`
- Tests forward simulation of the Repressilator ODE model
- Loads true parameters from `testdata/protein_numbers/parameters/params_003.json`
- Loads expected simulation output from `testdata/protein_numbers/simulation_003.txt`
- Simulates using `RepressilatorModel.simulate()` with true parameters
- **Validation criteria**:
  - RMSE for nuclear protein (p1) < 36 molecules
  - RMSE for cytosolic protein (p2) < 36 molecules

**Test 2**: `test_infer_parameters()`
- Tests parameter inference (fitting) on synthetic data
- Extracts cell #1 data (80 timepoints) from ground truth
- Runs parameter inference using `infer_parameters()` with differential evolution
- Compares fitted parameters to true parameters
- Simulates with fitted parameters and checks fit quality
- **Validation criteria**:
  - RMSE for nuclear protein < 36 molecules
  - RMSE for cytosolic protein < 36 molecules
  - Prints standard errors for each fitted parameter

## Ground Truth Data

### `testdata/F_vs_amount.txt`

**Format**: Text file with 11,520 rows (144 timepoints × 80 cells)

**Columns** (10 total):
- `[0]`: Nuclear fluorescence intensity (arbitrary units)
- `[1]`: Nuclear protein count 
- `[2]`: Nuclear-related metadata
- `[3]`: Cytosolic fluorescence intensity (arbitrary units)
- `[4]`: Cytosolic protein count
- `[5]`: Cytosolic-related metadata
- `[6]`: Cell index (0-79 within each timepoint)
- `[7]`: Cell flag (used to identify specific cells in tests)
- `[8]`: Cell centroid Y position (row, pixels)
- `[9]`: Cell centroid X position (column, pixels)



### `testdata/protein_numbers/`

**`simulation_001.txt`**: Synthetic ODE simulation
- Column 0: Time (seconds)
- Column 1: Nuclear protein p1 
- Column 2: Cytosolic protein p2 

**`parameters/params_001.json`**: True parameter values


