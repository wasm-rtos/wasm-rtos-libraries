# Decisions

The choices that shape these libraries: what was decided, why, and what
was considered instead. The kernel's own are in the wasm-rtos repository's
`docs/decisions.md`.

## 1. A library inside the program, not in the system

**Decided (2026-11-02).** The Arduino core is source that `build.sh`
compiles into the sketch's `.wasm`. The operating system has no Arduino
code; a program that does not use the core carries none of it. The core is
written here, under these libraries' license, against wasm-rtos's ABI.

**Why.** The kernel offers one interface to every language; what a language
or framework adds belongs to the programs that use it. A sketch is then an
ordinary program: it runs on every board and in the simulator, and the
core changes without touching any device.

**Instead.** The core in the kernel: every build would pay for it, and the
ABI would grow a second, Arduino-shaped interface. The official core's
API sources: under the LGPL, which these public-domain libraries do not
take in.

## 2. Pins and buses by their names

**Decided (2026-11-02).** Pin `N` is the device profile's pin `"gpioN"`;
`LED_BUILTIN` is `"led"`; `A0` is the analog channel `"a0"`; `Wire` is
`"i2c0"`, `Wire1` `"i2c1"`, `SPI` `"spi0"`, `Serial1` `"uart1"`. A sketch
may name other pins (`wr_arduino_pin_name`).

**Why.** Numbers mean one chip's pins; names mean what is wired. The same
sketch then runs on any board whose profile has the names, as programs do.

**Instead.** A table of numbers per board: a build per board, which the
system is meant to avoid.

## 3. Interrupts between loop() calls

**Decided (2026-11-02).** `attachInterrupt` watches the pin through the
kernel; the handler runs between two `loop()` calls and inside `delay()`,
never in the middle of other code.

**Why.** The kernel never calls into a program (ABI rule): a program runs
its own code, one call at a time. Between `loop()` calls and in `delay()` a
sketch is at a point where nothing of its own is half done, which is where
handlers can run without making every library interrupt-safe.

**Instead.** Real asynchronous handlers: callbacks into programs, which
the ABI leaves out for 1.x; and handlers in the middle of code, which most
libraries are not written for.

## 4. Serial is the program's output

**Decided (2026-11-02).** `Serial` writes to standard output, which the
system keeps per program or sends to a UART; reading it gives nothing.
`Serial1` and `Serial2` are the board's serial ports.

**Why.** What a sketch prints is its output; where it goes is the system's
business (the page's console, a board's cable). The ports for wires to
other devices are the numbered ones, as on boards with several.

**Instead.** `Serial` as the first UART: a sketch's prints would take a
port away from devices, and be lost on boards where that UART is wired to
something.

## 5. Buses as transactions

**Decided (2026-11-02, 1-Wire 2026-11-07).** `Wire` turns
`beginTransmission` … `endTransmission(false)` + `requestFrom` into one
transaction with a repeated start; `SPI` and `OneWire` take the whole bus,
each call one transaction; libraries drive chip select with
`digitalWrite`; `OneWire.search()` runs in the kernel.

**Why.** The kernel moves bytes on a bus in transactions it can run in the
background and share between programs; a byte at a time from a program
would be slow and could not keep a protocol's timing.

**Instead.** Bit-banging from the program: the kernel's clock counts
milliseconds, and the program can be preempted between bits.

## 6. Own code with the same API, only where the hardware differs

**Decided (2026-11-07, LED strips 2026-11-08).** Libraries that only use
the Arduino API are built unmodified (displays, sensors, 1-Wire
thermometers were checked). Where a library writes a chip's registers, the
library here is our own code with the same API and header name, on a
kernel interface: `Encoder` on `counter`, the LED strip library on
`pulse`; even its brightness arithmetic follows the original, so fades give
the same colors.

**Why.** Sketches name these headers; with the same API they build as
they are. The hardware work (counting edges, timing bits) belongs to the
kernel's drivers, which the boards implement.

**Instead.** Patching each library's register code per board: one copy
per chip, and none on the page.

## 7. Time in milliseconds

**Decided (2026-11-02).** `millis`, `micros` and `delay` are WASI clocks and
sleep; `delayMicroseconds` waits at least as long by giving other
programs a turn. `pulseIn` and microsecond bit-banging are not offered.

**Why.** The kernel's clock counts milliseconds and programs share the
processor; a busy loop of microseconds would steal it and still not be
exact.

**Instead.** Spinning on a fine clock: exact only on a board running one
program, which is not what the system is.
