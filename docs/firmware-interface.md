# Firmware interface

The contract between this board and the emulator firmware in `../gbc-32`.
For each function the board provides, this lists what the firmware needs
from the MCU. Update it whenever a hardware choice changes a pin or
peripheral assignment, because the firmware's platform layer is written
against it.

> Status: seeded from the firmware's needs before any hardware decisions.
> Requirement-level only; no pins assigned yet. Decided so far: ADR 0001
> (ROM in external FMC SDRAM), and the MCU: STM32H735IGK6 (recorded in
> `../gbc-32/README.md` Stage 3; its hardware ADR is not written yet).
> Firmware constraint from that MCU: unaligned reads from the FMC region
> are unsupported (ES0491 §2.7.3), so the ROM in SDRAM is read byte by byte.

| Function | Firmware needs | Candidate STM32H7 peripheral | Notes |
|---|---|---|---|
| Display | Push 160×144 (scaled) frames at 59.7 Hz without stalling emulation | FMC (8/16-bit parallel, as on the dev shield), LTDC (RGB), or SPI + DMA | The dev board uses FMC. Choosing LTDC or SPI changes the platform layer. |
| ROM storage | Read ROM banks (up to 8 MiB per game) fast enough for bank switching | SDMMC (eMMC) | Library on eMMC, uploaded over USB. The active game is copied into SDRAM at load (ADR 0001); see the RAM line |
| Saves | Write cartridge RAM (up to 128 KiB) + RTC state | SDMMC (eMMC) | Power-loss safety (flush on low battery?) TBD |
| Audio | Stream stereo samples continuously | SAI or I2S + DMA → DAC/amp | Sample rate decided in firmware M6 |
| Input | ~8–12 buttons, read once per frame | GPIO (with pull-ups) | Debouncing in firmware |
| RAM | Emulator state (~200 KiB) + framebuffer(s) + the full active ROM (≤ 8 MiB) | Internal SRAM for emulator state and cartridge RAM; external 16-bit SDR SDRAM on FMC (≥ 8 MiB, aiming for 32 MiB) for the ROM | Decided in ADR 0001. Core keeps `rom: &[u8]` unchanged. Platform layer: FMC SDRAM init, eMMC → SDRAM loader, MPU region marking SDRAM normal/cacheable (default `0xC0000000` mapping is device memory). If the display also uses FMC, it shares the data bus with the SDRAM. |
| Timekeeping | RTC for MBC3 games (Pokémon G/S/C) across power-off | MCU RTC + backup domain (VBAT) | Coin cell or supercap for VBAT? |
| Debug | Flashing and debugging | SWD (+ SWO) | Header or pads on both board revisions |
| Power monitoring | Battery level for warnings and safe save-flush | ADC or fuel-gauge IC (I2C) | |
