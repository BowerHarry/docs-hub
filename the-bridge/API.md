# TheBridge API Reference

Base URL: `http://ipodbridge.local` (runtime identifier)

## Common Behavior

- All endpoints return JSON unless an error occurs.
- When the iPod is not mounted, endpoints return `503 Service Unavailable` with body `USB not mounted`.
- Paths can target any location under the `/usb` mount.
- Path validation rejects `..` and `\` to prevent traversal.
- Benchmark endpoints (`/sink`, `/fill`) are available only when `THEBRIDGE_ENABLE_BENCHMARKS=1`.

## GET /status

Returns device status and diagnostics.

**Example**
```bash
curl http://ipodbridge.local/status
```

**Response (example)**
```json
{"wifi":"connected","usb":"mounted", "...": "..."}
```

## POST /upload

Streams request body to a file on the iPod.

**Query params**
- `path` or `name`: file name or full path under `/usb`
  - Examples: `song.mp3` or `/usb/iPod_Control/Music/song.mp3` or `/usb/iPod_Control/Artwork/Album/cover.jpg`
- `overwrite` or `ow`: `1|0` or `true|false` (default: false)
- `mkdirs` or `mk`: `1|0` or `true|false` (default: false)
  - When true, parent directories are created if missing.

**Examples**
```bash
curl -X POST --data-binary @song.mp3 \
  "http://ipodbridge.local/upload?path=song.mp3&overwrite=0"
```

```bash
curl -X POST --data-binary @song.mp3 \
  "http://ipodbridge.local/upload?path=song.mp3&overwrite=1"
```

**Response (example)**
```json
{"status":"ok","file":"/usb/iPod_Control/Music/song.mp3","bytes":5242880,"overwritten":false}
```

**Errors**
- `400 Bad Request` – invalid or missing path
- `409 Conflict` – file exists (when overwrite is false) or path is a directory
- `500 Internal Server Error` – filesystem error

## GET /find

Checks whether a file exists and returns metadata.

**Query params**
- `path` or `name`

**Example**
```bash
curl "http://ipodbridge.local/find?path=song.mp3"
```

**Response (found)**
```json
{"exists":true,"path":"/usb/iPod_Control/Music/song.mp3","size":5242880,"is_dir":false}
```

**Response (missing)**
```json
{"exists":false,"path":"/usb/iPod_Control/Music/song.mp3"}
```

**Errors**
- `400 Bad Request` – invalid or missing path

## DELETE /delete

Deletes a file from the iPod.

**Query params**
- `path` or `name`

**Example**
```bash
curl -X DELETE "http://ipodbridge.local/delete?path=song.mp3"
```

**Response (example)**
```json
{"status":"ok","path":"/usb/iPod_Control/Music/song.mp3"}
```

**Errors**
- `400 Bad Request` – invalid path or path is a directory
- `404 Not Found` – file does not exist
- `500 Internal Server Error` – delete failed

## GET /device

Returns USB device identifiers and strings.

**Example**
```bash
curl "http://ipodbridge.local/device"
```

**Response (example)**
```json
{"vid":"0x05ac","pid":"0x1201","serial":"...","product":"iPod","manufacturer":"Apple"}
```

## POST /sink (benchmarks only)

Consumes request body without writing to disk. Use this to measure raw Wi‑Fi upload throughput.

**Example**
```bash
dd if=/dev/zero bs=1m count=32 | \
  curl -X POST --data-binary @- http://ipodbridge.local/sink
```

**Response (example)**
```json
{"status":"ok","bytes":33554432,"elapsed_ms":1234}
```

## POST /fill (benchmarks only)

Generates data on the ESP32 and writes it to the iPod. Use this to measure disk write throughput without Wi‑Fi as the bottleneck.

**Query params**
- `bytes` (required): number of bytes to write
- `path` or `name` (optional): path under `/usb`
- `overwrite` or `ow` (optional): `1|0` or `true|false`
- `mkdirs` or `mk` (optional): `1|0` or `true|false`

**Example**
```bash
curl -X POST "http://ipodbridge.local/fill?bytes=33554432&path=iPod_Control/Music/fill_test.bin&overwrite=1"
```

**Response (example)**
```json
{"status":"ok","bytes":33554432,"elapsed_ms":5678,"file":"/usb/iPod_Control/Music/fill_test.bin"}
```
