# ST7789 Library Modification Log

# STM32F446RE + ST7789 (240×320) Bring-Up Notes

## Purpose

The original library was designed for several ST7789 display variants,
but it did not directly support the 2.4-inch 240×320 display used in
this project.

Several modifications were required to make the display operate
correctly on an STM32F446RE and to improve performance for future
camera-streaming experiments.

------------------------------------------------------------------------

## 1. Added 240×320 Display Support

### Original state

The library only provided predefined configurations for:

-   135×240
-   170×320
-   240×240

### Modification

Added a new configuration:

``` c
#define USING_240X320
```

Added the corresponding dimensions:

``` c
#define ST7789_WIDTH 240
#define ST7789_HEIGHT 320
```

### Why?

The display module used in this project has a native resolution of
240×320 pixels.

Without this change, the address window and coordinate calculations were
incorrect.

------------------------------------------------------------------------

## 2. Corrected Rotation Configuration

### Modification

``` c
#define ST7789_ROTATION 2
```

### Why?

Different ST7789 modules mount the LCD panel differently.

Several rotations were tested until the correct image orientation was
found.

------------------------------------------------------------------------

## 3. Verified X_SHIFT and Y_SHIFT

### Why?

Many ST7789 displays use an internal frame buffer that is larger than
the visible display area.

The shift values compensate for this internal offset.

Incorrect values result in:

-   Misaligned images
-   Cropped images
-   Incorrect coordinates

------------------------------------------------------------------------

## 4. Disabled DMA During Initial Debugging

### Original state

``` c
#define USE_DMA
```

### Modification

DMA was temporarily disabled.

### Why?

Disabling DMA simplified debugging and allowed the SPI communication and
initialization sequence to be verified first.

Once the display was working correctly, DMA was re-enabled.

------------------------------------------------------------------------

## 5. Fixed RGB565 Byte Ordering During DMA Transfers

### Observed behavior

Colors appeared incorrectly:

-   Red became pink
-   Yellow became light orange

### Cause

DMA was transmitting 16-bit pixel data using an incorrect byte order.

### Solution

The pixel bytes were swapped before transmission.

### Why?

The ST7789 expects pixel data in big-endian RGB565 order.

------------------------------------------------------------------------

## 6. Verified SPI Clock Configuration

### Modification

The SPI prescaler was reduced to:

``` text
SPI Prescaler = 2
```

### Why?

Increasing the SPI clock improved transfer bandwidth.

### Performance results

  Configuration        Full-screen fill
  -------------------- ------------------
  Previous SPI clock   110 ms
  Prescaler = 2        55 ms

------------------------------------------------------------------------

## 7. Benchmarked Full-Screen DMA Rendering

### Test

``` c
ST7789_Fill_Color(RED);
```

### Result

``` text
240×320 fill time = 56 ms
```

### Why?

This benchmark established the maximum throughput of the display
pipeline.

------------------------------------------------------------------------

## 8. Benchmarked Image Rendering

### Test image

-   Resolution: 213×120
-   Format: RGB565

### Result

``` text
Image rendering time = 18 ms
```

### Why?

This benchmark simulated a future camera frame.

------------------------------------------------------------------------

## 9. Investigated Rectangle Rendering Performance

### Observed behavior

A 100×100 rectangle required approximately 340 ms.

### Cause

The library implementation used:

``` c
DrawFilledRectangle()
        ↓
DrawLine()
        ↓
DrawPixel()
```

Instead of using DMA.

### Why this matters

The current graphics implementation is optimized for static graphics
rather than real-time video.

------------------------------------------------------------------------

## 10. Validated Partial Screen Updates

### Test

The same image was rendered at multiple coordinates.

### Why?

This verified:

-   Address windowing
-   Coordinate translation
-   Partial updates
-   Rotation settings

------------------------------------------------------------------------

## Final Performance

  Operation                    Time    Estimated FPS
  ---------------------------- ------- ---------------
  Full-screen fill (240×320)   56 ms   \~18 FPS
  Image rendering (213×120)    18 ms   \~55 FPS

------------------------------------------------------------------------

## Future Work

-   Develop a custom ST7789 driver.
-   Implement line-buffered rendering.
-   Configure the OV7670 camera.
-   Capture frames using DCMI + DMA.
-   Stream camera frames to the display.

------------------------------------------------------------------------

## Acknowledgments

The original ST7789 implementation used in this project was based on the
work of Floyd-Fish.

Repository:

https://github.com/Floyd-Fish/ST7789-STM32

This project extends the original implementation through debugging,
configuration changes, DMA validation, benchmarking, and preparation for
future camera integration.
