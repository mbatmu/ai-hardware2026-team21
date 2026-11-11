# AI Hardware Project Proposal

**Course:** ECE 4332 / ECE 6332 — AI Hardware Design and Implementation
**Submitted:** November 5, 2026

---

## 1. Project Title

**Team name:** Team21

**Team members:**
- Matthew Batmunkh
- Nabil Abdelbaset
- Chris Dougherty

**Title:** *Trade-offs in a Low-Latency FPGA Accelerator*

---

## 2. Platform Selection

**Category:** Edge-AI — FPGA-based edge inference

**Justification:** An FPGA lets us build a custom AI circuit without making a chip. The free AMD Vivado tools report speed, size, and power for a real chip (Zynq-7020), so no board is needed. We also run the model on an Arduino Tiny Machine Learning Kit for comparison.

---

## 3. Problem Definition

Some applications need AI answers in under a microsecond at low power, which CPUs and GPUs can't deliver for single inputs. FPGAs can, but designers must trade off precision, parallelism, and chip space. We measure these trade-offs on a small neural network from the hls4ml particle-physics benchmark (16 inputs, 5 outputs, about 4,400 weights).

---

## 4. Technical Objectives

| # | Objective | Quantitative target |
|---|---|---|
| 1 | **Accuracy** | Within 2 points of the full-precision model using 8 bits or fewer |
| 2 | **Latency** | 200 ns or less per inference at 200 MHz |
| 3 | **Fit on the chip** | At least one design that fits the Zynq-7020 at 200 MHz |
| 4 | **Resource savings** | At least 50% fewer resources going from 16 to 6 bits |
| 5 | **Comparison** | 24 designs compared, plus energy per inference vs. CPU, GPU, and Arduino |

---

## 5. Methodology

- **Hardware:** Zynq-7020 FPGA (simulated in Vivado), Arduino Tiny ML Kit, laptop CPU, Colab GPU
- **Software:** Python, TensorFlow, QKeras, hls4ml, Vivado, Icarus Verilog
- **Model design:** train the network, then retrain it at 8, 6, and 4 bits
- **Hardware design:** build a circuit for each bit count and parallelism setting with hls4ml, plus one layer written by hand in Verilog
- **Metrics:** accuracy, latency, resource use, power, energy per inference
- **Validation:** the circuit must match the Python model's predictions on at least 99% of test samples

---

## 6. Expected Deliverables

- FPGA designs and a hand-written Verilog layer
- Results spreadsheet and graphs
- Demo
- GitHub repository
- Presentation slides
- Final report

---

## 7. Team Responsibilities

| Name | Role | Responsibilities |
|---|---|---|
| Nabil Abdelbaset | Software | Model training, CPU/GPU tests, GitHub, report |
| Matthew Batmunkh | Hardware | hls4ml, Vivado, Verilog layer |
| Chris Dougherty | Testing and Results | Verification, Arduino tests, power, graphs |

---

## 8. Timeline and Milestones

| Week | Milestone | Deliverable |
|---|---|---|
| 2 | Proposal | PDF + GitHub submission |
| 4 | Midterm presentation | Slides, preliminary results |
| 6 | Integration & testing | Full results, working prototype |
| Dec. 12 | Final presentation | Report, demo, GitHub archive |

---

## 9. Resources Required

- Arduino Tiny Machine Learning Kit (class board library)
- USB power meter
- Computer with AMD Vivado (ECE lab)
- Google Colab
- hls4ml jet-tagging dataset (OpenML)

---

## 10. References

1. J. Duarte et al., "Fast inference of deep neural networks in FPGAs for particle physics," *Journal of Instrumentation*, 2018.
2. C. N. Coelho Jr. et al., "Automatic heterogeneous quantization of deep neural networks for low-latency inference on the edge for particle detectors," *Nature Machine Intelligence*, 2021.
3. V. Sze et al., "Efficient Processing of Deep Neural Networks: A Tutorial and Survey," *Proceedings of the IEEE*, 2017.
4. hls4ml — https://github.com/fastmachinelearning/hls4ml
5. AMD, *Vitis High-Level Synthesis User Guide* (UG1399)
