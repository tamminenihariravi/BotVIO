# BotVIO Dataset Guide

Welcome to **BotVIO**! If you are new to this repository, this document will guide you on the datasets used to train and evaluate this model, and how to structure your files to use them.

## 1. What Data is the Model Trained On?
BotVIO is trained and evaluated primarily on the **KITTI Dataset**. Specifically, it relies heavily on the **KITTI Odometry Dataset** for tracking camera trajectories, combined with synchronized **IMU data** for Visual-Inertial Odometry (VIO).

The `evaluations/options.py` script reveals that by default, the codebase looks for datasets at:
- Training Data Path: `../../SLAM_Datasets/kitti_raw`
- Evaluation Data Path: `../../SLAM_Datasets/kitti_odom/`

The README notes that the exact data preparation steps match those of [Visual-Selective-VIO](https://github.com/mingyuyng/Visual-Selective-VIO), which involves interpolating the raw KITTI IMU data and associating it with the KITTI Odometry frames.

## 2. How to Arrange the KITTI Folders
In order to run the training or evaluation scripts (e.g., `eigen`, `odom` splits), your KITTI data must be organized in a very specific way. 

Below is the directory structure that BotVIO expects for the `kitti_odom` dataset (based on `datasets/kitti_dataset.py`, `evaluations/evaluate_pose_vio.py` and `datasets/mono_dataset.py`):

```text
kitti_odom/
├── sequences/                 # Contains the images and timestamps
│   ├── 00/
│   │   ├── image_2/           # Left camera color images (000000.png, 000001.png, ...)
│   │   ├── image_3/           # Right camera color images (if using stereo)
│   │   └── times.txt          # Timestamps for each frame in the sequence
│   ├── 01/
│   │   ├── image_2/
│   │   ├── image_3/
│   │   └── times.txt
│   └── ...                    # up to sequence 21
│
├── poses/                     # Contains the Ground Truth poses (for evaluation/training)
│   ├── 00.txt                 # GT poses in KITTI 3x4 flattened format for seq 00
│   ├── 01.txt
│   └── ...                    
│
└── imus/                      # Contains the extracted and interpolated IMU data
    ├── 00.mat                 # IMU data for sequence 00 in MATLAB format
    ├── 01.mat
    └── ...
```

### Details on the Folder Contents:

1. **`sequences/`**: This folder contains subfolders for each KITTI Odometry sequence (00 to 21). Inside each sequence, `image_2` contains the left RGB images, and `image_3` contains the right RGB images. The images must be named as zero-padded 6-digit integers (e.g., `000000.png` or `000000.jpg`).
2. **`poses/`**: This folder contains the ground truth trajectory `.txt` files. Each file corresponds to a sequence and contains space-separated values corresponding to a flattened 3x4 transformation matrix mapping local camera coordinates to a global frame.
3. **`imus/`**: The KITTI Odometry dataset does not natively come with ready-to-use IMU files aligned with the frames. You are expected to process the raw KITTI OXTS/IMU data and generate `.mat` (MATLAB) files containing an array/dict named `imu_data_interp`. This array contains the 6-axis IMU readings (acceleration and angular rates) interpolated to match the image frame intervals. The frequency of the IMU is expected to be sampled `10` times per image interval in this project (`IMU_FREQ = 10` in `mono_dataset.py`).

## 3. Configuration & Next Steps
When running any `.py` script under `evaluations/` (like `evaluate_pose_vio.py`), you need to pass the `--data_path` flag to point to the root directory where you structured the dataset:

```bash
python ./evaluations/evaluate_pose_vio.py --eval_data_path /path/to/your/kitti_odom/
```

Good luck exploring the BotVIO project!
