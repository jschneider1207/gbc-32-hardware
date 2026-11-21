# 0001: External FMC SDRAM holds the active ROM

- Status: Accepted
- Date: 2026-11-21

## Context
The game library is stored on eMMC, and ROMs are uploaded over USB. eMMC is a
block device read through SDMMC, and the CPU can't address it directly. The
emulator core (`../gbc-32`, ADR 0006) takes the cartridge ROM as a byte
slice, `rom: &'a [u8]`, so the whole ROM has to sit in memory the CPU can
address directly. ROMs go up to 8 MiB (the MBC5 maximum). Internal SRAM on
the STM32H7 is about 1 MB, split across several regions (from memory, RM0433;
not yet checked against the datasheet), so a full ROM won't fit there.

Options considered:

- **A1. FMC SDRAM** (16-bit SDR). Mapped into the CPU's address space and
  cacheable, with plenty of bandwidth (about 200 MB/s peak at 16 bits ×
  ~100 MHz). Costs about 39 FMC pins and careful layout. The bus is shared
  with an FMC-driven display if we use one.
- **A2. QSPI/OctoSPI PSRAM.** Few pins, but slower. On the H753, the QUADSPI
  peripheral's memory-mapped mode is read-only (from memory), so loading the
  ROM needs indirect-mode writes. OctoSPI would mean a different H7 part from
  the one the firmware is developed on.
- **A3. QSPI NOR, as on the dev shield, rewritten on every game switch.** The
  firmware path is identical to the dev board, but chip erase and program are
  slow, and switching games wears the flash.
- **B. Bank-fetch API in the core.** The platform layer serves 16 KiB banks
  from eMMC into an internal-RAM cache. No external RAM, but a cache miss
  costs about 1 ms against a 16.7 ms frame. Frame timing becomes unpredictable
  if a game thrashes the cache, and the core API would change.

## Decision
The board uses an external 16-bit SDR SDRAM on the STM32H7 FMC, sized at
least 8 MiB. We'll aim for 32 MiB (a 256 Mbit part such as the Winbond
W9825G6KH or ISSI IS42S16160J), because it costs about the same and leaves
room for framebuffers. Neither part has been checked against a datasheet or
JLC stock yet.

At game start, the platform layer copies the selected ROM from eMMC into
SDRAM and passes the core a `&'static [u8]` over it. Cartridge RAM
(≤ 128 KiB) stays in internal SRAM, and saves are written back to eMMC.

We won't add SDRAM to the dev setup (NUCLEO-H753ZI + X-NUCLEO-GFX02Z1); no
Nucleo SDRAM shield was found. Stage 2 measures headroom with the ROM read
from memory-mapped QSPI NOR instead. That is slower than FMC SDRAM in both
bandwidth and latency, so it's a conservative stand-in. The prototype board
is the first hardware to run from SDRAM.

## Consequences
- **No change to the emulator core.** It keeps the ROM slice. Its only
  constraints are to assume nothing about where the ROM lives and to read it
  byte by byte through the slice.
- **Firmware platform layer (custom board only):** it needs FMC SDRAM init,
  the eMMC → SDRAM loader, and an MPU region that makes the SDRAM normal,
  cacheable memory (or FMC bank swap). The default `0xC0000000` mapping is
  device memory: uncacheable, and unaligned reads fault (from memory; check
  against PM0253 and RM0433).
- **Firmware Stage 2 action:** measure full-speed headroom on real games with
  the ROM in memory-mapped QSPI NOR and D-cache on, and report cycles per
  frame. If it's marginal, reconsider buying a Discovery kit with SDRAM
  (e.g. STM32H745I-DISCO / H747I-DISCO) before the prototype spin.
- **MCU package:** with eMMC, audio over SAI, the display, buttons, USB and
  debug also needing pins, the SDRAM's ~39 pins push the MCU to LQFP176 or
  BGA. That's acceptable, because the board is fab-assembled (PCBA).
- **Display interaction:** if the display also uses FMC, it shares the SDRAM
  data bus. The LCD flex cable is then a stub on a ~100 MHz bus and needs a
  short connector run with series resistors, or a bus buffer. The other route
  is an LTDC/RGB panel with its framebuffer in SDRAM. This goes to the
  display decision.
- **Layout:** at least 4 layers with a solid ground plane. Put the SDRAM next
  to the MCU and length-match the data, address and command lines to SDCLK.
  Give the prototype test points on SDCLK and the control lines so the FMC
  can be run at a reduced clock during bring-up.
- **Follow-ups:**
  - Pick the MCU part number and package.
  - Confirm the SDRAM part against its datasheet and JLC stock.
  - Add the datasheets to `datasheets/`.
  - Update `docs/firmware-interface.md` (RAM row).

## Update 2026-11-21 (verification note; the decision is unchanged)

- **Unaligned reads, checked against the datasheets.** The "from memory"
  note above is now settled for the chosen MCU, STM32H735IGK6. ES0491 rev 9
  §2.7.3, "Unsupported read access with unaligned address" (FMC, status P =
  partial workaround), says: "Read access with unaligned address, such as a
  half-word read access starting at odd address, is not supported." The
  workaround is to compile the code that accesses the FMC region so that
  every access is aligned. The erratum doesn't depend on the MPU mapping,
  so it applies even after the SDRAM is remapped as normal, cacheable
  memory. In Rust the `strict-align` target feature is unstable (on 1.100
  it warns and is slated to become an error, rust-lang/rust#162235). The
  firmware therefore enforces byte reads of the ROM in code, which is a
  task in `../gbc-32/docs/plan/stage-1.md`.
- **Follow-up status.** The MCU part number and package are picked
  (STM32H735IGK6, UFBGA176+25; recorded in `../gbc-32/README.md` Stage 3),
  but there is no hardware ADR for it yet. The datasheets are in
  `datasheets/` (DS13312, RM0468, ES0491).
