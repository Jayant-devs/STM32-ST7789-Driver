# Phase 3 & Beyond: OV7670 Camera & DCMI Integration

## Overview

This document outlines the planned architecture for capturing real-time camera frames using the STM32F446RE Digital Camera Interface (DCMI) and streaming them directly to the ST7789 display.

## System Architecture Pipeline

```text
OV7670 Camera Module
       ↓ (I2C / SCCB Configuration)
STM32 DCMI Peripheral (Parallel Data Bus + HSYNC / VSYNC / PCLK)
       ↓ (DMA Transfer)
RAM Line Buffer / Double Buffer
       ↓ (SPI2 + DMA Stream)
ST7789 240×320 TFT Display (~30+ FPS Goal)
```

## Milestone Objectives
- **Phase 3**: Double-buffering architecture setup on STM32 RAM.
- **Phase 4**: OV7670 camera initialization via I2C/SCCB.
- **Phase 5**: DCMI + DMA frame capture configuration.
- **Phase 6**: Complete real-time camera-to-TFT video streaming pipeline.
