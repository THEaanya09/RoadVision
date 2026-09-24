# RoadVision


## Model and Training Configuration

Best candidate by mAP50: **Experiment 7**.

| Parameter | Value |
|---|---|
| Model | YOLO11n |
| Image size | 640 |
| Epochs | 80 |
| Optimizer | AdamW |
| Initial learning rate (`lr0`) | 0.00025 |
| Final LR factor (`lrf`) | 0.01 |
| Weight decay | 0.0005 |
| Warmup | 3 epochs |
| Augmentation | Moderate |
| Seed | 42 |
| Deterministic | Yes |

## Experiments

Several controlled experiments were run, changing one factor at a time relative to a baseline:

- stronger augmentation
- longer training
- higher resolution (960)
- YOLO11s (larger model)
- D20 oversampling
- loss-weight changes
- lower learning rate

**Summary:** Experiment 7 gave the strongest overall **mAP50**. Experiment 9 gave the strongest overall **mAP50-95**, but neither materially solved the overall detection problem. None of the variations changed the conclusion that the dataset is too small.

## Results

### Best current model: Experiment 7 (test set, 21 images)

| Precision | Recall | mAP50 | mAP50-95 |
|---|---|---|---|
| 0.3910 | 0.1175 | 0.0772 | 0.0191 |

### Per-class (test set)

| Class | Precision | Recall | mAP50 | mAP50-95 |
|---|---|---|---|---|
| D00 | 1.0000* | 0 | 0 | n/a |
| D10 | 0 | 0 | 0.00391 | 0.00176 |
| D20 | 0.0646 | 0.1081 | 0.0282 | 0.00678 |
| D40 | 0.4993 | 0.3617 | 0.2766 | 0.0677 |

\* D00 precision of 1.0 with recall 0 is a degenerate result (effectively no useful detections), not a sign of good performance.

**Reading these numbers:** only D40 shows moderate signal. D20 shows weak signal. D00 and D10 are effectively undetected. With 21 test images, per-class figures are highly variance-prone.

## Error Analysis

The analysis included confidence-threshold sweeps, IoU-based matching, confusion matrices, a focused D20/D40 investigation, and qualitative inspection of predictions.

**Findings**

- **D20** showed some localization signal, but detection quality was weak.
- **D40** had comparatively better localization, but confidence stayed low.
- **D20 and D40 were meaningfully confused with each other.**
- **Very low confidence thresholds** produced many candidate boxes, but these were not practically useful detections.
- **Threshold tuning does not fix the underlying limitation.** Lowering the threshold increases candidates and false positives without a real gain in usable detections. The limiting factors are model/data capacity, not the operating point.

## Example Predictions / Inference

A final prediction run was performed on the 21-image test set:

| Setting | Value |
|---|---|
| Confidence threshold | 0.05 |
| IoU threshold | 0.7 |
| Image size | 640 |

Annotated predictions and label outputs were saved.

### Qualitative comparison (ground truth vs. baseline vs. fine-tuned)

The panels below are shown deliberately, including the failures. Predictions appear rendered at very low confidence thresholds, which is why many overlapping low-score boxes are visible. This illustrates the finding above: at low thresholds the model produces large numbers of candidate boxes that are not useful detections.

![Ground truth vs baseline vs fine-tuned, example set 1](assets/images/gt_baseline_finetuned_grid.jpeg)

![Ground truth vs baseline vs fine-tuned, example 2](assets/images/gt_baseline_finetuned_1.jpeg)

![Ground truth vs baseline, example 3](assets/images/gt_baseline_2.jpeg)

![Ground truth vs baseline, example 4](assets/images/gt_baseline_3.jpeg)

## Limitations

- **Small dataset:** 117 images (64 labeled), 21 test images. Metrics are noisy and not statistically robust.
- **Weak performance:** overall test mAP50 is 0.077 and mAP50-95 is 0.019.
- **Zero-signal classes:** D00 and D10 have effectively no useful detection performance.
- **Class confusion:** D20 and D40 are meaningfully confused.
- **Duplicates not fully removed:** possible leakage across splits; numbers may be optimistic.
- **Class definitions not formally verified** for this dataset.
- **No external or cross-domain validation** has been done yet.
- **No deployment.** There is no API, frontend, container, or hosted demo.
- **Not production-ready** and should not be used for real inspection decisions.

The project currently demonstrates the **experimentation and evaluation process** more strongly than production performance.

## Future Work

> Everything in this section is **planned, not implemented.**

**Data**
- Expand with legitimate public/real-world road-defect datasets
- Verify dataset licenses and class definitions
- Harmonize labels across sources
- Content-hash deduplication across all sources
- Create a new leakage-free, deterministic split
- Improve D20 and D10 coverage

**Modeling and evaluation**
- Retrain on the expanded data
- Evaluate on the new held-out test set **and** on the original Custom-City test set (cross-domain generalization)
- Compare before/after with IoU-based error analysis

**Product (only if results justify it)**
- FastAPI inference API
- Minimal React frontend
- Docker image and public deployment

## Repository Structure

> Adjust to match the actual repository layout.

## How to Reproduce

1. Install dependencies:
```bash
   pip install ultralytics
```
2. Place the dataset and the frozen split manifest under `data/` and point the dataset YAML at the train/val/test lists.
3. Train Experiment 7:
```bash
   yolo detect train model=yolo11n.pt data=configs/data.yaml \
     imgsz=640 epochs=80 optimizer=AdamW lr0=0.00025 lrf=0.01 \
     weight_decay=0.0005 warmup_epochs=3 seed=42 deterministic=True
```
   Augmentation settings for the "moderate" profile should match the values recorded in the experiment notebook.
4. Evaluate on the test split:
```bash
   yolo detect val model=runs/detect/<exp>/weights/best.pt data=configs/data.yaml split=test imgsz=640
```
5. Run final inference:
```bash
   yolo detect predict model=runs/detect/<exp>/weights/best.pt \
     source=<test_images_dir> conf=0.05 iou=0.7 imgsz=640 save=True save_txt=True
```

Exact results may vary slightly across hardware and library versions.

## Tech Stack

- Python
- Ultralytics YOLO11
- PyTorch
- Jupyter / Google Colab (experimentation)
- NumPy, Matplotlib (analysis and visualization)

## Current Project Status

**Experimental / research prototype.** Data pipeline, split, experiments, and error analysis are complete for the current small dataset. Dataset expansion and any deployment work have not started.

## License / Dataset Attribution

- **Code license:** _to be added by the repository owner._
- **Dataset:** the current dataset is a small custom set. **Source, provenance, and license terms for every image are not yet fully documented.** Some images may originate from third-party or stock sources. Do not redistribute the dataset or derived weights until provenance and licenses are verified.
- Any future public datasets will be listed here with their licenses and citations.
