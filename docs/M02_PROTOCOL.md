# Phomemo M02 protocol notes

## Current repository flow

The application is a single static file: `index.html`.

Current flow after this diagnostic iteration:

1. User taps `Conectar impresora`.
2. Web Bluetooth requests a device with M02-compatible optional services.
3. The app connects to GATT, lists every service and characteristic, and selects the write characteristic.
4. User taps `TEST — RASTER 1 VEZ`.
5. The app locks the print state from `idle` to `preparing`.
6. Diagnostic v26 rasterizes the textarea content, trims blank vertical rows, and caps it to one 24-line raster band.
7. The payload should fit in the initial credit window and should not wait for a second ready-refill.
8. Each write is sent once, with no retry of the full job.
9. On completion or error, the state returns to `idle`.

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

The app prefers `writeValueWithoutResponse` for `FF02` when available. If not available, it falls back to `writeValue`.

Diagnostic v15 tried `writeValue` with response because `FF02` advertises it, but the real Bluefy/M02 path was too slow: only 6 packets / 288 bytes were sent in more than 10 seconds before user abort. That makes `withResponse` impractical for this printer/browser combination.

Current diagnostic chunking:

- Chunk size: 180 bytes while flow-control testing is active.
- Delay between chunks: 30 ms.
- Raster block height: 24 lines.
- Full job retries: none.

Reason for the raster-band change: the first diagnostic capture showed one tap, one job, 137 BLE writes, 6545 bytes, and no extra writes after `END JOB #1`. If the printer physically repeated after that, the browser did not resend the job. The likely cause is printer-side interpretation of one oversized raster block.

Reason for keeping `withoutResponse`: the second diagnostic capture showed `withResponse` was not usable in Bluefy for this device. The next test keeps safe raster bands but returns to the fast write method.

Diagnostic v16 completed the whole job with `withoutResponse` and no JS resend, then the printer continued physically after `END JOB`. The important notification is:

```text
NOTIFY FF03: 01 07
```

Current interpretation: `FF03` is likely credit-based flow control. Sending 93 packets without consuming credits can overflow the device-side buffer and leave the firmware printing from its own queue, which the browser cannot abort after the fact.

Diagnostic v17 gates every BLE write behind `FF03` credits. If no new credit arrives, the app stops with a timeout instead of dumping the full raster into the printer. The regular print button is temporarily blocked; only the micro test is enabled until the flow-control behavior is confirmed.

Diagnostic v17 confirmed only the initial `+7` credits arrived. With 48-byte packets, that allowed only 336 bytes and the micro test stopped safely. Diagnostic v18 treats each credit as one BLE-packet credit rather than one raster-line credit. The 1169-byte micro test should fit in 7 writes at 180 bytes each while still respecting the initial credits.

Diagnostic v18 completed the 1169-byte micro test in 7 writes using the initial `+7` credits. Diagnostic v19 sends a two-band test and never lets one credit window cross into the next raster band. If no new `FF03` credits arrive after the first complete band, it stops before sending band 2.

Diagnostic v19 showed no new `FF03` credit frame after band 1, but `FF01` repeatedly sent `1A 0F 0C` after the first band. Diagnostic v20 treats that frame as "printer ready / band complete" and refills one last known credit window (`+7`) only while the app is actively waiting for the next segment.

Diagnostic v20 completed two 24-line bands: the second band started only after `FF01 1A 0F 0C` refilled the last known credit grant. Diagnostic v21 enables the normal print button again, but caps jobs at 120 lines until longer jobs are validated.

Diagnostic v22 changes the dedicated test button to print only the textarea content using the current font size and alignment. It does not add boxes, bars, labels, or any other test artwork. The text-only test still uses the same 24-line segments, BLE credit gate, and 120-line safety cap as normal safe printing.

Diagnostic v23 trims fully blank raster rows above and below the text-only test before sending. This keeps the test focused on visible text and avoids spending paper on empty vertical padding.

The real printer still produced 7 physical copies after the v23 text-only raster test, even though the browser log showed one job and no writes after `END JOB`. Diagnostic v24 blocks normal raster printing again and changes the test button to a short native ESC/POS text probe. It sends no `GS v 0` raster bands and does not use the `FF03: 01 07` value as a send-window refill. Diagnostic v25 adds the loaded app version to the copied diagnostic report and connection log to catch stale browser pages.

Diagnostic v25 confirmed native text writes are accepted by BLE but do not print on the M02. Diagnostic v26 returns to raster mode, but restricts the test to a single raster band so the job never crosses into a second `GS v 0` block or a ready-refill window.

Diagnostic v27 keeps the single-band raster test and exposes a write-channel selector. The printer advertises several writable characteristics (`FF02`, transparent UART, `FEC7`, and `FF82`). Testing the same payload across channels should show whether repeated physical output is tied to the `FF00 / FF02` path.

Diagnostic v27 showed `FF80 / FF82` accepts BLE writes but does not print raster data on this printer. Diagnostic v28 returns to `FF00 / FF02`, but uses BLE writes with response and a slower 120 ms inter-packet delay for the single-band raster probe.

These values are intentionally visible in the diagnostic panel. They are not treated as proof of correctness; the next real Bluefy log should confirm whether the printer accepts the sequence once and returns to ready state.

## Abort behavior

`Abort` only stops the local JavaScript send loop. It does not send `ESC @` as a fake cancel command anymore.

Reason: no source reviewed here confirms `ESC @` as a real cancel/reset-safe abort for an in-progress M02 job. If a real M02 cancel command is later identified from captures or documentation, it can be added explicitly.

## Sources used

- `vivier/phomemo-tools`: reverse-engineered M02 ESC/POS notes and GPLv3 implementation.
- `vivier/phomemo-tools/tools/phomemo-filter.py`: M02 raster packing/header/footer behavior. Reviewed for protocol only; code was not copied.
- `sgrankin/phomemo/PROTOCOL.md`: M02-family BLE UUIDs, write mode notes, raster format, and M02X caveats.
- Qiita article "Phomemo で 感熱紙に Hello World !": BLE characteristic listing for `ff01`, `ff02`, and `ff03`.
- `print_master_ble`: MIT-licensed Flutter package documentation describing `FF03` as credit-based flow control for Zhuhai Quin printers. Used as protocol evidence, not copied as code.

## Current uncertainty

We still need a real diagnostic copy from the physical M02 in Bluefy:

- Actual services and characteristics exposed by the selected printer.
- Whether `ff02` is selected.
- Whether the write method used is `withoutResponse` or `withResponse`.
- Exact count of BLE writes and bytes sent.
- Whether repeated physical printing happens after `END JOB`.
- Whether the printer sends notifications/status frames during or after the job.
- Whether `FF03` keeps sending credit frames while the printer drains data.
