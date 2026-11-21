# Requirements

Owned by the human. This seed version contains only what has been decided
so far; everything marked **TBD** is open.

## Decided

- **Purpose:** standalone handheld that runs the Game Boy Color 32 emulator
  (`../gbc-32`) at full speed, with video, audio and input.
- **Storage:** eMMC for game ROMs and save files. The active ROM is copied
  into external SDRAM to run (`decisions/0001-external-sdram-for-rom.md`).
- **Audio:** dedicated audio DAC/amplifier driving onboard speaker(s).
- **Build plan:** a prototype board (Stage 3) and then a refined board
  (Stage 4) that fixes what prototype bring-up uncovers.
- **MCU:** STM32H735IGK6 (UFBGA176+25, 550 MHz, on-chip SMPS), used on both
  boards. See `decisions/0002-mcu-stm32h735igk6.md`. The firmware is still
  developed on the STM32H753ZI dev board for Stage 2.
- **Connectivity:** USB carries data, so ROMs are uploaded to the eMMC over
  USB, not just charging.
- **Enclosure:** custom 3D-printed. The Stage 3 prototype doesn't have to
  fit it: it can be a large flat bring-up board. The Stage 4 refined board
  is the one that fits the enclosure.
- **Manufacturing:** JLCPCB fabrication and PCBA (machine assembly, not
  hand-soldered), so BGA, extended and globally sourced parts are all
  acceptable. 6-layer board. Quantity: one finished device. JLC's minimum
  assembly batch may leave spare boards.

## TBD

- **Display:** size, resolution, interface (parallel/FMC, SPI, RGB/LTDC,
  MIPI) and controller. See `firmware-interface.md` for the firmware side
  of this choice.
- **Input:** button set (D-pad, A, B, Start, Select, plus extras such as
  menu or volume), switch type.
- **Power:** battery chemistry and capacity, target play time, charging
  (USB-C?), power switch, fuel gauge.
- **Audio extras:** headphone jack? Volume control (analog or digital)?
- **Connectivity:** firmware update over USB as well? Form of the
  debug/programming header (SWD pads vs. connector) on each board.
- **Enclosure / form factor:** dimensions, layout (vertical vs. horizontal),
  how the board is mounted.
- **Manufacturing:** cost budget, if there is a cap.
