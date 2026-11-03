# Simulation-Based Evaluation of Memory Interfaces for an FPGA Accelerator on Xilinx Zynq-7000 SoC

B.Tech ECE, FPGA Based Systems Design, VIT Chennai. Project faculty: Dr. Hariharan I.

| Team member | Register number |
|---|---|
| U. Pherarasan | 23BEC1247 |
| Divakar V | 23BEC1273 |
| Vinith Kumar B | 23BEC1275 |
| Nizsaanth Nobel A | 23BEC1285 |

## Overview
This project compares three ways of holding data for an FPGA accelerator on a Digilent ZedBoard (Zynq-7000 XC7Z020): **BRAM**, **LUTRAM** and **external DDR** reached through the Zynq Processing System. The application is a 4 x 4 matrix multiplication written with Vitis HLS. An ARM Cortex-A9 software version is used as the baseline.

## Architecture
- Zynq Processing System (ARM Cortex-A9, DDR controller)
- HLS memory IP `m_axi_memory_0` (control at `0x40000000`)
- HLS matrix accelerator `matrix_mult_accel_0` (control at `0x40010000`)
- AXI4-Lite control path from the PS to both IPs
- AXI master data path from both IPs, through the AXI memory interconnect, to PS `S_AXI_HP0` and DDR (`0x00000000` to `0x1FFFFFFF`)
- No AXI DMA is used

## Tools
Vitis HLS 2026.1, Vivado 2026.1, Vitis Unified IDE 2026.1, serial terminal (PuTTY).

## Results

### Resource and timing (as reported by the team)
| Metric | BRAM | LUTRAM | DDR |
|---|---|---|---|
| LUTs | 67 | 195 | 285 * |
| Flip-flops | 35 | 67 | 112 * |
| RAMB18 | 1 | 0 | 2 * |
| LUTs used as memory | 0 | 128 | 0 * |
| WNS | +5.994 ns | +6.284 ns | +5.421 ns * |
| Reported latency | 1 cycle | 1 cycle | 12 cycles * |

\* DDR figures were supplied for this README and must be matched to a saved report in `reports/` before the repository is made public. Latency values are reported memory-implementation figures, not the full application execution time.

### ARM Cortex-A9 software baseline (measured)
| Item | Value |
|---|---|
| Matrix size | 4 x 4 |
| Repetitions | 10,000 |
| CPU clock | 666,666,687 Hz |
| Global Timer | 333,333,343 ticks/s |
| Elapsed ticks | 2,015,209 |
| Total time | 6,045,626 ns |
| Average per multiplication | about 604.56 ns |
| Checksum | 360 |
| Correctness | PASS |

Expected output matrix:
```
 1   4   9  16
 5  12  21  32
 9  20  33  48
13  28  45  64
```

## Status
| Item | Status |
|---|---|
| HLS accelerator, AXI integration, Vivado implementation, bitstream, Vitis platform | Completed |
| BRAM, LUTRAM and DDR evaluation | Evaluated (see table above) |
| ARM software benchmark | Completed, PASS |
| FPGA accelerator execution on the board | **Not verified.** Execution stopped when accessing the accelerator `AP_CTRL` register. |
| FPGA execution time and speedup over ARM | **Not available** |

No speedup claim is made, because no valid accelerator execution time has been obtained.

## Repository layout
```
hls/          HLS sources (bram_memory.cpp, lutram_memory.cpp, matrix_mult_accel.cpp)
vitis/        ARM and accelerator application sources (src folders only)
reports/      Utilization, timing and HLS synthesis reports used for the tables
screenshots/  Vivado block design, PuTTY outputs
docs/         Review explanation and notes
```
Build outputs, ELF files, bitstreams and Vivado/Vitis generated folders are not committed (see `.gitignore`). Attach the `.xsa`, `.bit` and `.elf` files to a GitHub Release if they need to be shared.

## Known issue
During hardware testing, the program stopped at the access to the accelerator control register `AP_CTRL` (offset `0x00`, base `0x40010000`). The cause has not been identified yet.

## Next steps
1. Find the cause of the `AP_CTRL` access failure and obtain a successful accelerator run.
2. Capture the PuTTY output and measure accelerator time per multiplication.
3. Compute speedup only after both runs show correct output.
