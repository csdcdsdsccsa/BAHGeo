# BAHGeo

Bidirectional Adaptive Hierarchical Network for Cross-View Object Geo-Localization

## Abstract

Cross-view object geo-localization (CVOGL) aims to locate a target in a georeferenced satellite image using a point prompt in a ground-level or unmanned aerial vehicle (UAV) query image. Existing methods often aggregate cross-view information at a single semantic level or provide insufficiently coordinated feature interactions across scales. This paper proposes BAHGeo, a Bidirectional Adaptive Hierarchical Network for CVOGL, to address these limitations. BAHGeo integrates target-centered spatial priors with hierarchical visual features through three components. First, the Hierarchical Symmetric Gaussian Position Encoding (HSGPE) module encodes the local target prior and surrounding spatial context using a core Gaussian response and an outer-ring response. Second, the Bi-Level Bidirectional Guidance Module (BLGM) establishes a coarse-to-fine guidance path, through which high-level features guide lower-level matching, and a fine-to-coarse feedback path, through which fine spatial cues refine high-level representations. Finally, the Adaptive Cross-Scale Interaction Module (ACSIM) aligns features across scales and controls feature fusion through adaptive receptive-field weighting and spatially gated residual connections. Results from ablation and generalization experiments show that BAHGeo achieves state-of-the-art performance on the evaluated cross-view object geo-localization tasks.

## Repository structure

| Path | Contents |
| --- | --- |
| `dataset/` | CVOGL loading and training augmentation |
| `model/` | BAHGeo, the shared Swin-T backbone, position encoding, and cross-view attention |
| `scripts/` | Drone-to-satellite and ground-to-satellite training and evaluation |
| `utils/` | Detection objectives, box decoding, and evaluation metrics |
| `train.py` | Training and evaluation entry point |

## Datasets

| Dataset | Access |
| --- | --- |
| CVOGL | [DetGeo benchmark instructions](https://github.com/sunyuxi/DetGeo#dataset) and [CVOGL download](https://drive.google.com/file/d/1WCwnK_rrU--ZOIQtmaKdR0TXcmtzU4cf/view). Includes DroneAerial and SVI. |
| CVOGL-Seg | [CVOS dataset instructions](https://github.com/zqwlearning/CVOS-Code#dataset) and [mask download](https://drive.google.com/file/d/1b3w0uY16uhkYI80d14pNMS2E4_ml7CAV/view). Obtain the corresponding images from CVOGL. |
| VIGOR-Building | [Zenodo release v1.0.0](https://doi.org/10.5281/zenodo.14523573) and [official dataset project](https://github.com/YuanuanLi/VIGOR-Building). |
| CVOGL-Weather | [Baidu Netdisk](https://pan.baidu.com/s/1rsNvpTN5fafq1JlESRwtPA?pwd=b4jq), extraction code `b4jq`. The archive provides five weather variants of the CVOGL DroneAerial query images and the original split manifests. Obtain the reference satellite images from CVOGL. |

The training and evaluation commands below cover the original CVOGL splits. The other dataset links identify resources used in the paper's additional experiments.

## Installation

Use a Python environment with a mutually compatible CUDA-enabled PyTorch and torchvision installation. Install the remaining dependencies with:

```bash
python -m pip install -r requirements.txt
```

The first model construction uses torchvision's pretrained Swin-T weights, which are downloaded if they are absent from the local cache.

## Data layout

Place the two CVOGL subsets under `data/`, or pass another data directory via `DATA_ROOT`:

```text
data/
├── CVOGL_DroneAerial/
│   ├── CVOGL_DroneAerial_train.pth
│   ├── CVOGL_DroneAerial_val.pth
│   ├── CVOGL_DroneAerial_test.pth
│   ├── query/
│   └── satellite/
└── CVOGL_SVI/
    ├── CVOGL_SVI_train.pth
    ├── CVOGL_SVI_val.pth
    ├── CVOGL_SVI_test.pth
    ├── query/
    └── satellite/
```

## Training and evaluation

```bash
DATA_ROOT=/path/to/data bash scripts/run_drone.sh
DATA_ROOT=/path/to/data bash scripts/run_svi.sh
```

The scripts select checkpoints by validation Acc@0.50 and evaluate the selected checkpoint on the corresponding test split. The default configuration uses one GPU, batch size 7, 24 workers, 25 epochs, learning rate 0.0001, and seed 2024. Set `PYTHON_BIN` or `OUTPUT_DIR` to override the executable or output location. Training logs and test metrics are recorded in `metrics.jsonl` and `test_metrics.json`.

To evaluate an existing checkpoint:

```bash
python train.py test --dataset CVOGL_DroneAerial --data-root /path/to/data \
  --output-dir runs/drone_eval --checkpoint /path/to/drone_model_best.pth.tar
python train.py test --dataset CVOGL_SVI --data-root /path/to/data \
  --output-dir runs/svi_eval --checkpoint /path/to/svi_model_best.pth.tar
```

## Acknowledgments

The CVOGL benchmark and original data splits are from [DetGeo](https://github.com/sunyuxi/DetGeo). The CVOGL-Seg masks are from [CVOS](https://github.com/zqwlearning/CVOS-Code), and VIGOR-Building is from [its dataset authors](https://github.com/YuanuanLi/VIGOR-Building).
