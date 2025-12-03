# Future Avenues of Research & Development

This roadmap outlines where the **2D-3D-ISO-Graph** project is heading. We’ve identified a few places where the current architecture hits a wall and mapped out how we want to fix them to make the scene graphs actually useful for public safety.

## 1. Robust 3D Instance Association
Right now, the **Global Instance Manager** is too brittle. It relies purely on Euclidean distance between centroids, which breaks down with occlusions, large objects (like walls), or drift.

**Proposed Solution:**
We need to move beyond simple geometry.
* **Visual Re-ID:** Integrate **DINOv2** or **CLIP** embeddings. If a fire extinguisher looks the same in Frame A and Frame B, merge them even if the spatial math is slightly off.
* **3D IoU & Kalman Filters:** Use 3D Intersection-over-Union for bounding boxes and Kalman filters to smooth out trajectory jitter.

## 2. Moving to Real-Time (ROS2 Integration)
The current offline pipeline (ingesting static `.png` files) is a non-starter for emergency response. First responders need live data.

We need to refactor the pipeline into a modular **ROS2 system** coupled with **RTAB-Map** or **ORB-SLAM3**. The goal is to bind semantic labels directly to SLAM KeyFrames and handle dynamic state changes (like a door opening) in real-time.

## 3. "Chat-with-Map" (LLM Querying)
Querying the current `.graphml` output requires knowing Cypher or NetworkX, which isn't viable during an incident.

**The Fix:**
* **GraphRAG:** Flatten the graph into a vector store and hook it up to an LLM (**Llama 3** / **GPT-4o**). This lets a user ask, *"Is there a fire extinguisher near the stairs?"* and get an immediate answer.
* **Validation:** We can also use RL agents to navigate the graph and prove that our topology generation actually works.

## 4. ISO 19164 Compliance & Interoperability
Our current mapping is too basic for true compliance. We capture existence, but not the detailed geometry needed for egress calculations.

To fix this, we need to extract **NavMeshes** from floor segmentation and link them to **Pathway nodes**. We also need to build exporters for **CityGML 3.0** and **IFC** so this data plays nice with standard municipal BIM software.

## 5. Better Visualization (Semantic Gaussian Splatting)
Point clouds look grainy and are hard to read. We should look into **Semantic Gaussian Splats** to get higher fidelity.

Ideally, we want a hybrid rendering: a photorealistic Gaussian Splat environment with the schematic Scene Graph overlaid. This gives users **"X-Ray vision"**—seeing the realistic room while highlighting hidden assets like standpipes or electrical panels.

---

### Contributing
If you want to tackle any of these, open an Issue tagged `research` so we can discuss the approach before you start coding.
