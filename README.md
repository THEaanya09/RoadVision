# RoadVision

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
