# TrainCheck Artifact Evaluation Guide

Welcome to the artifact evaluation guide for **TrainCheck** (OSDI'25). This document outlines the procedures needed to reproduce our results and guides you through the key experiments presented in the paper.

> **Note:** We may update both the main TrainCheck repository and the evaluation workloads repository during the evaluation period.  
> Please make sure to **pull the latest version** of each repository before proceeding.

## ✅ Checklist

- [ ] Environment set up (Python, dependencies, 2 CUDA GPUs with ≥ 12GiB memory each)
- [ ] Ran **[Silent Issue Detection](#eval-silent-issue-detection)** experiment
- [ ] Ran **[Invariant Transferability](#eval-transferability)** evaluation
- [ ] Ran **[False Positive Rate](#false-positive-rate)** evaluation
- [ ] Ran **[Performance Overhead](#eval-performance-overhead)** measurement
- [ ] Verified outputs match expected results (tolerances noted per experiment)

## 📎 Additional Resources

In addition to this guide, you will need the following resources throughout the evaluation process:

1. [**5-Minute Tutorial**](./5-min-tutorial.md) — A quick walkthrough that introduces TrainCheck’s workflow using a real-world bug.
2. [**TrainCheck Installation Guide**](./installation-guide.md) — Step-by-step instructions for setting up TrainCheck.
3. [**Technical Usage Guide**](./technical-doc.md) — Detailed documentation on how to use TrainCheck, configure instrumentation, and interpret outputs.
4. [**Evaluation Workloads Repository**](https://github.com/OrderLab/TrainCheck-Evaluation-Workloads) — Contains all evaluation workloads used in the experiments.

## 1. Overview

**TrainCheck** is an invariant-based tool for detecting silent correctness issues in PyTorch training pipelines.

This artifact enables reproduction of the four main evaluation results from the paper:

- **[Silent Issue Detection (Section 5.1)](#eval-silent-issue-detection)**
- **[Invariant Transferability (Section 5.3)](#eval-transferability)**
- **[False Positive Rate (Section 5.4)](#false-positive-rate)**
- **[Performance Overhead (Section 5.5)](#eval-performance-overhead)**

To get familiar with TrainCheck, we recommend starting with the [**5-Minute Tutorial**](./5-min-tutorial.md), which walks you through detecting a real-world bug from Section 5.1.

### ⏱️ Recommended Evaluation Order

We suggest running the evaluations in the following order, based on automation level and runtime requirements:

1. Kick the tires – [5 min tutorial with TrainCheck](./5-min-tutorial.md)
2. Performance Overhead (~10 minutes)
3. False Positive Rate (~1.5 hours)
4. Transferability (~30 minutes)
5. Silent Issue Detection (~ variate, should be able to finish within one day)

## 2. Environment Requirements

For a full and efficient AE experience, we recommend the following setup:
- 🖥 1 machine with 2× CUDA-enabled GPUs
- Each GPU should have at least 12 GiB memory.
- Compatible with CUDA 11.8 or 12.1
- 🧠 32 host memory (recommended)

### 🔧 Recommended Hardware: Chameleon Cloud

Most experiments require **2× CUDA-enabled GPUs** with support for **CUDA 11.8+**. While some workloads can run on GPUs with as little as 2 GiB memory, the main experiments (e.g., Section 5.1) benefit from higher-capacity GPUs.

We recommend using the `compute_liqid` node type on [Chameleon Cloud](https://www.chameleoncloud.org):

- ✅ `liqid01` and `liqid02`:  
  These nodes each have **2× A100 GPUs (40 GiB)** and allow you to reproduce **all results** in the paper.

- 🆗 Other `compute_liqid` nodes with **1× A100 GPU**:  
  These are sufficient for all **single-GPU experiments** and let you reproduce **~90%** of results.

Please consult the estimated runtimes in each evaluation section before making reservations.  
⏱️ If working full-time on the artifact, **2 days should be sufficient**, but we recommend reserving **at least 5 days** to allow for possible setup delays or debugging.

### Software Notes

1. If you’re using Chameleon instances:
    - Please start your machine with an Ubuntu 22.04 image that includes recent GPU drivers.
    - (TBD: we will provide the specific image ID or setup instructions once finalized.)

2. Follow [Installation Guide](./installation-guide.md) to install TrainCheck.

⏭️ Once your environment is set up, we recommend starting with the [5-Minute Tutorial with TrainCheck](./5-min-tutorial.md).
It will help you get familiar with the workflow and also verify that your installation is working correctly.

## Eval: Silent Issue Detection

## Eval: False Positive Rate

⏳ Estimated Completion Time: 2 hour.
- Trace Collection: ~10 minutes
- Invariant Inference & Checking: ~1.5 hours

### 🎯 Goal

This evaluation measures the false positive rate of alarms from TrainCheck's invariants.

### 📂 Resources & Scripts

- Automation Scripts:
    1. `traincheck-ae-resources/fp_rate/ae_fp.py`
- Workloads:
    1. The PyTorch official pipelines that will be used has been prepared at `traincheck-ae-resource/fp_rate/workloads`. We have shortened training for these scripts such that the experiments run fast without . For AE purposes, you won't need to know the details as the automation scripts will automatically handle the running of these scripts.

### 🛠 How to Run

0. Make sure you have a working TrainCheck installation by following [TrainCheck Installation Guide](./installation-guide.md).

1. Install necessary dependencies for the false positive evaluation workloads.
```bash
conda activate traincheck # change this if you installed TrainCheck in a different environment.
cd fp_rate
pip3 install -r requirements.txt
```

2. Execute `ae_fp.py`
The workload `ddp-multigpu` will need 2 GPUs. We have provided the trace for `ddp-multigpu` in case you do not have two GPUs.

If you need to use our pre-computed trace for `ddp-multigpu`, remove the `--overwrite-existing-results` argument.
```bash
python3 ae_fp.py --bench workloads
```

Or, if you have a machine with 2 GPUs, execute the below command, such that the original results will be re-computed.
```bash
python3 ae_fp.py --bench workloads --overwrite-existing-results
```

### What to Expect During Execution

The script is long running. It performs three tasks at same time. 
1. It collects trace for all the workloads.
2. It infers invariants for three setups in Section 5.4.
3. It checks inferred invariants on the validation workloads.

The experiments might fail if environment installation issues or disruption happens. When you run into problems, please refer to [⚠️ Notes & Troubleshooting](#️-notes--troubleshooting).

### ⚠️ Notes & Troubleshooting

The script will automatically detect any errors in any (1) trace collection, (2) inference tasks, (3) checking tasks. If you encounter any trace collection issues, please check for any missing environment dependencies.

If you encounter any issues on invariant inference tasks or invariant checking tasks, please try to rerun the experiment by adding `--overwrite-existing-results` or delete all `trace_*` folders except for `trace_ddp-multigpu`.

If you see persistent issues, it will likely be a environment issue or software bug. Please contact us for help.

### How to verify the results?

The `ae_fp.py` script generates a file called `fp.csv` under the current directory. Looking like this

```csv
setup,fp_rate
1-input,0.3105
4-input,0.1127
6-input,0.1066
```

These values correspond to the results reported in Section 5.4 of the paper.
You should verify that the false positive rates are similar (within a 5% margin of error) or lower.

## Eval: Transferability

⏳ Estimated Completion Time: TBD hour.
- Trace Collection: x hours
- Invariant Inference: x hours
- Invariant Checking: x hours

### 🎯 Goal

This evaluation measures the transferability of invariants inferred by TrainCheck. 

## Eval: Performance Overhead

⏳ Estimated Completion Time: 1.5 hour.

### 🎯 Goal

This evaluation measures the runtime overhead introduced by TrainCheck’s instrumentation compared to un-instrumented runs across a set of representative ML workloads, during the invariant checking stage. The results correspond to Section 5.5 of the paper.


### 📂 Resources & Scripts

- Automation Scripts: 
  - `eval_scripts/perf_benchmark/run_all.xsh`: run the experiments and collect data.
  - `eval_scripts/perf_benchmark/analysis.xsh`: analyze raw data and produce input for the plot script.
  - `eval_scripts/perf_benchmark/plot_e2e.py` and `eval_scripts/perf_benchmark/plot_micro.py`: plot the figures in Section 5.5.
  
- Workloads (You probably won't need to touch this):
    - Located in [overhead-e2e](../eval_scripts/perf_benchmark/overhead-e2e) and [overhead-micro](../eval_scripts/perf_benchmark/overhead-micro)
	- No pre-collected data is required—this evaluation runs end-to-end automatically and is pretty light weight

- Deployed 100 invariants:
    [eval_scripts/perf_benchmark/overhead-e2e/sampled_100_invariants.json](../eval_scripts/perf_benchmark/overhead-e2e/sampled_100_invariants.json)


### 🛠 How to Run

1. Navigate to the performance benchmark directory:
    ```bash
    cd eval_scripts/perf_benchmark/
    ```

2. Run the full benchmark suite using:
    ```bash
    xonsh eval_scripts/perf_benchmark/run_all.xsh
    ```
This script will:
- Execute each workload in three modes:
    - No instrumentation
	- TrainCheck selective instrumentation with 100 invariants deployed
	- Python settrace baseline (a lightweight instrumentation baseline)
- Measure per-iteration training time.
- Save raw results in a folder named: `perf_eval_res_<commit_hash>`

You should then execute the below commands that analyze the data and produce plots.
```bash
xonsh analysis.xsh --res_folder perf_eval_res_<commit_hash>

python3 plot_e2e.py -o perf_eval_res_<commit_hash>/macro.pdf -i perf_eval_res_<commit_hash>/overhead_e2e.csv -t <commit_hash>

python3 plot_micro.py -o perf_eval_res_<commit_hash>/micro.pdf -i perf_eval_res_<commit_hash>/wrapper_overhead_micro.csv -t <commit_hash>
```

### Expected Output
Key files in `perf_eval_res_<commit_hash>`:
- `overhead_e2e.csv` and `marco.pdf` data and plot for benchmarks presented in Section 5.5.
- `wrapper_overhead_micro.csv` and `micro.pdf`: data and plot for the pure wrapper overhead on individual APIs.

### ✅ How to Verify
	•	Check that the overhead percentages in overhead_results.csv are consistent with those reported in Section 5.5.
	•	Variations (within ±15% TODO confirm) are expected due to runtime and hardware differences.


### ⚠️ Notes & Troubleshooting
1. **Do Not Run Other GPU Tasks in Parallel**

    For stable performance measurements, the evaluation scripts will periodically terminate all CUDA processes to ensure a clean environment. 
    Please avoid running any other GPU workloads during this evaluation.

2. **Handling Failed Workloads**

    If an end-to-end workload fails:
    - Navigate to the corresponding workload folder.
    - Manually rerun it using:
    ```bash
    traincheck-collect --use-config --config md-config-var.yml -i ../sampled_100_invariants.json
    ```
	- If the issue does not reproduce consistently, simply delete the result folder and rerun the full benchmark.
	- If the failure is consistent, please contact us for support.
