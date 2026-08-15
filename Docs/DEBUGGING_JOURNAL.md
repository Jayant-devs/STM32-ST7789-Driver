# Embedded Systems Debugging Journal

**Project:** STM32F446RE + ST7789 TFT Display Driver Bring-Up  
**Author:** Jayant Chopra  
**Target Hardware:** STM32F446RE (ARM Cortex-M4), ST7789 2.4" 240×320 SPI TFT Display  

---

## 📌 Introduction & Purpose

Bring-up of embedded hardware often exposes subtle mismatches between microcontrollers, peripheral buses, software libraries, and display controllers. 

This journal documents the engineering problems, root cause analyses, debugging strategies, and hardware/software solutions encountered during Phase 1 of this project.

---

## 🐞 Bug #1: Incorrect Colors during DMA Transfers (Pink Red & Light Orange Yellow)

### 🚨 Observed Symptom
When rendering test patterns or color fills using Direct Memory Access (DMA):
- Pure **RED** (`0xF800`) rendered as a bright **PINK / MAGENTA**.
- Pure **YELLOW** (`0xFFE0`) rendered as **LIGHT ORANGE**.
- Primary colors appeared inverted or shifted in tint across the panel.

### 🔍 Investigation & Root Cause
1. Polled SPI transfers without DMA rendered standard colors, but enabling DMA corrupted color output.
2. The ST7789 display controller expects 16-bit RGB565 pixel data in **Big-Endian** order (High Byte sent over SPI first, followed by Low Byte).
3. The ARM Cortex-M4 (STM32F446RE) memory architecture is **Little-Endian**. 
4. When passing raw 16-bit pixel buffers (`uint16_t`) directly to `HAL_SPI_Transmit_DMA()`, the STM32 memory layout transferred the Low Byte first, swapping the red, green, and blue color bit fields inside the ST7789 input register.

### 💡 Solution
Implemented a byte-swapping conversion routine (`__builtin_bswap16` or manual byte shifts `(color >> 8) | (color << 8)`) before passing frame buffer regions to the DMA transmission handler.

```c
// Endianness byte swap for ST7789 RGB565 DMA stream
uint16_t swap_color(uint16_t color) {
    return (color >> 8) | (color << 8);
}
```

---

## 🐞 Bug #2: Severe Rendering Latency (~340 ms for Color Fill & Rectangles Without DMA)

### 🚨 Observed Symptom
Calling primitive drawing functions like `DrawFilledRectangle()` or rendering full-screen color fills without DMA resulted in visually noticeable slow "wipes" across the screen, taking up to **~340 ms** to render a 100×100 block or full red fill.

### 🔍 Investigation & Root Cause
Inspecting the original driver's call stack revealed nested pixel-by-pixel function calls:
```text
DrawFilledRectangle()
       ↓
   DrawLine()
       ↓
   DrawPixel()
```
For a 240×320 display (76,800 pixels), `DrawPixel()` was sending separate SPI commands (Column Address Set `0x2A`, Page Address Set `0x2B`, Memory Write `0x2C`) and toggling CS/DC GPIO control lines **76,800 times per frame**!

### 💡 Solution
1. Configured ST7789 **Address Windowing** once per block transfer (`ST7789_SetAddressWindow(x0, y0, x1, y1)`).
2. Switched from pixel-by-pixel software loops to **DMA Block Transfers** (`HAL_SPI_Transmit_DMA()`), streaming pixel blocks continuously without CPU intervention.
3. Rendering time for full-screen fills dropped from **340 ms down to 56 ms**.

---

## 🐞 Bug #3: Display Image Truncation & Coordinate Misalignment (X_SHIFT & Y_SHIFT)

### 🚨 Observed Symptom
Image arrays (e.g. 213×120 test image) appeared cropped on the edges, offset vertically by several pixels, or rendered outside the visible LCD screen boundary.

### 🔍 Investigation & Root Cause
1. ST7789 controller chips contain internal RAM buffers sized up to 240×320 pixels, but generic LCD breakout modules vary in physical screen dimensions and mounting orientations.
2. The library defaulted to 240×240 or 135×240 configurations, leaving active window shift offsets (`X_SHIFT` and `Y_SHIFT`) unset for 240×320 displays.
3. Incorrect shift offsets caused the address window commands (`0x2A` / `0x2B`) to write into off-screen controller RAM.

### 💡 Solution
- Added explicit native support for 240×320: `#define USING_240X320`, `#define ST7789_WIDTH 240`, `#define ST7789_HEIGHT 320`.
- Verified rotation setting `#define ST7789_ROTATION 2` (MADCTL `0x36` register value) and confirmed `X_SHIFT = 0`, `Y_SHIFT = 0` for this specific 2.4" panel hardware variant.

---

## 🐞 Bug #4: SPI Clock Prescaler Tuning & Configuration Discrepancy

### 🚨 Observed Symptom
In initial CubeMX setup configuration (`spi_config.png`), the SPI2 prescaler was set to **4** (`BaudRatePrescaler = SPI_BAUDRATEPRESCALER_4`), resulting in a full-screen fill time of ~110 ms.

### 🔍 Investigation & Root Cause
1. On STM32F446RE, SPI2 sits on the APB1 peripheral bus (max clock speed 45 MHz).
2. Operating at prescaler 4 delivered ~11.25 MHz SPI clock frequency, which was safe for initial breadboard wiring but throttled display throughput.

### 💡 Solution
- Tested SPI stability while reducing prescaler from 4 down to **2** (delivering ~22.5 MHz clock frequency).
- Full-screen 240×320 fill time improved from **110 ms to 56 ms** (~18 FPS).
- Note: CubeMX setup screenshot `spi_config.png` reflects the initial conservative prescaler = 4 bring-up configuration, which was later tuned to prescaler = 2 in code.

---

## 🐞 Bug #5: DMA Initialization Conflicts During Initial Bring-Up

### 🚨 Observed Symptom
During initial display startup, enabling `#define USE_DMA` caused the LCD initialization command sequence to fail or freeze randomly before display startup finished.

### 🔍 Investigation & Root Cause
Interleaving multi-byte SPI commands (like software reset `0x01`, sleep out `0x11`, display on `0x29`) with DMA transfer interrupts caused race conditions before the display controller completed hardware wake-up routines.

### 💡 Solution
1. Disabled DMA (`//#define USE_DMA`) during initial board bring-up and verified command responses using blocking `HAL_SPI_Transmit()`.
2. Once initialization sequence stability was verified, DMA was re-enabled strictly for bulk pixel data transfers (`ST7789_Fill_Color`, `ST7789_DrawImage`), keeping startup command sequences on blocking HAL SPI calls.

---

## 📊 Summary of Debugging Impact

| Challenge / Bug | Initial State / Symptom | Root Cause | Final Resolution | Performance / Outcome |
| :--- | :--- | :--- | :--- | :--- |
| **Color Corruption** | Pink Red & Orange Yellow | Cortex-M4 Little-Endian vs ST7789 Big-Endian | Pre-swapped RGB565 bytes for DMA | Accurate RGB colors |
| **Rendering Latency** | ~340 ms per fill | Pixel-by-pixel `DrawPixel` software loop | DMA window block transfers | **56 ms fill time** |
| **Image Shift** | Cropped & offset image | Missing 240×320 offset & MADCTL rotation | Configured `USING_240X320` & `ROTATION 2` | Perfect window alignment |
| **SPI Throughput** | Prescaler = 4 (110 ms) | Conservative initial bus speed | Tuned SPI prescaler to **2** | **55 ms fill time (2x speedup)** |
| **DMA Instability** | Random freeze on startup | DMA race condition during LCD init | Polled SPI for LCD init, DMA for frames | 100% startup reliability |
