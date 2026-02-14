# Airgapped QR Code Transfer Web App

Airgapped QR Code Transfer is a browser-only file transfer tool for offline or isolated environments.
It moves files from a sender screen to a receiver camera using QR frames.

The app is built with Vue.js and uses:
- `qrcode.js` for QR generation
- `zbar-wasm` for QR scanning
- `pako` for gzip compression/decompression

[![Open in Flexpilot AI Web IDE](https://badges.flexpilot.ai/open-in-web-ide.svg)](https://flexpilot.ai/web-ide-redirect?provider=github&owner=mohankumarelec&repo=airgapped-qr-code-transfer&branch=master)

## What Is Implemented

- Adaptive QR payload sizing to avoid `code length overflow`
- Sender presets for speed/reliability tradeoffs
- Scanner auto-negotiation (no manual scanner tuning UI)
- Selective retransmission with human-friendly retry code
- Per-chunk CRC32 validation on receiver
- End-to-end SHA-256 verification for final file integrity
- Duplicate packet suppression and optimized chunk storage for better long-run performance

## Quick Start

1. Open `scanner.html` on receiver device.
2. Allow camera permission.
3. Open `generator.html` on sender device.
4. Select a sender preset.
5. Choose a file and click `Start Transfer`.
6. Point receiver camera at sender QR.
7. If chunks are missing, generate retry code on receiver and paste it on sender.
8. Sender resends only missing chunks.

## Sender Presets

Sender tuning is preset-based now.
Choose profile in `generator.html` UI.

Available presets:
- `Fast`
- `Aggressive`
- `Extreme`
- `Balanced`
- `Reliable`

`Aggressive` and `Extreme` increase throughput but can increase packet loss depending on camera, focus, distance, and display quality.

### Edit Presets in Source

Presets are defined in `generator.html` under:
- `SENDER_PRESETS`
- `DEFAULT_PRESET_NAME`

Update these objects to customize profile behavior for your hardware.

## Scanner Auto-Negotiation

Scanner has no manual performance form now.
It auto-adjusts scanning behavior from sender profile metadata plus local camera heuristics:
- scan interval
- scan resolution cap
- scan crop ratio
- duplicate payload skipping

This keeps receiver UX simple while still adapting to fast or reliable sender profiles.

## Retry and Integrity Flow

1. Receiver validates each chunk using CRC32.
2. Missing or corrupt chunks are tracked.
3. Receiver can generate retry code (`R1:...`) that encodes missing chunk ranges.
4. User pastes retry code into sender.
5. Sender validates retry-code checksum and session, then resends only requested chunks.
6. After reconstruction, receiver computes SHA-256 and compares with sender SHA-256.

If SHA-256 mismatches, receiver requests retransmission.

## Packet Format (Current)

- Metadata packet: `M:...`
- Data packet: `D:...`
- Retry code: `R1:...`

Metadata includes session and file integrity info.
Data packet includes chunk index and CRC32.

## Performance Notes

- For weak cameras/displays, use `Reliable`.
- For strong cameras/high-brightness displays, try `Fast` or `Aggressive`.
- `Extreme` is best-effort/high-throughput and may require retry rounds.
- Keep QR centered, avoid glare, and keep camera in focus.
- Hard refresh both pages after pulling updates to avoid stale cached scripts.

## Local Usage

```sh
open generator.html
open scanner.html
```

## License

This project is licensed under the MIT License. See `LICENSE`.

## Acknowledgments

- [Vue.js](https://vuejs.org/)
- [pako](https://github.com/nodeca/pako)
- [qrcode.js](https://github.com/davidshimjs/qrcodejs)
- [zbar-wasm](https://github.com/undecaf/zbar-wasm)
