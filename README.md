# Repressilator Analysis

A Python package for analyzing time-series fluorescence microscopy images of bacterial cells expressing the Repressilator genetic circuit.

## Overview

The Repressilator is a synthetic genetic regulatory network consisting of three transcription repressors that form a cyclic negative feedback loop (Elowitz & Leibler, 2000). This package provides a complete analysis pipeline for quantifying protein dynamics from fluorescence microscopy:

1. **Image loading**: Read and organize time-series fluorescence and phase contrast images
2. **Cell segmentation**: Identify individual bacterial cells using phase contrast images
3. **Cell tracking**: Track cells across timepoints using spatial overlap
4. **Fluorescence extraction**: Measure fluorescence intensities for each protein in each cell
5. **Calibration**: Convert pixel intensities to absolute protein molecule counts
6. **ODE parameter inference**: Fit Repressilator model parameters using optimization

## Installation

```bash
# Clone the repository
git clone <repository-url>
cd Repressilator_analysis

# Install package in editable mode
pip install -e .
```

### Dependencies
- numpy
- scipy
- scikit-image
- matplotlib
- pandas
- pints (for ODE modeling)

## Package Structure

```
repressilator_analysis/
├── __init__.py              # Package initialization
├── image_loader.py          # Load and sort time-series images
├── fluorescence_extraction.py  # Cell segmentation, tracking, and fluorescence extraction
├── calibration.py           # Convert pixel intensities to molecule counts
├── ode_inference.py         # ODE model and parameter inference
├── pipeline.py              # Main analysis pipeline
├── utils.py                 # Utility functions
└── __main__.py              # Command-line entry point
```

## Quick Start

### Using Individual Modules

```python
from repressilator_analysis import image_loader, fluorescence_extraction, calibration

# Load images
timepoints, intensity_images, phase_images = image_loader.load_timeseries(
    "images/intensity",
    "images/phase"
)

# Track cells across time
tracks_dict, labeled_images = fluorescence_extraction.track_cells_across_time(
    phase_images,
    min_cell_area=5
)

# Load calibration
cal_nuclear = calibration.ProteinCalibration(
    "docs/Nuclear repressor 1 (66 kDa) calibration.txt",
    molecular_weight_kda=66,
    header=1
)

# Convert pixel intensities to molecule counts
pixel_intensity = 100.0
molecules = cal_nuclear.pixel_intensities_to_molecules(pixel_intensity)
print(f"Pixel intensity {pixel_intensity} = {molecules:.2e} molecules")
```

## Data Format

### Input Images

**Fluorescence Intensity Images** (`images/intensity/`)
- RGB PNG format (512×512 pixels typical)
- Naming convention: `sample_t+{time}m.png` where time is in minutes
- Red channel: Cytosolic repressor fluorescence
- Green channel: Nuclear/nucleoid repressor fluorescence
- Blue channel: (unused)

**Phase Contrast Images** (`images/phase/`)
- Grayscale or RGB PNG format
- Used for cell segmentation
- Must correspond 1:1 with intensity images

### Calibration Data

Located in `docs/` directory:
- `Nuclear repressor 1 (66 kDa) calibration.txt`
- `Cytosolic repressor (53 kDa) calibration.txt`

Format:
```
Mass/ng    repeat 1/A.U.    repeat 2/A.U.    repeat 3/A.U.
2.500e+01  5.969e+02        6.228e+02        8.278e+02
...
```

## Testing

Run the test suite:
```bash
pytest tests/
```

Tests include:
- Calibration accuracy validation
- Cell segmentation and tracking with ground truth positions
- Fluorescence extraction validation
- ODE model simulation accuracy
- Parameter inference on synthetic data

See `tests/README.md` for detailed testing documentation.

## References

- Elowitz, M. B., & Leibler, S. (2000). A synthetic oscillatory network of transcriptional regulators. *Nature*, 403(6767), 335-338. (See `docs/35002125.pdf`)


