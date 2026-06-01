# Telemetry Sentinel
 
Bare-metal Rust firmware on an STM32F407 that watches machine vibration, processes it on-device, and flags anomalies before failure - streaming condition-monitoring telemetry over MQTT to a time-series backend and live dashboard.
 
> A constrained sensor node -> edge gateway -> cloud pipeline, built as a study in real-time embedded systems and end-to-end IoT architecture.
 
## Architecture
 
```mermaid
flowchart LR
    S[Accelerometer] -->|SPI| M[STM32F407<br/>Rust / no_std / Embassy<br/>edge RMS + thresholding]
    M -->|USB CDC serial| G[Gateway service<br/>Rust]
    G -->|MQTT| DB[(TimescaleDB<br/>time-series store)]
    DB --> UI[Dashboard<br/>live + historical]
```
 
The microcontroller does the signal work at the edge - sampling the accelerometer and computing rolling RMS with anomaly thresholding - so only meaningful events travel upstream. A lightweight gateway forwards the serial stream over MQTT into a time-series database, visualized on a web dashboard. The link layer buffers locally and reconnects on drop for at-least-once delivery.
 
## Tech stack
 
**Firmware** - Rust, no_std, Embassy (async embedded), STM32F407 / ARM Cortex-M4, SPI, USB CDC, probe-rs, defmt
**Backend** - Rust, MQTT, TimescaleDB / PostgreSQL
**Frontend** - web dashboard (live + historical telemetry)
 
## Build
 
```bash
rustup target add thumbv7em-none-eabihf
cargo install probe-rs-tools --locked
cargo run --release --bin blinky   # builds, flashes, and attaches to the connected STM32F407
```
 
## Why this design
 
- **Edge-gateway topology** - keeps the constrained node focused on real-time sensing while networking lives where it is cheap, mirroring how industrial IoT systems are actually built.
- **On-device processing** - computing RMS and thresholds on the MCU cuts upstream bandwidth and surfaces condition changes early.
- **Reliability first** - local buffering and reconnect logic over a lossy link, not just a happy-path demo.