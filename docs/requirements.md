# Requirements

Owned by the human. This seed version contains only what has been decided
so far; everything marked **TBD** is open.

## Decided

- **Purpose:** standalone handheld that runs the Game Boy Color 32 emulator
  (`../gbc-32`) at full speed, with video, audio and input.
- **Storage:** eMMC for game ROMs and save files.
- **Audio:** dedicated audio DAC/amplifier driving onboard speaker(s).
- **Build plan:** a prototype board (Stage 3) and then a refined board
  (Stage 4) that fixes what prototype bring-up uncovers.
- **MCU:** STM32H735IGK6 (UFBGA176+25, 550 MHz, on-chip SMPS), used on both
  boards. See `decisions/0002-mcu-stm32h735igk6.md`. The firmware is still
  developed on the STM32H753ZI dev board for Stage 2.

## TBD

- **Display:** size, resolution, interface (parallel/FMC, SPI, RGB/LTDC,
  MIPI) and controller. See `firmware-interface.md` for the firmware side
  of this choice.
- **Input:** button set (D-pad, A, B, Start, Select, plus extras such as
  menu or volume), switch type.
- **Power:** battery chemistry and capacity, target play time, charging
  (USB-C?), power switch, fuel gauge.
- **Audio extras:** headphone jack? Volume control (analog or digital)?
- **Connectivity:** USB for charging only, or also data (ROM transfer,
  firmware update)? Debug/programming header (SWD).
- **Enclosure / form factor:** dimensions, layout (vertical vs. horizontal),
  how the board is mounted.
- **Manufacturing:** fab/assembly house, hand vs. machine assembly, budget
  per unit.
