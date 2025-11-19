# zaic2025-namanh-aeroeyes
This is the repository storing the source code of Nguyen Nam Anh's model attending Aeroeyes track in Zalo AI Challenge 2025.

# 🦅 ZAIC 2025: AeroEyes Spotter-Chaser Tracker (YOLOv8 & Optical Flow)

## 🏆 Competition and Objective

This project represents the solution developed for the **Zindi Africa Innovation Challenge (ZAIC) 2025: AeroEyes** competition. The core task is a challenging **Few-Shot Object Tracking** problem:
* Given a small set of **support images** of an unknown object.
* The system must detect and track that object throughout a long, unseen drone video.

The submitted solution uses a high-performance **Spotter-Chaser** architecture to achieve efficiency and robustness.

## 💡 Method: The Spotter-Chaser Architecture

The tracking system is designed to leverage the strengths of modern CNNs for precise object recognition and the efficiency of classic computer vision for continuous, frame-rate tracking.

### Stage 1: The Spotter (Detection & Verification)

The Spotter runs on a fixed cadence defined by $SPOTTER\_FPS = 20$, reducing the computational load of the most expensive steps.

| Step | Technique | Detail |
| :--- | :--- | :--- |
| **1. Detection** | **YOLOv8n Fine-Tuning** | A pre-trained `yolov8n.pt` model is fine-tuned on the multi-class training dataset to detect generic objects (car, person, ship, etc.). The confidence threshold is lowered to $0.25$ to ensure potential targets are not missed, generating more candidate boxes. |
| **2. Feature Extraction** | **ResNet50 Embeddings** | A pre-trained **ResNet50** model (truncated before the final FC layer) is used as a feature extractor. The output is a $2048$-dimensional feature vector (global average-pooled features) for any cropped image. |
| **3. Verification** | **Cosine Similarity** | The cosine similarity is calculated between the candidate object's embedding and the average prototype embedding (proto) derived from the support images. |
| **4. Activation** | **Threshold Check** | If $Similarity \ge EMBED\_SIM\_THRESHOLD$ ($0.30$), the object is confirmed as the target, and the Chaser is activated on the bounding box with the highest similarity score. |

### Stage 2: The Chaser (Lucas-Kanade Optical Flow)

The Chaser takes over immediately after a successful spot and runs on every frame until tracking fails, providing high temporal resolution.

| Step | Technique | Detail |
| :--- | :--- | :--- |
| **1. Keypoint Initialization** | **Shi-Tomasi Features** | Keypoints are initialized within the spotted bounding box using `cv2.goodFeaturesToTrack` (max $100$ corners). These points serve as anchors for tracking the object's movement. |
| **2. Tracking** | **Lucas-Kanade PyrLK** | The `cv2.calcOpticalFlowPyrLK` function is used to calculate the motion vector of the keypoints from the previous frame (`old_gray`) to the current frame (`frame\_gray`). |
| **3. BBox Update** | **Min/Max Bounding Box** | The new bounding box coordinates $(x1, y1, x2, y2)$ are determined by calculating the **minimum** and **maximum** x and y values of the successfully tracked keypoints. This adaptive, "shaky" logic minimizes drift. |
| **4. Termination** | **Failure Condition** | The Chaser terminates if fewer than 4 keypoints are successfully tracked (lost object), or the video ends, handing control back to the Spotter. |

## ⚙️ Model Training and Parameters

### YOLOv8 Fine-Tuning

| Parameter | Value | Description |
| :--- | :--- | :--- |
| **Base Model** | `yolov8n.pt` (Nano) | Chosen for speed and GPU memory efficiency. |
| **Data Config** | `data.yaml` | Dynamically generated from training videos, classifying objects into 5 categories. |
| **Epochs** | $10$ | Short training duration due to the use of heavy augmentation. |
| **Patience** | $5$ | Early stopping is enabled: training stops if the validation metric does not improve after 5 epochs. |
| **Augmentations** | Heavy | Includes rotation (30 deg), translation (0.1), scaling (0.4), vertical flip (0.5), and robust HSV adjustments to simulate diverse drone capture conditions. |

### Tracking Parameters

| Parameter | Value | Logic in Frames |
| :--- | :--- | :--- |
| `SPOTTER_FPS` | $20$ | Determines $\text{step} = \text{max}(1, \text{round}(\text{video\_fps} / 20))$. |
| `EMBED\_SIM\_THRESHOLD` | $0.30$ | Cosine similarity threshold for verification. |
| `MIN\_TRACK\_LENGTH\_FRAMES` | $5$ | Minimum required length for a Chaser track to be included in the final submission. |

## 🛠️ Project Structure and Execution

### File Structure
ZAIC-AeroEyes-YOLO-Chaser/ ├── README.md <-- This file: documentation ├── requirements.txt <-- Python dependency list ├── zaic2025-namanh-aeroeyes-v1-5-training.ipynb <-- Main Codebase ├── data.yaml <-- YOLO dataset config └── runs/detect/train/weights/best.pt <-- Fine-tuned YOLO weights

### Installation

```bash
# 1. Clone the repository
git clone [https://github.com/YourUsername/ZAIC-AeroEyes-YOLO-Chaser.git](https://github.com/YourUsername/ZAIC-AeroEyes-YOLO-Chaser.git)
cd ZAIC-AeroEyes-YOLO-Chaser

# 2. Install dependencies (Crucial for numpy<2.0)
pip install -r requirements.txt

```
## Execution
The entire workflow is managed within the Jupyter Notebook:

Run Cell 3 & 4: Prepare the data and data.yaml.

Run Cell 5: Fine-tune the YOLOv8 model.

Run Cell 6 & 7: Define and load the ResNet50 feature extractor.

Run Cell 9: Execute the Spotter-Chaser logic on the public test set and generate submission.json.
