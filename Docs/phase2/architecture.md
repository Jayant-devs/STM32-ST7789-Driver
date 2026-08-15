# Phase 2 Architecture: Custom DMA-Optimized Display Driver

## Overview

In Phase 2, the display driver will be redesigned from the ground up to support high-throughput, non-blocking video streaming for embedded camera applications.

## Planned Key Features & Driver API

Unlike traditional GUI libraries that rely on nested pixel loops (`DrawFilledRectangle` -> `DrawLine` -> `DrawPixel`), the custom Phase 2 driver is structured directly around DMA stream blocks and line-buffered memory transfers.

### Core API Specification
- `TFT_Init()` — Low-level hardware & ST7789 SPI initialization.
- `TFT_SetWindow(x0, y0, x1, y1)` — Configures address window once per frame/block transfer.
- `TFT_SendPixelsDMA(buffer, size)` — Non-blocking DMA transfer of pixel blocks.
- `TFT_SendLine(line_buffer)` — Line-by-line streaming suitable for camera input.
- `TFT_SendFrame(frame_buffer)` — Full frame buffer transfer routine with double buffering.

## Optimization Goals
- Eliminate software loop overheads.
- Integrate double-buffering / ping-pong buffer mechanics.
- Zero CPU load during frame rendering.
