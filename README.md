---
durum: taslak
---

## FIR IP

Parameterized FIR DSP IP Generator using **Scala** and **Chisel**.
> **Work in Progress**

---

## Project Status

Study phase is completed. The project is currently design
* [x] Phase 0 — Investigation & System Definition
* [x] Phase 1 — FIR Fundamentals & Filter Specifications
* [x] Phase 2 — end of fir-study
* [ ] Phase 3 — Reference Model
* [ ] Phase 4 — Architecture Decision
* [ ] Phase 5 — Chisel Implementation
* [ ] Phase 6 — Verification
* [ ] Phase 7 — Synthesis & Hardware Evaluation
* [ ] Phase 8 — IP Packaging & Documentation

## Objective

Develop a reusable FIR DSP IP generator whose hardware implementation can be configured according to system requirements such as:

* Number of taps
* Input data width
* Coefficient width
* Arithmetic precision
* Filter architecture
* Pipeline configuration
* Throughput requirements

The project is not limited to implementing the FIR equation itself. The main objective is to investigate the transformation:

```text
DSP Requirements
      ↓
Filter Specification
      ↓
Filter Design
      ↓
Fixed-Point Representation
      ↓
Hardware Architecture
      ↓
Chisel Generator
      ↓
RTL
      ↓
Verification
      ↓
Synthesis / FPGA Evaluation
```

## Technology and Versions

### Hardware Description / Generation

* Scala
* Chisel

### DSP / Reference Modeling

* Python
      * NumPy
      * SciPy

### Verification & Evaluation

* Scala / Chisel test infrastructure

## Repository Structure


## License
