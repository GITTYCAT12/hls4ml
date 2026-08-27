# HLS → RTL → Synthesis: VLSI Workflow

This note documents the hardware-design flow used when an ML model is converted into synthesizable hardware with hls4ml. The focus is on the VLSI aspects of the flow: RTL generation, synthesis, timing, and resource trade-offs.

## 1. Design Flow

```text
ML Model
   │
   ▼
Model Configuration
   │
   ▼
hls4ml Conversion
   │
   ▼
HLS C/C++ Representation
   │
   ▼
HLS Scheduling / Optimization
   │
   ▼
Generated RTL (Verilog/VHDL)
   │
   ▼
RTL Simulation / Verification
   │
   ▼
Logic Synthesis
   │
   ├── LUT / FF utilization
   ├── DSP utilization
   ├── BRAM utilization
   ├── Critical-path delay
   └── Estimated Fmax
   │
   ▼
Hardware Implementation
```

## 2. RTL Generation

hls4ml translates a supported neural-network description into an HLS implementation that can be synthesized to FPGA-oriented RTL. From a VLSI perspective, the important point is that the software model becomes a hardware datapath rather than executing as software on a CPU.

The generated design can contain arithmetic operators, registers, memory structures, control logic, and interfaces. HLS directives such as pipelining, unrolling, array partitioning, and reuse factors influence the resulting microarchitecture.

## 3. RTL Verification

Before synthesis, the generated RTL should be simulated against known input/output vectors. Verification should check functional equivalence between the reference model and the generated hardware representation.

Recommended checks:

- Reset and clock behavior
- Input/output protocol
- Numerical accuracy after quantization
- Correct latency
- Deterministic output for identical inputs
- Corner-case behavior

## 4. Logic Synthesis

The RTL is passed to an FPGA synthesis tool such as Vivado. Synthesis maps RTL constructs into device resources and produces timing and utilization reports.

Key metrics to record are:

| Metric | VLSI meaning |
|---|---|
| LUT | Combinational logic utilization |
| FF | Sequential storage utilization |
| DSP | Dedicated arithmetic resources |
| BRAM | On-chip memory utilization |
| Critical path | Longest timing path |
| Fmax | Maximum estimated operating frequency |
| Latency | Cycles required for an inference |
| II | Cycles between successive pipeline inputs |

## 5. PPA Trade-off

Hardware optimization is a trade-off rather than a single-metric problem.

For example, increasing parallelism can improve throughput but increase LUT/DSP usage. Reducing numerical precision can reduce hardware cost, but excessive quantization can reduce model accuracy.

A useful engineering comparison is:

```text
                 Higher Throughput
                        ▲
                        │
             Pipeline / Unroll
                        │
                        │
Lower Area ◄────────────┼────────────► Higher Area
                        │
                        │
              Resource Sharing
                        │
                        ▼
                  Lower Throughput
```

## 6. VLSI Interpretation

The important engineering chain is:

**Algorithm → Architecture → RTL → Synthesis → PPA analysis**

This makes an hls4ml-based accelerator useful as a hardware-design study: architectural decisions made at the HLS level can be observed later as changes in RTL structure, resource utilization, latency, and timing.

## 7. Reproducibility

Any reported synthesis result should record:

- Model architecture
- Numerical precision
- HLS configuration
- Target FPGA/device
- Clock constraint
- HLS directives
- Tool/version information
- LUT/FF/DSP/BRAM usage
- Latency and initiation interval
- Timing result

Results should be generated from actual synthesis runs rather than estimated manually.

## 8. Next Experiments

The next hardware-focused experiments for this repository should compare:

1. Baseline HLS architecture
2. Pipeline optimization
3. Loop unrolling / parallelism
4. Array partitioning
5. Fixed-point precision
6. Resource utilization versus performance

These experiments will provide measurable evidence of FPGA/VLSI design decisions instead of treating hls4ml only as a model-conversion tool.
