# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [v1.0.0] - 2026-08-15

### 🎉 Added
- ST7789 240×320 display support (`USING_240X320` resolution macro configuration).
- Non-blocking SPI Direct Memory Access (DMA) block rendering pipeline.
- RGB565 test image rendering benchmark (`213×120` simulated camera frame).
- Multi-phase documentation structure (`Docs/phase1/`, `Docs/phase2/`, `Docs/phase3/`).
- Detailed developer modification log (`Docs/phase1/LIBRARY_MODIFICATIONS.md`) and embedded systems debugging journal (`Docs/phase1/DEBUGGING_JOURNAL.md`).

### 🐛 Fixed
- **RGB565 Endianness Mismatch**: Swapped pixel byte order before DMA transfers to resolve pinkish reds (`0xF800`) and orange yellows (`0xFFE0`).
- **Rotation Configuration**: Calibrated `ST7789_ROTATION 2` (MADCTL `0x36`) for correct orientation.
- **Display Shift Offsets**: Configured `X_SHIFT = 0` and `Y_SHIFT = 0` to align coordinates and eliminate image cropping.
- **DMA Bring-Up Stability**: Isolated LCD command initialization on blocking HAL SPI calls before enabling DMA stream interrupts.

### ⚡ Performance Improvements
- **SPI Prescaler Optimization**: Reduced SPI2 prescaler from 4 (11.25 MHz) to 2 (22.5 MHz), cutting full-screen fill time from **110 ms to 56 ms** (~18 FPS).
- **DMA Acceleration**: Replaced nested software pixel loops (`~340 ms` fill time) with DMA block transfers (**56 ms** full-screen 240×320 fill, **18 ms** 213×120 image render).
