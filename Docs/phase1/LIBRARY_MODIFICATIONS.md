# ST7789 Library Modifications

## Project Information

**MCU:** STM32F446RE  
**Display:** 2.4" ST7789 TFT (240×320)  
**Original Library Author:** Floyd-Fish  
**Original Repository:** https://github.com/Floyd-Fish/ST7789-STM32  

---

# Why This Document Exists

The original library was used as the starting point for this project. However, several modifications were required to support a 240×320 display and to prepare the display driver for future camera-streaming experiments.

This document records every modification made to the original implementation.

---

# Modification 1: Added 240×320 Display Support

## Original Library
Supported displays:
- 135×240
- 170×320
- 240×240

## Added Configuration
```c
#define USING_240X320
```
```c
#define ST7789_WIDTH 240
#define ST7789_HEIGHT 320
```

## Reason
The display used in this project has a native resolution of 240×320 pixels. Without adding native support, image rendering and coordinate calculations were incorrect.

---

# Modification 2: Rotation Configuration

## Added
```c
#define ST7789_ROTATION 2
```

## Reason
The image orientation was incorrect. Different ST7789 modules mount the LCD panel differently, requiring different MADCTL configurations. Several rotation values were tested before the correct orientation was found.

---

# Modification 3: X_SHIFT and Y_SHIFT Verification

## Reason
ST7789 controllers frequently use an internal frame buffer that differs from the visible display area. The shift values compensate for these internal offsets.

Incorrect values resulted in:
- Cropped images
- Misaligned coordinates
- Improper image positioning

---

# Modification 4: DMA Debugging

## Original Configuration
```c
#define USE_DMA
```

## Procedure
DMA was temporarily disabled.

## Reason
Disabling DMA simplified debugging. The display initialization sequence and SPI communication were verified before DMA was re-enabled.

---

# Modification 5: RGB565 Byte-Order Correction

## Observed Behavior
Incorrect colors were displayed:
- Red appeared pink.
- Yellow appeared orange.

## Root Cause
DMA was transmitting 16-bit RGB565 pixels with an incorrect byte order.

## Solution
Pixel bytes were swapped before transmission.

## Reason
The ST7789 expects RGB565 data in big-endian format.

---

# Modification 6: SPI Clock Optimization

## Original Result
```text
Full-screen refresh = 110 ms
```

## Updated Configuration
```text
SPI Prescaler = 2 (CubeMX initial configuration photo shows Prescaler = 4)
```

## Updated Result
```text
Full-screen refresh = 56 ms
```

## Reason
Increasing the SPI clock nearly doubled the display throughput.

---

# Modification 7: Performance Benchmarking

## Full-Screen Fill
```c
ST7789_Fill_Color(RED);
```
Result:
```text
240×320 = 56 ms (with DMA) vs ~340 ms (without DMA / polled rendering)
```

---

## Image Rendering
Image specifications:
```text
Resolution = 213×120
Format = RGB565
```
Result:
```text
Image rendering = 18 ms
```

---

# Modification 8: Rectangle Rendering Investigation

## Observed Behavior
```text
100×100 rectangle = 340 ms (without DMA)
```

## Investigation
The original implementation used:
```text
DrawFilledRectangle()
       ↓
   DrawLine()
       ↓
   DrawPixel()
```
instead of DMA-based block transfers.

## Conclusion
The library is optimized for static GUI rendering rather than real-time video streaming.

---

# Modification 9: Partial Screen Update Validation

The same image was rendered at multiple screen positions. This verified:
- Coordinate translation
- Address windowing
- Rotation settings
- Partial updates

---

# Final Benchmark Results

| Operation | Resolution | Time | FPS |
| :--- | :--- | :--- | :--- |
| **Screen Fill (Polled / No DMA)** | 240×320 | ~340 ms | ~3 |
| **Screen Fill (DMA Enabled)** | 240×320 | 56 ms | ~18 |
| **Image Rendering (DMA)** | 213×120 | 18 ms | ~55 |

---

# Future Work

- Custom ST7789 driver
- DMA-only rendering
- Line-buffer implementation
- OV7670 integration
- DCMI frame capture
- Live camera feed

---

# Acknowledgments

This project is based on the original ST7789 library developed by **Floyd-Fish**.

Additional work performed in this repository includes:
- Display adaptation
- DMA validation
- RGB565 debugging
- SPI optimization
- Performance benchmarking

Original repository: https://github.com/Floyd-Fish/ST7789-STM32
