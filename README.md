# 🛡️ Nexus Sentinel — Real-Time Weapon Detection & Surveillance System

Nexus Sentinel is an AI-powered, edge-ready security surveillance system designed to detect weapons and identify personnel in real-time. Built on top of the state-of-the-art **YOLOv8** object detection framework, it leverages a custom-trained dataset of over **14,000+ images** to classify targets, trigger instant multi-threaded audio alerts, and help security personnel monitor environments efficiently.

---

## 🚀 Key Features

*   **Real-Time Threat Detection:** Low-latency object detection on webcam feed classifying objects into `weapon` (threat) and `person` (authorized/staff).
*   **Dual-Tone Audio Alarm System:** High-frequency, multi-threaded audio warning using the Windows `winsound` API. Triggered asynchronously to ensure zero lag in the video stream.
*   **Robust & Smart CUDA Training Pipeline:** Auto-recovers from CUDA Out-Of-Memory (OOM) errors during training by dynamically halving the batch size (trying `4` $\rightarrow$ `2` $\rightarrow$ `1`).
*   **Automated Evaluation:** Post-training evaluation focusing specifically on the **weapon class mAP50** metric to confirm defense-grade accuracy.
*   **COCO to YOLO Annotation Converter:** Dedicated scripts to automatically convert COCO JSON annotations into normalized YOLO coordinate formats, restructure folders, and clear index caches.

---

## 🛠️ Technology Stack

*   **Model Framework:** Ultralytics YOLOv8 (Nano/Small weights)
*   **Deep Learning Backend:** PyTorch (with CUDA acceleration optimized for NVIDIA RTX GPUs)
*   **Computer Vision:** OpenCV (cv2)
*   **Data Processing:** Pandas, Pathlib, JSON
*   **Hardware Target:** Optimized for edge deployment and local CUDA devices (e.g., RTX 4070)

---

## 📂 Project Structure

```bash
Nezussss/
│
├── train/                      # Training dataset split (images & labels)
├── valid/                      # Validation dataset split (images & labels)
│
├── train_sentinel.py           # Training orchestrator with CUDA OOM handling & validation evaluation
├── live_sentinel.py            # Live webcam inference script with multi-threaded audio/visual alerts
├── sentinel_demo.py            # Defensive mode demo script with high-confidence (70%+) filtering
├── main.py                     # Standard webcam live inference script (50% confidence)
├── demo.py                     # Camera test script using pre-trained COCO YOLOv8n weights
│
├── fix_labels.py               # Dataset utility: Converts COCO annotations to YOLO format with custom mapping
├── fix_data.py                 # Dataset utility: Moves label files to structured folders & clears YOLO caches
│
├── data.yaml                   # YOLO dataset configuration file specifying train/val paths and classes
├── README.roboflow.txt         # Dataset summary exported from Roboflow (14,989 images)
└── best.pt                     # Best weights saved from custom training (copied here for inference)
```

---

## ⚙️ Installation & Setup

### 1. Prerequisites
*   **OS:** Windows (required for the native `winsound` buzzer alert)
*   **Python:** Version 3.8 to 3.11 recommended
*   **GPU:** CUDA-enabled NVIDIA GPU (highly recommended for real-time inference and training)

### 2. Set Up Virtual Environment
Initialize a virtual environment in the project directory:
```powershell
python -m venv venv
venv\Scripts\activate
```

### 3. Install Dependencies
Install PyTorch (with CUDA support matching your system drivers) and required packages:
```powershell
# Install PyTorch, Torchvision and Torchaudio (CUDA 11.8/12.1 compatible example)
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121

# Install YOLOv8, OpenCV, and Pandas
pip install ultralytics opencv-python pandas
```

### 4. Configuration & Paths Setup
Open the Python files and update the base directory paths to match your local filesystem:
*   **`data.yaml`**: Update the `path` key to point to your workspace root directory.
*   **`train_sentinel.py`**: Update `base_dir` variables.
*   **`fix_data.py` & `fix_labels.py`**: Update the `base_dir` / `BASE` paths.

---

## 📖 Step-by-Step Usage

### Step 1: Restructure & Prepare Labels
If your dataset is downloaded in COCO JSON format, convert it to YOLO-compatible files and structure the workspace directories:
```powershell
python fix_labels.py
python fix_data.py
```

### Step 2: Train the Model
Launch the custom YOLOv8 model training. The script automatically handles batch-size fallbacks if your GPU memory fills up, trains for `10` epochs, and prints the evaluated weapon mAP50.
```powershell
python train_sentinel.py
```
*Once training completes, copy the generated best weights file from `runs/sentinel_training/weights/best.pt` to the root of this workspace.*

### Step 3: Run Live Detection
Start the real-time webcam threat detection system:

*   **Standard Live Sentinel (with alarms):**
    ```powershell
    python live_sentinel.py
    ```
*   **High-Confidence Defense Mode (70%+ confidence filter):**
    ```powershell
    python sentinel_demo.py
    ```
*   **Standard Visual Test:**
    ```powershell
    python main.py
    ```

---

## 🎯 Model Configuration Detail
The class mapping for this custom dataset is configured as:
*   `0`: `person` (Staff / Authorized Personnel)
*   `1`: `weapon` (Critical Threat Alert)
*   `2`: `undefined`

In **Defense Mode**, the threat alert is only triggered when confidence exceeds **70%**, minimizing false positives and ensuring reliable deployment in high-security environments.
