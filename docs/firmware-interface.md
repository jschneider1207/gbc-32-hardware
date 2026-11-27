# Firmware interface

The contract between this board and the emulator firmware in `../gbc-32`.
For each function the board provides, this lists what the firmware needs
from the MCU. Update it whenever a hardware choice changes a pin or
peripheral assignment, because the firmware's platform layer is written
against it.

> Status: seeded from the firmware's needs before any hardware decisions.
> Requirement-level only; no pins assigned yet. Decided so far: ADR 0002
> (MCU: STM32H735IGK6, also recorded in `../gbc-32/README.md` Stage 3) and
> ADR 0003 (ROM in HyperRAM on OCTOSPI; it supersedes ADR 0001's FMC SDRAM).
> The FMC unaligned-read erratum (ES0491 §2.7.3) no longer applies to the
> ROM: memory-mapped HyperBus reads are allowed at any address and length
> (ES0491 §2.8.4 Table 5). OCTOSPI rules that do apply: WRAPSIZE = 0
> (§2.8.2, §2.8.3, §2.8.15) and HyperBus latency ≥ 6 (§2.8.13).
>
> Updated 2026-11-24 from the firmware side (measured on the
> `thumbv7em-none-eabihf` build of `gbc-core`, `../gbc-32/docs/perf.md`).

| Function | Firmware needs | Candidate STM32H7 peripheral | Notes |
|---|---|---|---|
| Display | Push 160×144 (scaled) frames at 59.7 Hz without stalling emulation | FMC (8/16-bit parallel, as on the dev shield), LTDC (RGB), or SPI + DMA | The dev board uses FMC. Choosing LTDC or SPI changes the platform layer. |
| ROM storage | Read ROM banks (up to 8 MiB per game) fast enough for bank switching | SDMMC (eMMC) | Library on eMMC, uploaded over USB. The active game is copied into HyperRAM at load (ADR 0003); see the RAM line |
| Saves | Write cartridge RAM (up to 128 KiB) + RTC state | SDMMC (eMMC) | Power-loss safety (flush on low battery?) TBD |
| Audio | Stream stereo samples continuously | SAI or I2S + DMA → DAC/amp | 32,768 Hz stereo, 16-bit samples (`../gbc-32` ADR 0021). After a DC-blocking filter the signal can reach ±61,440 (17 bits signed), so the filter must not run in `i16` |
| Input | ~8–12 buttons, read once per frame | GPIO (with pull-ups) | Debouncing in firmware |
| RAM | Emulator state (measured 51.8 KiB: `Memory` 50,968 B + `Emulator` 848 B, plus 2,304 B of tables) + framebuffer(s) + cartridge RAM (≤ 128 KiB) + the full active ROM (≤ 8 MiB) | DTCM for emulator state and stack; AXI SRAM for frame buffers (2 × 46,080 B), audio buffers and cartridge RAM; ITCM for all of `gbc-core`'s code (97 KB); external 3.0 V HyperRAM on OCTOSPI (≥ 8 MiB) for the ROM | Decided in ADR 0003. Core keeps `rom: &[u8]` unchanged. Platform layer: OCTOSPI HyperBus init, eMMC → HyperRAM loader (memory-mapped writes, RM0468 §25.2), MPU attributes for the mapped window checked (normal, cacheable). **Option byte:** `TCM_AXI_SHARED` = 01 (RM0468 §2.4 Table 8; `FLASH_OPTSR2_PRG`, §4.9.25): 128 KiB ITCM + 256 KiB AXI SRAM. At 10 (192 KiB ITCM) AXI SRAM is 192 KiB, 34,816 B short of the 231,424 B listed above. ITCM/DTCM are reachable only by the CPU and MDMA (DS13312 §3.3.2), so DMA buffers go in AXI SRAM. A full-panel LTDC frame buffer of 153,600 B does not fit in AXI SRAM next to 128 KiB of cartridge RAM (284,672 B > 262,144 B). |
| Timekeeping | RTC for MBC3 games (Pokémon G/S/C) across power-off | MCU RTC + backup domain (VBAT) | Coin cell or supercap for VBAT? |
| Debug | Flashing and debugging | SWD (+ SWO) | Header or pads on both board revisions |
| Power monitoring | Battery level for warnings and safe save-flush | ADC or fuel-gauge IC (I2C) | |
