# MP3D Sample Modalities Export

Generated for the 20 curated A/B/C samples.

## Available outputs

Each sample folder contains:

- `connectivity_topdown.png`  
  Local top-down connectivity graph. The A/B start viewpoint and hidden C viewpoint are marked, with arrows for A/B/C headings.

- `start_viewpoint_360_heading_sweep.png`  
  Twelve perspective renders from the A/B start viewpoint, every 30 degrees.

- `hidden_viewpoint_360_heading_sweep.png`  
  Twelve perspective renders from the hidden C viewpoint, every 30 degrees.

- `start_viewpoint_skybox_panorama.jpg`  
  Preprocessed full skybox panorama for the A/B start viewpoint, when available.

- `hidden_viewpoint_skybox_panorama.jpg`  
  Preprocessed full skybox panorama for the hidden C viewpoint, when available.

- `modalities.json`  
  Per-sample metadata, including viewpoint graph distance, depth statistics, and room labels.

- `a_depth_raw.png`, `b_depth_raw.png`, `c_hidden_oracle_depth_raw.png`  
  Perspective depth maps for Views A/B/C. These are 16-bit PNGs from MatterSim; values are z-depth in 0.25 mm units, so divide by 4000 for meters. A zero value means no depth reading.

- `a_depth_color.png`, `b_depth_color.png`, `c_hidden_oracle_depth_color.png`  
  Colorized previews of the same depth maps for quick inspection.

The root file `modalities_manifest.json` summarizes all 20 samples.

## Graph distance

The export records:

- Euclidean distance between start and hidden C viewpoints.
- Shortest path distance over the Matterport connectivity graph.
- Shortest path hop count.
- Viewpoint IDs along the graph path.

For these samples, C is a direct connectivity neighbor, so graph hops are usually 1.

## Depth Maps

Depth is generated from the downloaded MP3D depth resources:

- `undistorted_depth_images`
- `undistorted_camera_parameters`
- MatterSim-compatible `*_skybox_depth_small.png` files produced by `Matterport3DSimulator/scripts/depth_to_skybox.py`

The manifest records min/median/max depth in meters and the valid-pixel ratio for each A/B/C view.

## Room Labels

Room labels are parsed from:

`house_segmentations/panorama_to_region.txt`

Each sample records the start viewpoint room label and the hidden C viewpoint room label. The raw MP3D region code and region index are preserved in `modalities.json`.

## Navigation Action and Object Detection

Added after the original modality export:

- `navigation_action` in each `modalities.json`
  Human-readable one-step movement label derived from start-to-C relative heading,
  distance, elevation, and the selected hidden-C camera heading.

- `object_detection` in each `modalities.json`
  YOLO COCO detections for View A, View B, and hidden View C. Each view includes
  labels, confidences, bounding boxes, label counts, and an annotated preview PNG.
  These are detector outputs for analysis, not MP3D ground-truth object annotations.
