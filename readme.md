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
- Interleaved chunk repeats (`A B C ... A B C ...`) to reduce burst-loss impact
- Retry backoff for high-loss rounds (slower but more robust retries when needed)
- Chunk safety margin to avoid operating at fragile QR capacity limits
- Large-transfer adaptive initial repeat boosting to avoid early-stall progress
- Generator auto-fits and centers QR in viewport when transfer starts
- Scanner stall watchdog with soft/hard auto-recovery
- Metadata heartbeat injection during long sender passes
- Simplified retry code format (`R2`) with backward-compatible sender parsing (`R1` still accepted)

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
Current defaults are tuned for higher-resolution cameras with larger chunk targets on high-performance presets.

### Edit Presets in Source

Presets are defined in `generator.html` under:
- `SENDER_PRESETS`
- `DEFAULT_PRESET_NAME`

Update these objects to customize profile behavior for your hardware.

Common preset knobs:
- `initial_chunk_size`, `min_chunk_size`: payload sizing bounds
- `chunk_safety_ratio`: keeps effective chunk payload below edge capacity for better decode reliability
- `frame_interval_ms`, `start_delay_ms`: pacing controls
- `metadata_repeat_count`, `initial_pass_repeat_count`, `retry_repeat_count`: baseline redundancy
- `metadata_heartbeat_interval`: inject metadata frame every N data packets to help scanner re-lock mid-pass
- `large_transfer_chunk_threshold`, `large_transfer_repeat_boost`: add initial-pass repeats when chunk count is high
- `very_large_transfer_chunk_threshold`, `very_large_transfer_repeat_boost`: add more repeats for very large transfers
- `max_initial_pass_repeat_count`: cap auto-boosted initial-pass repeats
- `retry_high_loss_threshold`: missing-ratio trigger for safer retry mode
- `retry_high_loss_frame_multiplier`, `retry_high_loss_start_delay_multiplier`: retry pacing backoff multipliers
- `retry_high_loss_metadata_extra`, `retry_high_loss_repeat_increment`: extra redundancy added only for high-loss retries

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
3. Receiver generates compact retry code (`R2:session:ranges:crc`) for missing chunk ranges.
4. User pastes retry code into sender.
5. Sender validates retry-code checksum and session, then resends only requested chunks.
6. If missing ratio is high, sender auto-applies retry backoff (slower cadence + extra redundancy) for that retry round.
7. After reconstruction, receiver computes SHA-256 and compares with sender SHA-256.

For large transfers, sender may also automatically increase initial interleaved repeat rounds based on total chunk count.

If SHA-256 mismatches, receiver requests retransmission.

## Packet Format (Current)

- Metadata packet: `M:...`
- Data packet: `D:...`
- Retry code: `R2:...` (sender accepts legacy `R1:...` too)

Metadata includes session and file integrity info.
Data packet includes chunk index and CRC32.

## Performance Notes

- For weak cameras/displays, use `Reliable`.
- For strong cameras/high-brightness displays, try `Fast` or `Aggressive`.
- `Extreme` is best-effort/high-throughput and may require retry rounds.
- If `Extreme` shows notable loss, rely on retry backoff or reduce `chunk_safety_ratio`/increase `frame_interval_ms` in preset tuning.
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
