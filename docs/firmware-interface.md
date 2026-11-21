# Firmware interface

The contract between this board and the emulator firmware in `../gbc-32`.
For each function the board provides, this lists what the firmware needs
from the MCU. Update it whenever a hardware choice changes a pin or
peripheral assignment, because the firmware's platform layer is written
against it.

> Status: seeded from the firmware's needs before any hardware decisions.
> Requirement-level only; no pins assigned yet.

| Function | Firmware needs | Candidate STM32H7 peripheral | Notes |
|---|---|---|---|
| Display | Push 160×144 (scaled) frames at 59.7 Hz without stalling emulation | FMC (8/16-bit parallel, as on the dev shield), LTDC (RGB), or SPI + DMA | The dev board uses FMC. Choosing LTDC or SPI changes the platform layer. |
| ROM storage | Read ROM banks (up to 8 MiB per game) fast enough for bank switching | SDMMC (eMMC) | The firmware will likely copy the active game into RAM or cache it; see the RAM line |
| Saves | Write cartridge RAM (up to 128 KiB) + RTC state | SDMMC (eMMC) | Power-loss safety (flush on low battery?) TBD |
| Audio | Stream stereo samples continuously | SAI or I2S + DMA → DAC/amp | Sample rate decided in firmware M6 |
| Input | ~8–12 buttons, read once per frame | GPIO (with pull-ups) | Debouncing in firmware |
| RAM | Emulator state (~200 KiB) + framebuffer(s) + possibly the full ROM (≤ 8 MiB) | Internal SRAM; external SDRAM/PSRAM via FMC/OCTOSPI if ROMs are loaded into RAM | Key open question: execute ROM from eMMC cache, or load it into external RAM? |
| Timekeeping | RTC for MBC3 games (Pokémon G/S/C) across power-off | MCU RTC + backup domain (VBAT) | Coin cell or supercap for VBAT? |
| Debug | Flashing and debugging | SWD (+ SWO) | Header or pads on both board revisions |
| Power monitoring | Battery level for warnings and safe save-flush | ADC or fuel-gauge IC (I2C) | |
