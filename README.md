# wasm-rtos-libraries

Libraries for programs that run on [wasm-rtos](https://github.com/wasm-rtos/wasm-rtos).
They become part of a program's `.wasm`; the operating system carries none
of them.

| Directory | What it is |
|---|---|
| [`arduino/`](arduino/README.md) | Arduino core and ports of Arduino libraries (Wire, SPI, OneWire, Encoder, NeoPixel): sketches built into wasm-rtos programs |

Each library builds against `include/wasm-rtos.h` from a wasm-rtos checkout,
found through `WASM_RTOS` (by default `wasm-rtos` next to this repository).
