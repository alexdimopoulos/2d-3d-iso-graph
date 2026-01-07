# 2D-3D-ISO-Graph: Automated 3D Public Safety Scene Graph Generation

[Paper] [Project] [Demo] [Video]

2D-3D-ISO-Graph is a computer vision pipeline that automates the creation of 3D indoor scene graphs from 2D imagery. It generates a rich, queryable, and standardized 3D Scene Graph fully compliant with the **ISO 19164:2024(en) "Indoor feature model"** standard.

This work addresses the critical challenge of scarce and inconsistent 3D training data by pioneering a 2D-first approach. Instead of relying on sparse 3D data, the pipeline leverages robust pre-trained 2D models (YOLO, SAM, BLIP, OCR) to detect features in high-resolution images, then projects these detections into 3D space using corresponding depth data. The final output is a lightweight, structured 3D Scene Graph ideal for public safety, navigation, and location-based applications.

This project is the direct successor to the research in [PointCloudCity-Open3D-ML](https://github.com/alexdimopoulos/PointCloudCity-Open3D-ML), which demonstrated that training 3D-native ML models was computationally expensive and severely limited by data scarcity.

## Installation

### Prerequisites

- Docker with NVIDIA Container Toolkit
- CUDA-enabled GPU
- Python 3.10 or higher (for local development)

### Clone the repository and build the Docker image:

```bash
git clone https://github.com/your-org/2d-3d-iso-graph.git
cd 2d-3d-iso-graph
docker build -t 3d-yoloe-processor .
```

### Directory Structure

Organize your project directory as follows:

```
.
├── data/               # Input datasets
│   └── [area_name]/    # e.g., "area_1"
│       ├── data/
│       │   └── rgb/
│       │       └── ..._domain_rgb.png
│       └── global_xyz/
│           └── rgb/
│               └── ..._domain_global_xyz.exr
├── model_weights/      # Model checkpoints (.pt, .ts)
├── output_results/     # Generated outputs
├── scripts/            # Helper scripts
├── Dockerfile
├── run.py
├── run_all.sh
└── visualize.py
```

## Getting Started

### Basic Usage

```bash
# Launch the container with GPU access
docker run -it --rm --gpus all \
  -v $(pwd)/data:/datasets \
  -v $(pwd)/model_weights:/usr/src/app/model_weights \
  -v $(pwd)/output_results:/usr/src/app/output_results \
  3d-yoloe-processor

# Inside the container, run the pipeline
./run_all.sh /datasets/area_1

# Or run the core script directly
python run.py /datasets/area_1
```

### Visualization

```bash
# View the scene graph with Rerun visualizer
python visualize.py output_results/area_1_scene_graph.graphml
```

The output `.graphml` file can also be opened in graph visualization tools like Gephi or yEd, or loaded with Python's NetworkX library.

## Architecture

2D-3D-ISO-Graph consists of a multi-stage perception engine that processes each frame to incrementally build the 3D scene graph:

| Stage | Component | Function |
|-------|-----------|----------|
| Detection | YOLOE | Open-vocabulary object detection |
| Verification | BLIP-VQA | Confirms detections via visual Q&A |
| Text Recognition | PaddleOCR | Detects public safety signage (EXIT, AED, FIRE) |
| Context | Object365/HomeObjects | Domain-specific furniture detection |
| Segmentation | SAM | Generates precision masks for verified objects |
| Fusion | Instance Manager | Cross-frame 3D association and de-duplication |

The pipeline processes 2D images (`.png`) with corresponding 3D coordinate data (`.exr`), projecting verified 2D detections into 3D space and assembling them into an ISO-compliant scene graph.

## ISO 19164 Standard

The ISO 19164:2024(en) "Indoor feature model" provides a concise indoor feature model specifically designed for location-based services and emergency response. Unlike IFC (ISO 16739-1), which is too complex for indoor emergency situations, ISO 19164 offers a streamlined classification system.

The `CLASS_REGISTRY` maps AI-detected objects to formal ISO classifications:

| Detection | ISO Class | ISO Superclass |
|-----------|-----------|----------------|
| Fire extinguisher, AED, Exit sign | Facility | AttachedFeature |
| Stairs, Elevator, Ramp | Stair, Elevator, Ramp | Pathway |
| Door, Window, Wall | Door, Window, Wall | ConstructiveFeature |

## Pipeline Results

The system outputs:

- **Scene Graph**: ISO-compliant `.graphml` containing objects, 3D positions, sizes, and spatial relationships
- **Instance Point Clouds**: Individual `.ply` files for each detected object
- **Processing Logs**: Frame-by-frame detection and association records

## Development

To set up the local development environment:

```bash
pip install -r requirements.txt
```

## Contributing

See CONTRIBUTING.md for guidelines.

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgements

This project builds on several open-source foundations: YOLOE for open-vocabulary detection, SAM for segmentation, BLIP for visual question answering, and PaddleOCR for text recognition. We thank the maintainers of these projects and the broader computer vision research community.

## Citing 2D-3D-ISO-Graph

If you use 2D-3D-ISO-Graph in your research, please use the following BibTeX entry.

```bibtex
@misc{2d3disograph2025,
      title={2D-3D-ISO-Graph: Automated 3D Public Safety Scene Graph Generation},
      author={Your Name},
      year={2025},
      url={https://github.com/your-org/2d-3d-iso-graph},
}
```

## About

Automated generation of ISO 19164-compliant 3D scene graphs from 2D imagery for indoor public safety and navigation applications.
