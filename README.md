# 2D-3D-ISO-Graph: Automated 3D Public Safety Scene Graph Generation

<p align="center">
  <strong>Generates spatial intelligence for indoor environments by building ISO 19164-compliant 3D scene graphs from a 2D AI pipeline.</strong>
</p>


This project introduces a state-of-the-art computer vision pipeline that automates the creation of 3D indoor maps. It directly addresses the critical industry challenge of scarce and inconsistent 3D training data by pioneering a 2D-first artificial intelligence approach.

The system generates a rich, queryable, and standardized 3D Scene Graph that is fully compliant with the **ISO 19164:2024(en) "Indoor feature model"** standard.

This work is the direct successor to the research in the `PointCloudCity-Open3D-ML` repository. Our previous findings demonstrated that training 3D-native ML models (like KPCONV) was computationally expensive and severely limited by the scarcity of consistently labeled 3D training data.

This project overcomes those challenges by:

* **Pioneering a 2D-First Approach**: Instead of relying on sparse 3D data, this pipeline leverages robust, pre-trained 2D models (YOLO, SAM, BLIP, OCR) to detect features in high-resolution images.
* **Intelligent 2D-to-3D Projection**: It projects high-confidence 2D detections into 3D space using corresponding depth data, effectively creating 3D labels from 2D intelligence.
* **Generating an Efficient Scene Graph**: The final output is not another massive, unstructured point cloud, but a lightweight, structured 3D Scene Graph. This format is efficient, easy to query, and ideal for public safety, navigation, and location-based applications.

The goal is a fully automated system that can ingest raw scan data (images + 3D points) and produce a high-fidelity, standardized, and semantically-rich digital twin of an indoor environment.

### 🎥 Project Demo

<p align="center">
  <a href="https://www.youtube.com/watch?v=xeld0KfRyXo" title="Watch the Project Demo">
    <img 
      src="https://img.youtube.com/vi/xeld0KfRyXo/hqdefault.jpg" 
      alt="Project Demo Video" 
      width="560">
  </a>
</p>


## 📜 Table of Contents

* [Core Capabilities](#-core-capabilities)
* [The 3D Scene Graph & ISO 19164 Standard](#-the-3d-scene-graph--iso-19164-standard)
* [Architecture & Data Flow](#-architecture--data-flow)
* [Setup & Execution](#-setup--execution)
    * [Prerequisites](#prerequisites)
    * [Directory Structure](#1-directory-structure)
    * [Build the Docker Image](#2-build-the-docker-image)
    * [Run the Pipeline](#3-run-the-pipeline)
    * [Visualize the Results](#4-visualize-the-results)

---

## 🚀 Core Capabilities

* **Advanced 2D Perception Engine**: Orchestrates several state-of-the-art models for comprehensive scene understanding:
    * **YOLOE**: Open-vocabulary object detection.
    * **BLIP-VQA**: Verifies YOLOE's findings (e.g., Q: "Is this a fire extinguisher?" A: "Yes.") to reduce false positives.
    * **PaddleOCR**: Detects and reads text to find critical public safety features like 'EXIT' signs, 'AED' placards, and 'FIRE' labels.
    * **Domain-Specific Detection**: Specialist models (Object365/HomeObjects) for contextual objects and furniture.
    * **SAM (Segment Anything Model)**: Generates high-precision masks for all final, verified objects.
* **Intelligent Fusion & De-duplication**: A robust "specialist-first" fusion logic prioritizes high-trust detections (e.g., VQA-verified and OCR-detected) over generalist detections, resolving conflicts and minimizing noise.
* **Cross-Frame 3D Instance Association**: A "Global Instance Manager" tracks objects in 3D space across multiple frames. By comparing 3D centroids, it intelligently merges observations of the same object (e.g., a fire extinguisher seen from three different angles) into a single, unified instance.
* **ISO-Compliant Scene Graph Generation**: The primary output is a `.graphml` scene graph that strictly adheres to the ISO 19164:2024(en) standard. This provides a "core semantic classification system" specifically for location-based services and emergency response.

---

## 🏛️ The 3D Scene Graph & ISO 19164 Standard

> A point cloud is a massive collection of points. A **3D Scene Graph** is an intelligent, structured database that understands relationships. It knows a `Building` contains a `Floor`, which contains a `Room`, which in turn contains a `Facility` (like a fire extinguisher).

While other standards like IFC (ISO 16739-1) exist, they are often "too complex to be used in their current format for indoor emergency situations." The **ISO 19164:2024(en) "Indoor feature model"** is designed as a "relatively independent and concise indoor feature model" specifically for this purpose.

### How This Project Implements ISO 19164

Our AI pipeline automatically populates this standard. The `CLASS_REGISTRY` in the code maps AI-detected objects directly to their formal ISO classifications:

* **Public Safety (Facility)**: Objects like "fire extinguisher," "AED," and "emergency exit sign" are classified as:
    * `iso_class`: "Facility"
    * `iso_superclass`: "AttachedFeature"
* **Egress (Pathway)**: "Stairs," "elevators," and "ramps" are classified as:
    * `iso_class`: "Stair", `iso_class`: "Elevator", etc.
    * `iso_superclass`: "Pathway"
* **Structure (ConstructiveFeature)**: "Doors," "windows," and "walls" are classified under:
    * `iso_class`: "Door", `iso_class`: "Window", etc.
    * `iso_superclass`: "ConstructiveFeature"

The output `.graphml` file is this ISO-compliant data model, containing not just the objects but also their 3D positions, sizes, and relationships, ready for any compliant application.

---

## ⚙️ Architecture & Data Flow

The system processes each frame to incrementally build the 3D scene graph.

1.  **Ingest**: The system loads a 2D image (`.png`) and its corresponding 3D coordinate data (`.exr`).
2.  **2D Perception & Verification**:
    * YOLOE finds all potential objects.
    * BLIP-VQA confirms each YOLOE detection.
    * PaddleOCR finds and reads all text, matching public safety keywords.
3.  **2D Contextual Detection**: Object3Example and HomeObjects models run to find general furniture and obstacles.
4.  **Fuse & Segment**: All 2D boxes are de-duplicated using the hierarchical, specialist-first logic. The final, clean set of boxes is fed to SAM to generate precise pixel masks.
5.  **Project & Extract**: Each 2D mask is used to extract the corresponding 3D points from the `.exr` file.
6.  **3D Association & Graph Assembly**: The Global Instance Manager analyzes the 3D centroid of each new object.
    * **New Object**: A new instance ID is created, a `.ply` file is saved for it, and a new node is added to the scene graph.
    * **Existing Object**: The new 3D points are merged with the existing object's `.ply` file, refining its shape.
7.  **Output**: The `scene_graph.graphml` is saved to disk incrementally after each new object is added, ensuring robustness against interruptions.

---

## 🔧 Setup & Execution

The entire pipeline is containerized for easy and reproducible execution using Docker and NVIDIA GPUs.

### Prerequisites

* **Docker**: [Install Docker](https://docs.docker.com/get-docker/)
* **NVIDIA GPU**: A CUDA-enabled GPU is required.
* **NVIDIA Container Toolkit**: Required for Docker to access the GPU. [Install instructions](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html).

### 1. Directory Structure

The script expects a specific layout for data, models, and outputs. Your local project directory should be organized as follows. The `docker run` command will map these local folders to the correct paths inside the container.

```bash
.
├── 3d_data/            <-- (Optional: For storing/accessing other 3D assets)
├── scripts/            <-- (Contains helper scripts for the pipeline)
├── model_weights/      <-- (Stores .pt, .ts, etc. Mounted to /usr/src/app/model_weights)
├── output_results/     <-- (Git-ignored. For all generated files. Mounted to /usr/src/app/output_results)
├── data/               <-- (Input datasets. Mounted to /datasets)
│   └── [area_name]/    <-- e.g., "area_1". This is the path you'll pass to the script.
│       ├── data/
│       │   └── rgb/
│       │       └── ..._domain_rgb.png
│       └── global_xyz/
│           └── rgb/    <-- Note: The EXR finder expects this 'rgb' subfolder
│               └── ..._domain_global_xyz.exr
│
├── .dockerignore       <-- (Specifies files to ignore during Docker build)
├── commands.txt        <-- (Reference for useful commands)
├── requirements.txt    <-- (Python dependencies for the project)
├── run_all.sh          <-- (Main execution script to run the full pipeline)
```
### 2. Build the Docker Image
From the root of the project directory, build the image:

```bash
docker build -t 3d-yoloe-processor .
```
### 3. Run the Pipeline
This command launches the container, mounts your local directories, and gives you an interactive terminal.

```bash
docker run -it --rm --gpus all \
  -v $(pwd)/data:/datasets \
  -v $(pwd)/model_weights:/usr/src/app/model_weights \
  -v $(pwd)/output_results:/usr/src/app/output_results \
  3d-yoloe-processor
Once inside the container, run the main script. You must pass the path to the specific area you want to process (e.g., area_1):
```
```bash

# Inside the Docker container:
# You can run the full pipeline using the shell script:
./run_all.sh /datasets/area_1

# Or, to run the core script manually:
python run.py /datasets/area_1
The script will begin processing all images in /datasets/area_1/data/rgb/, saving all outputs (logs, graphs, and instance .ply files) to your local output_results/ directory.
```
### 4. Visualize the Results
After the pipeline runs, you can visualize the generated scene graph and point clouds:

Scene Graph: The output output_results/[area_name]_scene_graph.graphml is a standard .graphml file that can be opened in tools like Gephi or yEd, or loaded with Python's NetworkX library for analysis.

3D Visualization: Use the included Rerun-based visualizer script (visualize.py) to view the final scene graph overlaid on the generated 3D point cloud.
├── Dockerfile          <-- (Definition for the Docker container)
├── visualize.py        <-- (Rerun-based script to visualize results)
└── run.py              <-- (Core Python script for the pipeline, likely called by run_all.sh)
