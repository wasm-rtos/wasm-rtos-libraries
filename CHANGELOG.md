# Changelog

## 1.0.0 — 2026-11-13

The first release, for wasm-rtos's guest ABI 1.8.

- **The Arduino core**, built into a program's `.wasm` by
  `arduino/build.sh`: `pinMode`, `digitalRead`/`Write`, interrupts (between
  `loop()` calls and in `delay()`), `millis`/`delay`, `analogRead`,
  `analogWrite` (PWM or an analog output), `tone`, `Serial` as the program's
  output, `Serial1`/`Serial2` on the board's ports, `String`, `Print`,
  `Stream`. Pins and buses by the device's names.
- **Libraries**: `Wire` (repeated starts as one transaction), `SPI`,
  `OneWire` (search in the kernel), `Encoder` on the counter interface, and
  the LED strip API on the pulse interface.
- Libraries that use only the Arduino API build unmodified.

Decisions: [`docs/decisions.md`](docs/decisions.md).
