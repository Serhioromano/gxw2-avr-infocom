# AVRInfacomV3

PLC source exports for an automatic transfer switch (АВР) that selects between two mains feeds and a standby generator. Target: Mitsubishi FX3G, programmed in GX Works 2 using Structured Text (`.iecst`).

## Layout

- `PRG_INIT.iecst` — initializes alarm definitions and assigns breaker alarm IDs.
- `PRG_MAIN.iecst` — handles manual and automatic source selection, breaker commands and interlocks, and generator start control.
- `PRG_PROCESS.iecst` — initializes Modbus RTU and handles alarms, resets, and event logging.
- `_gvl.csv` — shared labels; `_io.csv` — physical I/O mappings; `PRG_*.csv` — local program labels.
- `FB_VVA.csv` — breaker function-block labels. `FB_VAV.iecst` is empty; note the different spelling.
- `пкл.txt` — PLC IO bindings.

## Working here

- Use `gxw2-st` for ST and label edits, and `gxw2-libraries` for Coolmay alarm, Modbus, time-control, and utility functions.
- Preserve the GX Works 2 label format (UTF-16 LE with BOM, tab-delimited despite the `.csv` extension).
- No native GX Works 2 project or automated build/test workflow is included. Validate PLC changes in the GX Works 2 project.
- Update this file after code changes. Leave commits to the maintainer.
