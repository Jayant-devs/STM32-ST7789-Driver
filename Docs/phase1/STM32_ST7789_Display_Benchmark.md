# STM32F446RE + ST7789 Display Driver

## Phase 1: Display Bring-up and Performance Benchmarking

### Project Overview

This phase focused on validating a high-speed display pipeline for
future integration with an OV7670 camera.

The objective was to evaluate the feasibility of real-time image
rendering on a 240×320 ST7789 TFT display using SPI and DMA on an
STM32F446RE.

## Hardware

  Component         Details
  ----------------- ---------------------
  MCU               STM32F446RE
  Display           ST7789 2.4-inch TFT
  Interface         SPI2
  Data Format       RGB565
  Transfer Method   DMA

## Software Stack

``` text
STM32CubeIDE
        ↓
HAL
        ↓
SPI2
        ↓
DMA
        ↓
ST7789 Driver
```

## Benchmark Results

  Operation          Resolution   Time
  ------------------ ------------ -------
  Full-screen fill   240×320      56 ms
  Image rendering    213×120      18 ms

Estimated performance:

-   240×320 fill rate: \~18 FPS
-   213×120 image rendering: \~55 FPS

## Challenges Encountered

-   Display initialization
-   SPI clock tuning
-   DMA configuration
-   RGB565 byte-order mismatch
-   240×320 display configuration
-   Display orientation and coordinate mapping
-   Partial image rendering

## Test Demonstration

Add your benchmark video or a GIF in the repository:

``` text
docs/
├── benchmark.gif
└── images/
```

## Future Work

-   Phase 1 ✓ ST7789 display bring-up
-   Phase 2 → Custom DMA-optimized display driver
-   Phase 3 → OV7670 camera initialization
-   Phase 4 → DCMI frame capture
-   Phase 5 → Live camera feed on the TFT

## Acknowledgments

The initial ST7789 driver implementation was based on the work of
Floyd-Fish.

Further modifications included:

-   240×320 display support
-   DMA-related fixes
-   RGB565 byte-order corrections
-   Performance benchmarking

Original repository:

https://github.com/Floyd-Fish/ST7789-STM32
