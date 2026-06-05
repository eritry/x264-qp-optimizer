# x264 QP Optimization with Evolutionary Algorithms

Research code for a graduation thesis on optimizing H.264/AVC video encoding parameters with evolutionary algorithms. The project focuses on frame-level quantization parameter (QP) selection under a target bitrate constraint, with the goal of improving rate–distortion performance compared with the default x264 encoder decisions.

The experiments use a patched version of `x264` that allows QP values to be specified manually for individual frames. The optimization pipeline evaluates candidate QP vectors using PSNR and bitrate metrics and compares the proposed genetic-algorithm-based approach with x264, simulated annealing, a `(1+1)` evolutionary algorithm, and a Lagrangian relaxation baseline.

## Thesis context

Original thesis title: *Video encoding parameter selection using evolutionary algorithms*.

The research goal was to develop an algorithm for selecting H.264/AVC quantization parameters that provides the best compression quality for a given bit budget. The proposed approach was evaluated on YUV video sequences and showed better rate–distortion performance than x264 in the tested scenarios.

Key reported results from the thesis:

- Frame-level QP optimization for H.264/AVC encoding.
- Genetic algorithm for QP selection under bitrate constraints.
- Support for experiments with fixed frame types and with I/P frame type selection.
- Comparison against x264 and alternative optimization methods.
- Average BD-rate improvement reported in the thesis:
  - `-12.39%` without frame type selection.
  - `-15.84%` with I/P frame type selection.
- Training speed-up of approximately `20–30x` using an estimated fitness function based on precomputed frame statistics.

## Repository structure

```text
.
├── src/
│   ├── ga.py              # Main genetic algorithm implementation
│   ├── anneal.py          # Simulated annealing baseline
│   ├── lagrange.py        # Lagrangian relaxation baseline
│   ├── 1+1.py             # (1+1) evolutionary algorithm baseline
│   ├── collect_results.py # Result aggregation and comparison with x264
│   ├── metrics.py         # PSNR and bitrate-related metrics
│   └── utils.py           # Encoding, decoding, table loading, and helper functions
├── tables/                # Precomputed per-frame bitrate/quality lookup tables
├── dataset/
│   └── info/frames.info   # Number of frames for each dataset video
└── x264_patch/            # Patched x264 source file for manual per-frame QP control
```

## Method overview

The optimization problem is to choose encoding parameters for each frame so that video quality is maximized while the resulting bitrate stays below a target constraint.

The main optimized parameter is the H.264/AVC quantization parameter, QP. Lower QP usually gives better quality and a larger bitrate; higher QP gives stronger compression and lower quality. Instead of relying only on x264's built-in parameter selection, this project searches for better per-frame QP vectors.

The main algorithm is implemented in `src/ga.py`:

1. Generate a population of candidate QP vectors.
2. Estimate bitrate and quality using precomputed frame-level tables.
3. Select, mutate, and recombine candidates using a genetic algorithm.
4. Periodically validate promising candidates with actual x264 encoding/decoding.
5. Compare the resulting PSNR/bitrate trade-off with x264.

The repository also contains baseline methods:

- `src/lagrange.py` — QP selection using Lagrangian relaxation and binary search over the Lagrange multiplier.
- `src/anneal.py` — simulated annealing for QP vector optimization.
- `src/1+1.py` — a simple `(1+1)` evolutionary optimization baseline.

## Requirements

The code was written as research code and contains environment-specific paths. Before running it, adjust paths in `src/utils.py` to point to your local encoder and decoder binaries.

Python dependencies used by the scripts include:

```text
numpy
scipy
scikit-learn
matplotlib
tqdm
wandb
```

External tools/data expected by the code:

- x264 encoder, including the patch from `x264_patch/` for per-frame QP control.
- H.264 decoder used in the original experiments.
- Raw YUV video files placed in `dataset/`.
- `dataset/info/frames.info` with frame counts.
- Precomputed lookup tables in `tables/`.

## Dataset

The thesis experiments used YUV sequences from the Xiph.org Video Test Media dataset. The first test set contained 8 CIF videos with resolution `352x288`; the second test set was constructed by concatenating clips to evaluate scenarios with scene changes and I/P frame type selection.

The raw dataset files are not included in this repository. To reproduce the experiments, place the required `.yuv` files in `dataset/` and make sure their frame counts are listed in `dataset/info/frames.info`.

Example `frames.info` format:

```text
akiyo 300
bus 150
container 300
foreman 300
```

## Running experiments

The main entry point is:

```bash
python src/ga.py
```

Before running, configure the list of input videos and target bitrates inside `src/ga.py`. The original code is experiment-oriented: parameters such as dataset names, target bitrates, paths, and statistics output files are configured directly in the scripts.

To run baseline methods:

```bash
python src/lagrange.py
python src/anneal.py
python src/1+1.py
```

To aggregate and compare genetic algorithm results with x264:

```bash
python src/collect_results.py
```

## Notes on reproducibility

This repository is preserved as thesis research code rather than a production-ready package. Some scripts contain hardcoded paths to the original experimental environment, for example encoder/decoder binaries and dataset locations. Reproducing the exact results requires:

1. Building or providing the patched x264 binary.
2. Providing the same or compatible YUV test sequences.
3. Regenerating or validating the lookup tables in `tables/`.
4. Updating local paths in `src/utils.py`.
5. Running the experiment scripts with the same bitrate targets and training parameters.

