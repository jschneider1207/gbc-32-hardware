# 0003: HyperRAM on OCTOSPI holds the active ROM (supersedes ADR 0001's SDRAM)

- Status: Accepted (human decision, recorded 2026-11-24; written up by the
  firmware agent at the human's request)
- Date: 2026-11-24
- Supersedes: the memory choice in ADR 0001 (16-bit FMC SDRAM). ADR 0001's
  other conclusions stand: the whole active ROM sits in CPU-addressable
  external memory, loaded from eMMC, and the core keeps `rom: &[u8]`.

## Context
ADR 0001 chose FMC SDRAM before the MCU was fixed. With the STM32H735IGK6
(ADR 0002) the trade-off changed. The exploration is in
`../gbc-32/docs/reference/stage3-rom-memory-options.md` (2026-11-23, source
tags per claim; confidence stated there as medium, pending a measurement on
an STM32H735G-DK). Its points, with their sources:

- **Speed is about equal on this MCU.** FMC SDCLK <= 95 MHz x 16 bit, about
  190 MB/s (DS13312 SDRAM timings); OCTOSPI DTR with DQS / HyperBus
  <= 100 MHz x 8 bit x 2 = 200 MB/s (DS13312 Table 82). A random 32-byte
  D-cache line fill is estimated at 130-165 CPU cycles from SDRAM and
  155-200 from HyperRAM (estimate, not measured).
- **Pins:** 11-12 instead of about 39, and FMC is no longer shared with an
  FMC display.
- **Errata:** the FMC unaligned-read erratum (ES0491 Rev 9 2.7.3) does not
  apply to OCTOSPI: 2.8.4 Table 5 allows memory-mapped reads at any address
  and any byte count in HyperBus mode. OCTOSPI has its own errata; the ones
  that matter: keep memory-side wrap off, WRAPSIZE = 0 (2.8.2, 2.8.3,
  2.8.15, the last with no workaround), and HyperBus with DQS needs
  latency >= 6 (2.8.13).
- **Testable before the PCB:** the STM32H735G-DK carries this MCU and
  package with an S70KL1281 HyperRAM on OCTOSPI2.
- **SDRAM timing problem found in ADR 0002's draft pin map:** SDNE0 on
  PC2_C adds 4.5 ns (DS13312 Table 76 note 2), which does not fit 95 MHz.
  It goes away with this decision.

## Decision
The active ROM lives in a **3.0 V HyperRAM on OCTOSPI**, memory-mapped.
The part is not chosen yet; candidates from the exploration: S70KL1281 (as
on the DK) or S27KL0642 class, at least 8 MiB (64 Mbit).

## Consequences
- ADR 0001's SDRAM is dropped. Its status line now points here.
- ADR 0002's pin-fit draft assumed FMC SDRAM (and SDNE0/SDCKE0 on PC2/PC3):
  to redo with OCTOSPI. The DK's OCTOSPIM_P2 hookup is the reference
  (exploration note, "DK pinout").
- Firmware (`docs/firmware-interface.md` updated): OCTOSPI HyperBus init,
  an eMMC -> HyperRAM loader (memory-mapped writes are supported, RM0468
  section 25.2), an MPU region making the mapped window normal/cacheable.
  The byte-wide ROM read rule in `../gbc-32` (its ADR 0027) existed for the
  FMC erratum and has no hardware reason left; retiring it is a firmware
  decision.
- Still open: the HyperRAM part number and JLC stock; the DK measurement
  the exploration asked for (real-game D-cache miss rates, bring-up
  settings: fixed latency, 100 MHz, REFRESH below tCSM); the display
  interface, now free to use FMC alone.
