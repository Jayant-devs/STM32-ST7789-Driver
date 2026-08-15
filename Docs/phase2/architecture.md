# Phase 2 Architecture: Custom DMA-Optimized Display Driver

## Overview

In Phase 2, the display driver will be redesigned from the ground up to support high-throughput, non-blocking video streaming for embedded camera applications.

Unlike traditional GUI libraries that rely on nested pixel loops (`DrawFilledRectangle` ➔ `DrawLine` ➔ `DrawPixel`), the custom Phase 2 driver is structured directly around DMA stream blocks and line-buffered memory transfers.

---

## 🏗️ Architecture Pipeline

### Phase 2: Custom DMA Display Driver Flow
```text
Application Logic
        │
        ▼
Graphics / Windowing API
        │
        ▼
Frame / Line Buffer (SRAM)
        │
        ▼
DMA Engine (Stream Transfer)
        │
        ▼
SPI2 Peripheral (22.5 MHz)
        │
        ▼
ST7789 TFT Controller (240×320)
```

### Phase 3+: DCMI Camera Stream Pipeline
```text
OV7670 Camera Module
        │
        ▼
DCMI Peripheral (Parallel Bus + HSYNC / VSYNC / PCLK)
        │
        ▼
DMA Engine (Camera Frame Capture)
        │
        ▼
Ping-Pong Double Buffer (SRAM)
        │
        ▼
SPI DMA Engine (Display Stream)
        │
        ▼
ST7789 240×320 Display (~30+ FPS Goal)
```

---

## 🛠️ Planned Driver API Specification

### Core Low-Level Driver Interface
- `TFT_Init()` — Low-level hardware, GPIO, & ST7789 SPI initialization sequence.
- `TFT_SetWindow(x0, y0, x1, y1)` — Configures address window once per block transfer.
- `TFT_SendPixelsDMA(buffer, size)` — Non-blocking DMA transfer of pixel blocks.
- `TFT_SendLine(line_buffer)` — Line-by-line streaming suitable for camera input.
- `TFT_SendFrame(frame_buffer)` — Full frame buffer transfer routine with double buffering.

---

## 🎯 Optimization Goals

1. **Zero CPU Overhead**: Complete offloading of pixel streaming to the DMA hardware controller.
2. **Line-Buffered Rendering**: Minimize RAM memory footprint by processing display data line-by-line.
3. **Double-Buffering**: Prevent screen tearing during real-time video playback.
