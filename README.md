# 🏸 Badminton Court Detection and Player Tracking

A computer vision system for automatically detecting and isolating the focused badminton court from multi-court video footage, with player tracking capabilities.

[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/)
[![OpenCV](https://img.shields.io/badge/OpenCV-4.8+-green.svg)](https://opencv.org/)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.0+-red.svg)](https://pytorch.org/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

![Uploading Screenshot From 2025-12-05 12-39-57.png…]()


## 🎯 Features

- **Court Detection**: Automatically detects badminton court boundaries using:
  - Edge detection (Canny + Hough Transform)
  - Color segmentation for court lines and surfaces
  - Adaptive algorithms that work across different lighting conditions

- **Focused Court Selection**: Identifies the main/active court when multiple courts are visible:
  - Size-based selection (largest court)
  - Position-based selection (center of frame)
  - Activity-based selection (optical flow analysis)

- **Court Masking**: Isolates the focused court by:
  - Masking irrelevant regions with configurable background color
  - Smooth edge feathering for natural transitions
  - Perspective-aware masking for behind-court areas

- **Player Tracking**: Tracks players on the focused court:
  - YOLO-based player detection (YOLOv8)
  - ByteTrack algorithm for consistent track IDs
  - Trajectory visualization

## 📋 Requirements

- Python 3.8+
- CUDA-capable GPU (recommended for real-time processing)

## 🚀 Installation

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/badminton-court-detection.git
cd badminton-court-detection
```

### 2. Create Virtual Environment (Recommended)

```bash
python -m venv venv
source venv/bin/activate  # Linux/Mac
# or
venv\Scripts\activate  # Windows
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Download YOLO Weights (Automatic)

The YOLO model weights will be downloaded automatically on first run. You can also pre-download them:

```bash
python -c "from ultralytics import YOLO; YOLO('yolov8n.pt')"
```

## 📖 Usage

### Basic Usage

Process a video with default settings (court masking + player tracking):

```bash
python main.py input_video.mp4 output_video.mp4
```

### Command Line Options

```bash
python main.py input.mp4 output.mp4 [options]

Options:
  -c, --config PATH       Path to configuration YAML file
  --no-mask               Disable court masking
  --no-tracking           Disable player tracking
  --mask-only             Only apply masking (no tracking)
  --preview               Show processing preview window
  --show-boundary         Draw court boundary on output
  --bg-color R,G,B        Mask background color (default: 0,0,0 = black)
  --model MODEL           YOLO model: yolov8n/s/m/l/x (default: yolov8n)
  --confidence FLOAT      Detection confidence threshold (default: 0.5)
  -v, --verbose           Enable verbose logging
  --debug                 Enable debug mode
```

### Examples

```bash
# Basic processing with player tracking
python main.py match.mp4 processed.mp4

# Mask only (no player tracking) - faster processing
python main.py match.mp4 processed.mp4 --mask-only

# Use higher accuracy model
python main.py match.mp4 processed.mp4 --model yolov8x

# Preview processing in real-time
python main.py match.mp4 processed.mp4 --preview

# Custom background color (white)
python main.py match.mp4 processed.mp4 --bg-color 255,255,255

# Show court boundary outline
python main.py match.mp4 processed.mp4 --show-boundary
```

### Python API

```python
from src.pipeline import BadmintonCourtProcessor

# Initialize processor
processor = BadmintonCourtProcessor()

# Process video file
processor.process_video(
    input_path='input.mp4',
    output_path='output.mp4',
    mask_court=True,
    track_players=True
)

# Or process frame by frame
for result in processor.process_video_generator('input.mp4'):
    court = result.court
    tracks = result.tracks
    frame = result.frame
    # Custom processing...
```

## ⚙️ Configuration

Configuration can be customized via YAML file or command line arguments.

### Default Configuration

See `config/default_config.yaml` for all available options:

```yaml
court_detection:
  method: "hybrid"  # edge, color, ml, or hybrid
  focus_selection:
    method: "largest_centered"
    size_weight: 0.7
    center_weight: 0.3

court_masking:
  background_color: [0, 0, 0]
  feather_edges: true
  mask_behind_court: true

player_detection:
  model: "yolov8n"
  confidence_threshold: 0.5

player_tracking:
  tracker: "bytetrack"
  visualization:
    show_track_id: true
    show_trajectory: true
```

## 🏗️ Project Structure

```
badminton-court-detection/
├── src/
│   ├── court_detection/
│   │   ├── line_detector.py      # Court line detection
│   │   ├── court_detector.py     # Court region detection
│   │   └── court_masker.py       # Court masking
│   ├── player_tracking/
│   │   ├── player_detector.py    # YOLO-based detection
│   │   └── tracker.py            # ByteTrack implementation
│   ├── video_processing/
│   │   └── video_handler.py      # Video I/O
│   ├── utils/
│   │   └── config.py             # Configuration management
│   └── pipeline.py               # Main processing pipeline
├── config/
│   └── default_config.yaml       # Default configuration
├── input/                        # Input videos
├── output/                       # Processed videos
├── main.py                       # CLI entry point
├── requirements.txt
└── README.md
```

## 📊 Methodology

### Court Detection Pipeline

1. **Edge Detection**: Canny edge detection with adaptive thresholds
2. **Line Detection**: Hough Line Transform to detect court lines
3. **Color Segmentation**: HSV-based detection of court surface (green/wood/blue)
4. **Region Formation**: Group lines and contours into court regions
5. **Court Selection**: Score courts by size, position, and activity

### Player Tracking Pipeline

1. **Detection**: YOLOv8 person detection filtered by court region
2. **Association**: ByteTrack algorithm for detection-to-track matching
3. **Track Management**: Maintain consistent IDs through occlusions
4. **Visualization**: Draw bounding boxes and trajectories

### Robustness Features

- **Adaptive Thresholding**: Parameters automatically adjust to image statistics
- **Temporal Smoothing**: Court detection smoothed across frames
- **No Fixed Hyperparameters**: Uses percentile-based thresholds and ratios
- **Multi-Method Detection**: Combines edge, color, and contour methods

## ⚠️ Limitations

1. **Extreme Angles**: Very oblique camera angles may reduce accuracy
2. **Poor Lighting**: Extremely dark or overexposed footage may affect detection
3. **Occluded Courts**: Heavily occluded courts may not be detected
4. **Fast Motion**: Very fast player movements may cause tracking ID switches

## 🔮 Future Improvements

- [ ] Deep learning court segmentation model
- [ ] Support for real-time camera input
- [ ] Multi-camera view support
- [ ] Pose estimation for player analysis
- [ ] Shuttlecock detection and tracking
- [ ] Match statistics generation

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [Ultralytics YOLO](https://github.com/ultralytics/ultralytics) for object detection
- [ByteTrack](https://github.com/ifzhang/ByteTrack) for multi-object tracking
- [OpenCV](https://opencv.org/) for computer vision utilities
- [Gradio](https://gradio.app/) for the web interface

## 📞 Contact

For questions or issues, please open an issue on GitHub.
