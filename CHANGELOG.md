# Changelog: PUMP for Digitakt

**This is the Digitakt mk1 version** (`pump-dt`). For the Digitone, see
[PUMP for Digitone](https://github.com/ClaidArremer/digitone-pump).

Each version's files are in `elemods/` (`pump-dt-<version>-os1.54.elemod` and `-os1.53`); the newest is there,
older ones are in the history of this repo.

## 1.4 (2026-11-01)

- **REL reads the other way**: more is slower, as the stock compressor's REL (0 PUMP, fast; 127 LEVEL, slow).
  Patterns saved before sound as they did; their REL now shows as 127 minus its old number.
- **RAT shows its ratios**: AUT, 2:1, 3:1, 4:1, 6:1, 10:1, 20:1, LIM (1.3 showed numbers, and 20:1 as 21).
- **FUNC + YES** is the stock TEMP SAVE again (1.3 took it for the graph).
- Every change shows in the **stock drop-down**, as the other pages do (*Ratio=4:1*, *Output Level=-3.5dB*).

## 1.3 (2026-10-24)

- The first release: the Digitone's PUMP on elekloader's Digitakt core 4.1, in place of the stock compressor.
  It takes the Compressor page, skips the stock compressor in the render, and keeps its settings in the
  compressor's eight parameters, so they are saved per pattern and over a power-off.
