# Sparse Spot LASCA

Analyze IR spot videos to extract physiological signals from skin reflections.

## Overview

This tool detects bright spots in IR video recordings and extracts two signals from each spot over time:

- **PPG (Photoplethysmography)**: Mean intensity within each spot ROI - captures blood volume changes
- **LASCA (Laser Speckle Contrast Analysis)**: Speckle contrast calculated as `std / mean` - captures blood flow dynamics

## Installation

```bash
pip install -r requirements.txt
```

## Usage

### Command Line

```bash
# Basic usage
python spot_analyzer.py video.mp4

# With custom parameters and output directory
python spot_analyzer.py video.mp4 -o results/ -r 15 -t 0.25 -d 30

# Without displaying plots (save only)
python spot_analyzer.py video.mp4 -o results/ --no-show
```

#### Arguments

| Argument | Description | Default |
|----------|-------------|---------|
| `video` | Path to input video file | (required) |
| `-o, --output` | Output directory for results | None |
| `-r, --radius` | Spot radius for ROI extraction (pixels) | 10 |
| `-t, --threshold` | Detection threshold (0-1, relative to max brightness) | 0.3 |
| `-d, --distance` | Minimum distance between detected spots (pixels) | 20 |
| `--no-show` | Don't display plots interactively | False |

### Python API

```python
from spot_analyzer import SpotAnalyzer, analyze_video_file

# Quick analysis
spots, signals = analyze_video_file(
    video_path="video.mp4",
    output_dir="results/",
    spot_radius=10,
    detection_threshold=0.3
)

# Access signals
for signal in signals:
    print(f"Spot {signal.spot_id}: {len(signal.ppg)} frames")
    print(f"  PPG range: {signal.ppg.min():.1f} - {signal.ppg.max():.1f}")
    print(f"  LASCA range: {signal.lasca.min():.3f} - {signal.lasca.max():.3f}")
```

### Advanced Usage

```python
from spot_analyzer import SpotAnalyzer, Spot
import cv2

# Initialize with custom parameters
analyzer = SpotAnalyzer(
    spot_radius=15,
    detection_threshold=0.25,
    min_spot_distance=25,
    blur_kernel=7
)

# Load first frame and detect spots
cap = cv2.VideoCapture("video.mp4")
ret, frame = cap.read()
spots = analyzer.detect_spots(frame)

# Or manually define spots
manual_spots = [
    Spot(id=0, center_x=100, center_y=150, radius=12),
    Spot(id=1, center_x=200, center_y=180, radius=12),
]

# Visualize detected spots
vis = analyzer.visualize_spots(frame, spots)
cv2.imwrite("spots.png", vis)

# Analyze video with detected or manual spots
signals = analyzer.analyze_video("video.mp4", spots=manual_spots)

# Plot results
fps = cap.get(cv2.CAP_PROP_FPS)
fig = analyzer.plot_signals(signals, fps=fps, save_path="signals.png")
```

## Output

When an output directory is specified, the tool generates:

1. **detected_spots.png**: First frame with detected spots marked (green circles with IDs)
2. **signals.png**: Time-series plots showing PPG and LASCA for each spot

## Signal Extraction

### PPG (Mean Intensity)
For each spot ROI (circular region), the mean pixel intensity is computed per frame:
```
PPG(t) = mean(pixels within spot radius)
```

### LASCA (Speckle Contrast)
Speckle contrast is the ratio of standard deviation to mean intensity:
```
LASCA(t) = std(pixels) / mean(pixels)
```
Lower LASCA values indicate higher blood flow (more motion blur in speckle pattern).

## Tips

- **Threshold tuning**: Start with 0.3, decrease to detect more spots, increase to detect only the brightest
- **Spot radius**: Should match the actual spot size in pixels for optimal signal extraction
- **Min distance**: Prevents detecting multiple peaks from a single spot; set larger than spot diameter
