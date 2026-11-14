[README_fpga_dma.md](https://github.com/user-attachments/files/34834557/README_fpga_dma.md)
# fpga-uart-dma

A DMA controller you can drive from a PC over UART, built for an iCE40 FPGA (iCEBreaker, UP5K).

    PC --UART--> uart_bridge --APB--> dma_ctrl --> 4 KB block RAM

- `rtl/dma_ctrl.sv`: DMA controller with APB registers, IRQ, and error checks
- `rtl/uart.sv`: 8N1 UART receiver and transmitter
- `rtl/uart_bridge.sv`: frame protocol to read/write DMA registers and poke/peek RAM
- `rtl/fpga_top.sv`: top level (power-on reset, single-port block RAM, status LEDs)
- `tb/tb_fpga.sv`: self-checking testbench where a simulated PC talks to the design over UART
- `fpga/icebreaker.pcf`: pin constraints for the iCEBreaker board

## UART frames (big-endian)
| Frame | Meaning | Reply |
|-------|---------|-------|
| `W` addr16 data32 | write DMA register | 0x06 |
| `R` addr16 | read DMA register | 4 bytes |
| `P` addr16 data32 | write RAM word | 0x06 |
| `Q` addr16 | read RAM word | 4 bytes |

## Run
    pip install verilator yowasp-yosys yowasp-nextpnr-ice40
    make sim       # Verilator simulation, 275 checks
    make pnr       # synthesis, place and route, bitstream
    make report    # utilization and timing

## Results (open-source flow: Yosys + nextpnr-ice40, iCE40 UP5K SG48)
| Item | Result |
|------|--------|
| Logic cells | 1021 of 5280 (19%) |
| Block RAM | 8 of 30 (26%) |
| Max clock | 27.9 MHz (target 12 MHz, timing met) |
| Bitstream | `build_fpga/fpga_top.bin` (generated) |

## Limits
Simulated and taken through synthesis, place and route, and bitstream generation. **Not yet run on
a real board**, and the pin numbers in the .pcf were not checked on hardware. No PC-side script is
included yet.
