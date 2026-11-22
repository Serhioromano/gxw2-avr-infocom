# AVRInfacomV3

PLC source exports for an automatic transfer switch (АВР) that selects between two mains feeds and a standby generator. Target: Mitsubishi FX3G, programmed in GX Works 2 using Structured Text (`.iecst`).

## Layout

- `PRG_INIT.iecst` — initializes alarm definitions and assigns breaker alarm IDs.
- `PRG_MAIN.iecst` — handles manual and automatic source selection, breaker commands and interlocks, and generator start control.
- `PRG_PROCESS.iecst` — initializes Modbus RTU and handles alarms, resets, and event logging.
- `_gvl.csv` — shared labels; `_io.csv` — physical I/O mappings; `PRG_*.csv` — local program labels.
- `FB_VAV.iecst` / `FB_VVA.csv` — breaker function-block body and labels for `VVA` (note the different filenames).
- `плк.txt` — PLC IO bindings.

## Breaker recovery

- On this installation all three breakers open on loss of drive power and begin a 7 s spring charge when power returns. Keep ON blocked for the existing 10 s startup delay after RUN or detected loss of drive power; do not latch new individual feedback alarms during that delay. The gate assumes `DI_Avarkontrol` (`X13`) is high in the normal state; verify at commissioning. X13 does not necessarily detect a UPS sag.
- `PRG_MAIN` owns position requests and selects one source at a time. `VVA` does not write the operating mode. It allows up to three ON attempts per request, with step `10` for the pulse and step `20` for the 7 s pause before retry or failure. On AUTO → MANUAL, copy the actual breaker positions into the requests and discard pending AUTO commands; normal protections still apply. Clear `xTempError` before each `fbAmIsOn` call; keep the `xError` latch separate. OFF is independent of ON retry timing. The generator is started by external automation at this site.

- In AUTO, use `TC32` (`K11`, nominal 1.1 s) to qualify a new ON request only after all three OFF inputs stay high continuously with no active request. Restart the wait after cancelling an ON request. Keep an existing request when its target closes or is retrying; the other breakers' OFF interlocks still apply. MANUAL does not use `TC32`.
- In MANUAL, `PRG_MAIN` clears each HMI ON toggle (`gxOn1`, `gxOn2`, `gxOnGenerator`) if its breaker is open and the ON request was rejected, cancelled, or exhausted its retries. Keep the toggle set while a request is still retrying or its breaker is closed. The HMI must display the PLC bit and not continuously rewrite it.

## ST formatting

- Write explanatory ST comments and label `Comment` descriptions in Russian. Keep identifiers unchanged; describe each declared label's purpose and signal polarity where applicable.
- Use `Vkl` in identifiers for the breaker ON command cycle: `xVklFailed`, `fbVklPulse`, `fbVklWait`, `iVklStep`, `iVklAttempts`. Keep actual breaker position distinct from commands.
- Write multi-argument function block calls and instruction calls with compound conditions in multiline form: place `Name(` on its own line, indent each argument by four spaces, and align the closing `);` with `Name(`. Use named arguments with `:=` for function blocks.
- In compound conditions, put each distinct condition on its own line, starting continuation lines with `AND` or `OR`. Add `(* ... *)` comments next to conditions whose purpose is not obvious. Keep closely related terms on one line when they express one reason (e.g. `OR TS1 OR TS2 (* A mains source is ready. *)`). Short, simple instructions and repetitive table-like calls may stay on one line.

## Working here

- Use `gxw2-st` for ST and label edits, and `gxw2-libraries` for Coolmay alarm, Modbus, time-control, and utility functions.
- Preserve the GX Works 2 label format (UTF-16 LE with BOM, tab-delimited despite the `.csv` extension).
- No native GX Works 2 project or automated build/test workflow is included. Validate PLC changes in the GX Works 2 project.
- Update this file after code changes. Leave commits to the maintainer.
