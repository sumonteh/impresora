# Phomemo M02 protocol notes

## Current repository flow

The application is a single static file: `index.html`.

Current flow after this diagnostic iteration:

1. User taps `Conectar impresora`.
2. Web Bluetooth requests a device with M02-compatible optional services.
3. The app connects to GATT, lists every service and characteristic, and selects the write characteristic.
4. User taps `TEST M02 — UNA IMPRESIÓN` or `Imprimir 1 copia`.
5. The app locks the print state from `idle` to `preparing`.
6. A 384 px wide monochrome raster is packed as 1 bpp, MSB first, with `1 = black`.
7. A single byte stream is built.
8. The stream is fragmented into 48 byte BLE writes.
9. Each write is sent once, with no retry of the full job.
10. On completion or error, the state returns to `idle`.

## BLE service and characteristics

Known M02-family UUIDs from public reverse engineering:

- Service: `0000ff00-0000-1000-8000-00805f9b34fb`
- Read/status: `0000ff01-0000-1000-8000-00805f9b34fb`
- Write: `0000ff02-0000-1000-8000-00805f9b34fb`
- Notify/unknown: `0000ff03-0000-1000-8000-00805f9b34fb`

The app now records the actual services and characteristics exposed by the real printer in the `Diagnóstico M02` panel. That real capture should be treated as more important than assumptions in code.

Real Bluefy capture from `Mr.in_M02` exposed short UUID strings:

- `FF00 / FF02`: write and writeWithoutResponse.
- `FF00 / FF01`: notify.
- `FF00 / FF03`: notify.
- `180A`: device information characteristics.
- Transparent UART service `49535343-FE7D-4AE5-8FA9-9FAFD205E455`.
- `FEE7` service.
- `FF80 / FF82`: write and writeWithoutResponse.

The application now normalizes short and long UUID forms before selecting `FF02`.

## Raster format

- Width: 384 px.
- Bytes per line: 48.
- Bit order: MSB first across each row.
- Pixel meaning: `1 = black`, `0 = white`.
- Raster command: `GS v 0`.
- Width and line counts are little-endian.
- Diagnostic v14 sent a single 136-line `GS v 0` block. The app sent it only once, but this is still unsafe for this real M02 firmware.
- Diagnostic v15 splits raster data into 24-line blocks to avoid printer-side buffer replay.

Raster block header:

```text
1D 76 30 00 30 00 LL LH
```

For M02 width, `30 00` means 48 bytes per line. `LL LH` is the number of lines in this block.

## Current job bytes

The diagnostic job stream is:

```text
1B 40
1F 11 02 04
1D 76 30 00 30 00 <lines-low> <lines-high>
<48 bytes per raster line, at most 24 lines per block>
1B 64 04
```

The app initializes with `ESC @` and the M02-specific `1F 11 02 04` bytes that several public implementations identify as part of M02 initialization.

## Write method and chunking

The app now prefers `writeValue` with response when the selected characteristic advertises it. If not available, it falls back to `writeValueWithoutResponse`.

Current diagnostic chunking:

- Chunk size: 48 bytes.
- Delay between chunks: 30 ms.
- Raster block height: 24 lines.
- Full job retries: none.

Reason for the change: the first diagnostic capture showed one tap, one job, 137 BLE writes, 6545 bytes, and no extra writes after `END JOB #1`. If the printer physically repeated after that, the browser did not resend the job. The likely causes are printer-side interpretation of one oversized raster block, write-without-response buffering, or both.

These values are intentionally visible in the diagnostic panel. They are not treated as proof of correctness; the next real Bluefy log should confirm whether the printer accepts the sequence once and returns to ready state.

## Abort behavior

`Abort` only stops the local JavaScript send loop. It does not send `ESC @` as a fake cancel command anymore.

Reason: no source reviewed here confirms `ESC @` as a real cancel/reset-safe abort for an in-progress M02 job. If a real M02 cancel command is later identified from captures or documentation, it can be added explicitly.

## Sources used

- `vivier/phomemo-tools`: reverse-engineered M02 ESC/POS notes and GPLv3 implementation.
- `vivier/phomemo-tools/tools/phomemo-filter.py`: M02 raster packing/header/footer behavior. Reviewed for protocol only; code was not copied.
- `sgrankin/phomemo/PROTOCOL.md`: M02-family BLE UUIDs, write mode notes, raster format, and M02X caveats.
- Qiita article "Phomemo で 感熱紙に Hello World !": BLE characteristic listing for `ff01`, `ff02`, and `ff03`.

## Current uncertainty

We still need a real diagnostic copy from the physical M02 in Bluefy:

- Actual services and characteristics exposed by the selected printer.
- Whether `ff02` is selected.
- Whether the write method used is `withoutResponse` or `withResponse`.
- Exact count of BLE writes and bytes sent.
- Whether repeated physical printing happens after `END JOB`.
- Whether the printer sends notifications/status frames during or after the job.
