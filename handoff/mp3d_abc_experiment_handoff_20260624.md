# MP3D A/B/C Imagination Experiment Handoff

Date: 2026-06-24

This document summarizes the local MP3D experiment state for the next agent.

## Goal

Build a real Matterport3D-based A/B/C dataset where:

- View A and View B are shown to a model.
- Hidden View C is a nearby novel viewpoint and is withheld during prompting.
- The question should be meaningful from A/B, but C provides oracle evidence for evaluation.
- Extra 3D modalities should be available for analysis: connectivity graph, 360 panorama/heading sweeps, graph distance, depth map, and room label.

## Key Outputs

- Curated 20 question set:
  `C:\Users\Dylan\LabStuff\IndividualResearch\outputs\mp3d_abc_view_curated20_questions_final_20260624`

- Extra modality export:
  `C:\Users\Dylan\LabStuff\IndividualResearch\outputs\mp3d_sample_modalities_20260624`

- Public GitHub Pages repo:
  `C:\Users\Dylan\LabStuff\mp3d-old-questions-viewer`

- Public URL:
  `https://dylanzhao123.github.io/mp3d-old-questions-viewer/`

- Google Drive old-question folder:
  `https://drive.google.com/drive/folders/159IHC_EHi3oxHNUmPSZhlzdXqZLGCX4j`

## Local MP3D Resources

MP3D scans live under:

`C:\Users\Dylan\LabStuff\IndividualResearch\Matterport3DSimulator\data\v1\scans`

Local scan IDs used:

- `17DRP5sb8fy`
- `1LXtFkjw3qL`
- `1pXnuDYAj8r`
- `29hnd4uzFmX`
- `2t7WUuJeko7`
- `5LpN3gDmAk7`
- `5q7pvUzZiYa`
- `759xd9YjKW5`

Current scan data directory size: about 37.22 GB.

Downloaded resource types:

- `matterport_skybox_images`
- `matterport_depth_images`
- `undistorted_depth_images`
- `undistorted_camera_parameters`
- `house_segmentations`
- `region_segmentations`

`matterport_depth_images.zip` was downloaded but left zipped because MatterSim depth rendering uses the undistorted depth package after preprocessing.

## Depth Preprocessing

MatterSim expects merged depth skyboxes:

`<viewpoint_id>_skybox_depth_small.png`

These were generated with:

`Matterport3DSimulator/scripts/depth_to_skybox.py`

Inputs:

- `undistorted_depth_images`
- `undistorted_camera_parameters`

Generated depth skybox count:

- 744 `*_skybox_depth_small.png` files
- About 0.43 GB

Important implementation note:

In one process, creating both an RGB-only MatterSim instance and a depth-enabled MatterSim instance caused `state.depth` to become all zeros. The export script now uses one depth-enabled simulator for both RGB and depth rendering.

## Curated A/B/C Samples

The curated manifest is:

`outputs\mp3d_abc_view_curated20_questions_final_20260624\curated20_manifest.json`

Each item includes:

- `view_A.png`
- `view_B.png`
- `view_C_hidden_oracle.png`
- `contact_sheet.png`
- `metadata.json`

Question usage rule:

Show only View A, View B, and `question_for_model`. Keep View C and `hidden_oracle_answer` hidden until evaluation.

## Extra Modalities

Export script:

`scripts\export_mp3d_sample_modalities.py`

Output directory:

`outputs\mp3d_sample_modalities_20260624`

For each of the 20 samples, the folder contains:

- `connectivity_topdown.png`
- `start_viewpoint_360_heading_sweep.png`
- `hidden_viewpoint_360_heading_sweep.png`
- `start_viewpoint_skybox_panorama.jpg`
- `hidden_viewpoint_skybox_panorama.jpg`
- `a_depth_raw.png`
- `b_depth_raw.png`
- `c_hidden_oracle_depth_raw.png`
- `a_depth_color.png`
- `b_depth_color.png`
- `c_hidden_oracle_depth_color.png`
- `modalities.json`

Depth encoding:

- Raw depth PNGs are uint16.
- Values are z-depth in 0.25 mm units.
- Divide by 4000 to convert to meters.
- Zero means no valid reading.

Validation result:

- 20/20 samples have room labels.
- 60/60 A/B/C perspective depth maps are valid nonzero depth renders.
- Valid depth pixel ratio range: about 22.7% to 100%.
- Median depth range: about 0.514 m to 4.690 m.

Room labels are parsed from:

`house_segmentations\panorama_to_region.txt`

Each row maps a panorama/viewpoint ID to a region index and region code.

## GitHub Pages Viewer

Local Pages repo:

`C:\Users\Dylan\LabStuff\mp3d-old-questions-viewer`

Public repo:

`https://github.com/DylanZhao123/mp3d-old-questions-viewer`

Public URL:

`https://dylanzhao123.github.io/mp3d-old-questions-viewer/`

The viewer is static:

- `index.html`
- `viewer_data.json`
- sample folders with View A/B/C images
- `modalities/` folder with modality images

Build script that copies current local outputs into the Pages repo:

`scripts\build_mp3d_pages_modalities.py`

Run from:

`C:\Users\Dylan\LabStuff\IndividualResearch`

Command:

`python scripts\build_mp3d_pages_modalities.py`

Then commit and push from:

`C:\Users\Dylan\LabStuff\mp3d-old-questions-viewer`

GitHub Actions deploys Pages on pushes to `main`.

## Main Scripts

- `scripts\download_mp3d_depth_and_labels.py`
  Downloads MP3D depth and segmentation resources for the local scan list.

- `scripts\preprocess_mp3d_skyboxes_for_mattersim.py`
  Creates MatterSim RGB skybox images from MP3D skybox resources.

- `scripts\generate_mp3d_view_abc_candidates.py`
  Generates A/B/C candidates using connectivity-neighbor C viewpoints and heading selection.

- `scripts\review_mp3d_abc_candidates.py`
  Reviews/ranks candidate sets.

- `scripts\eval_mp3d_imagination_questions_gpt.py`
  Earlier GPT evaluation helper for question quality.

- `scripts\export_mp3d_sample_modalities.py`
  Exports connectivity, 360 sweeps, graph distance, depth maps, and room labels for curated samples.

- `scripts\build_mp3d_pages_modalities.py`
  Copies curated images and modality outputs into the GitHub Pages repo and creates `viewer_data.json`.

## Regeneration Notes

The MatterSim Python module is loaded from:

`Matterport3DSimulator\build-egl`

Most MatterSim commands were run inside WSL with micromamba env `mattersim`.

Typical WSL command shape:

`wsl bash -lc "cd /mnt/c/Users/Dylan/LabStuff/IndividualResearch && env MAMBA_ROOT_PREFIX=/home/dylan/.local/share/micromamba /home/dylan/.local/bin/micromamba run -n mattersim python scripts/export_mp3d_sample_modalities.py --repo-root /mnt/c/Users/Dylan/LabStuff/IndividualResearch"`

## Caveats

- The public GitHub repo includes derived images and metadata, not the full MP3D dataset.
- Do not commit the full `Matterport3DSimulator/data/v1/scans` directory into a repo.
- Some room region codes may be coarse; keep the raw region code and region index for audit.
- The hidden C image and C-derived modalities are oracle evidence and should not be shown to the model during the target evaluation.
