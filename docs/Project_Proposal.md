# AI Hardware Project Proposal

**Course:** ECE 4332 / ECE 6332 — AI Hardware Design and Implementation
**Submitted:** November 5, 2026

---

## 1. Project Title

**Team name:** Team21

**Team members:**
- Matthew Batmunkh
- Nabil Abdelbaset

**Title:** *Trade-offs in a Low-Latency FPGA Neural Network Accelerator*

---

## 2. Platform Selection

**Category:** Edge-AI — FPGA-based edge inference

**Justification:**

- **Custom hardware without fabrication.** An FPGA lets us design, verify, and implement a dedicated neural network accelerator without the cost of building a chip. The design is a spatial dataflow circuit, not software running on a processor.
- **Real, device-accurate results at no cost.** The free edition of AMD Vivado and Vitis HLS reports resource utilization, timing, and power for a real device, the AMD Zynq-7020 (`xc7z020clg400-1`, used on the PYNQ-Z2 board). No physical board is required.
- **Established open-source flow.** We use hls4ml, an open-source tool from the particle-physics community (CERN, Fermilab) that converts trained neural networks into FPGA firmware. It is used in real low-latency trigger systems, and published results exist for direct comparison.
- **Reproducible.** Every tool is free, so anyone can rerun our results from the GitHub repository.

---

## 3. Problem Definition

Many applications need AI decisions in microseconds at very low power: particle-collision triggers at the Large Hadron Collider, radar, autonomous vehicles, and industrial control. CPUs and GPUs cannot meet these latency targets for single inputs, because of instruction overhead, memory access, and batching requirements.

Custom FPGA accelerators can meet them, but the designer must balance three competing factors:

1. **Numerical precision:** fewer bits per weight shrink and speed up the hardware but can reduce accuracy.
2. **Parallelism:** more parallel multipliers lower latency but consume more of the chip.
3. **Resources:** a small edge FPGA has a fixed budget of logic (LUTs), multipliers (DSPs), and memory (BRAM).

This project quantifies these trade-offs through a design-space exploration of an FPGA accelerator for a neural network, combining hardware/software codesign (quantization-aware training) with hardware design choices (bit-width and parallelism). It addresses the core AI hardware concerns of **efficiency, latency, and scalability**, and compares the result against CPU and GPU execution of the same model.

**Target network:** a 4-layer multilayer perceptron (MLP) for the hls4ml LHC jet-tagging benchmark — 16 input features → 64 → 32 → 32 → 5 output classes, ReLU activations, softmax output, about 4,400 parameters.

---

## 4. Technical Objectives

These objectives form our design specification. Every result will be evaluated against them.

| # | Objective | Quantitative target |
|---|---|---|
| 1 | **Accuracy at low precision** | Within 2 percentage points of the floating-point model's test accuracy using ≤ 8-bit weights and activations |
| 2 | **Low latency** | ≤ 200 ns per inference at a 200 MHz clock for the fastest configuration that fits the device |
| 3 | **Fit on an edge FPGA** | At least one configuration using ≤ 80% of the Zynq-7020's LUTs and ≤ 100% of its DSPs, meeting timing at 200 MHz |
| 4 | **Resource savings from quantization** | ≥ 50% reduction in LUT + DSP usage from 16-bit to 6-bit at equal parallelism, with ≤ 2 points accuracy loss |
| 5 | **Design-space exploration and comparison** | ≥ 16 configurations (4 bit-widths × ≥ 4 reuse factors) on an accuracy-vs-resources Pareto plot; energy per inference compared with CPU and GPU baselines |

---

## 5. Methodology

We follow a **Specify → Design → Verify → Implement → Validate** flow.

### Hardware setup
- **Target device:** AMD Zynq-7020 (`xc7z020clg400-1`), synthesis and implementation only; no physical board required.
- **Clock target:** 200 MHz (5 ns period).

### Software tools
- **Model training:** Python, TensorFlow/Keras, QKeras (quantization-aware training)
- **Hardware generation:** hls4ml (Vitis backend)
- **Synthesis and implementation:** AMD Vitis HLS and Vivado (free edition)
- **Hand-written RTL verification:** Icarus Verilog and GTKWave (or the Vivado simulator)
- **Baselines:** laptop CPU and Google Colab GPU
- **Version control:** GitHub

### Model design
1. Train the baseline floating-point MLP on the jet-tagging dataset.
2. Retrain with QKeras at 8, 6, and 4 bits using quantization-aware training.
3. Accuracy recovery at low precision: extended quantization-aware training, magnitude pruning with retraining, and input-noise data augmentation.

### Hardware design
- **Top-down (automated):** convert each model with hls4ml, sweeping fixed-point precision (`ap_fixed` 16, 8, 6, 4 bits) and the **reuse factor** (1, 2, 4, 8, 16, 32), which sets how many times each hardware multiplier is shared.
- **Bottom-up (hand-designed):** write a parameterized multiply-accumulate (MAC) unit in Verilog and use it to build the final layer (32 → 5) as a time-multiplexed engine. Compare its resources and latency with the hls4ml-generated version of the same layer.

### Performance metrics
| Metric | Source |
|---|---|
| Accuracy | Python model and C-simulation of the hardware |
| Latency (cycles, ns) and initiation interval | Vitis HLS reports |
| Throughput (inferences/s) | Initiation interval × clock frequency |
| LUT, FF, DSP, BRAM utilization | Vitis HLS and Vivado implementation reports |
| Timing (achieved clock) | Vivado implementation timing report |
| Power (W) and energy per inference (J) | Vivado power report; energy = power × latency |

### Validation strategy
- **Functional verification:** hls4ml C-simulation and C/RTL co-simulation must predict the same class as the Python model on ≥ 99% of test samples.
- **RTL verification:** a Verilog testbench checks the hand-designed layer against Python reference outputs.
- **Implementation checks:** place and route for selected designs to confirm timing closure and accurate resource counts.
- **Power refinement:** for selected designs, switching activity from simulation (SAIF) is used to refine Vivado's default power estimate.
- **Sanity check:** baseline accuracy, latency, and resources are compared with published hls4ml results for the same benchmark.
- **Architecture comparison:** the same model is timed on a CPU and a GPU at batch sizes 1 and 1024, and placed alongside published TPU/edge-accelerator figures.

---

## 6. Expected Deliverables

- **Verified accelerator designs:** hls4ml-generated FPGA designs for all sweep configurations, verified against the Python model
- **Hand-designed Verilog layer:** MAC unit and layer engine with testbench and synthesis results
- **Results dataset:** `results.csv` with every configuration and its metrics
- **Reproducible scripts:** training, conversion, sweep, and plotting scripts that regenerate all results
- **Analysis:** Pareto plots (accuracy vs. resources), latency-vs-parallelism plots, energy-per-inference comparison against CPU and GPU
- **Demo:** live walkthrough of the flow — train, convert, verify, and report results for one configuration
- **GitHub repository:** organized per the class template, tagged `v1.0-final`
- **Presentation slides:** midterm and final
- **Final report:** IEEE format

---

## 7. Team Responsibilities



---

## 8. Timeline and Milestones

| Week | Date | Milestone | Deliverable |
|---|---|---|---|
| 2 | Nov 5 | Proposal | PDF + GitHub submission |
| 3 | Nov 12 | Baseline | Trained model; first 16-bit design verified and synthesized |
| 4 | Nov 19 | Midterm presentation | Slides; precision-sweep results (16, 8, 6, 4 bits) |
| 6 | Dec 3 | Integration & testing | Full precision × parallelism sweep, verified Verilog layer, CPU/GPU baselines |
| 7 | Dec 7 | Analysis complete | Power and energy results, accuracy-recovery results; experiments frozen |
| — | Dec 12 | Final presentation | Report, demo, GitHub archive (`v1.0-final`) |

---

## 9. Resources Required

**Hardware:** None beyond personal laptops or ECE lab computers. No FPGA board is required; all results come from synthesis and implementation.

**Software (all free):**
- AMD Vivado and Vitis HLS (free edition), at the version supported by hls4ml
- Python 3, TensorFlow/Keras, QKeras, hls4ml
- Icarus Verilog, GTKWave
- Google Colab (free tier) for GPU baselines

**Datasets:** hls4ml LHC jet-tagging dataset (`hls4ml_lhc_jets_hlf`, publicly available through OpenML)

**Compute access:** A Windows or Linux machine with Vivado installed (ECE lab machines preferred), about 16 GB of RAM, and sufficient disk space for the Vivado installation.

---

## 10. References

1. J. Duarte et al., "Fast inference of deep neural networks in FPGAs for particle physics," *Journal of Instrumentation*, vol. 13, P07027, 2018.
2. C. N. Coelho Jr. et al., "Automatic heterogeneous quantization of deep neural networks for low-latency inference on the edge for particle detectors," *Nature Machine Intelligence*, vol. 3, pp. 675–686, 2021.
3. T. Aarrestad et al., "Fast convolutional neural networks on FPGAs with hls4ml," *Machine Learning: Science and Technology*, vol. 2, 045015, 2021.
4. V. Sze, Y.-H. Chen, T.-J. Yang, and J. S. Emer, "Efficient Processing of Deep Neural Networks: A Tutorial and Survey," *Proceedings of the IEEE*, vol. 105, no. 12, 2017.
5. N. P. Jouppi et al., "In-Datacenter Performance Analysis of a Tensor Processing Unit," *Proc. ISCA*, 2017.
6. Y. Umuroglu et al., "FINN: A Framework for Fast, Scalable Binarized Neural Network Inference," *Proc. ACM/SIGDA FPGA*, 2017.
7. hls4ml — https://github.com/fastmachinelearning/hls4ml
8. hls4ml tutorial — https://github.com/fastmachinelearning/hls4ml-tutorial
9. QKeras — https://github.com/google/qkeras
10. AMD, *Vitis High-Level Synthesis User Guide* (UG1399)
11. AMD, *Vivado Design Suite User Guide: Synthesis* (UG901)
12. AMD, *Vivado Design Suite User Guide: Power Analysis and Optimization* (UG907)
