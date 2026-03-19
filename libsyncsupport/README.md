# libsyncsupport

Low-level Swift sync engine for iPod-style devices. Provides database writing, media placement, artwork generation, and optional transcoding. Designed to be transport-agnostic and usable by multiple applications.

## Features
- Write iTunesDB and ArtworkDB for classic iPod models.
- Place media files into F00–Fxx folders with colon path normalization.
- Generate/maintain a sync manifest to avoid unnecessary copies.
- Optional AAC/MP3 transcoding and artwork resizing.

## Requirements
- macOS 13+
- Swift 5.9+
- **ffmpeg** (recommended for MP3 transcoding): install with Homebrew (`brew install ffmpeg`). The sync engine prefers `ffmpeg` + `libmp3lame` for AAC/M4A → MP3; it may fall back to `afconvert` when `ffmpeg` is unavailable, but `ffmpeg` is the reliable path on modern macOS for many library sources.

## Usage
This package exposes `SyncService`, `SyncDevice`, and `SyncTrack` in the `LibSyncSupport` module.

```
import LibSyncSupport

let device = SyncDevice(
    name: "iPod",
    modelName: "iPod Nano 2nd Gen",
    mountPath: URL(fileURLWithPath: "/Volumes/iPod"),
    databaseType: .itunesDB
)

let service = SyncService()
try await service.sync(device: device, tracks: tracks)
```

## Compatibility Matrix

Status legend:
- ✅ **Tested** — physically verified on hardware
- 🔵 **Expected** — code implemented, same database format as a tested device
- 🟡 **Implemented** — code exists but untested on hardware
- ❌ **Not supported**

### iPod nano

| Model | Status | DB Format | Checksum | Notes |
| --- | --- | --- | --- | --- |
| 1st Gen | ✅ **Tested** | iTunesDB | None | MP3-only playback; AAC/M4A sources are transcoded to MP3 automatically. **ffmpeg** recommended for reliable transcoding on many systems. |
| 2nd Gen | ✅ **Tested** | iTunesDB | None | Physically verified working |
| 3rd Gen | ✅ **Tested** | iTunesDB | Hash58 | Primary development target |
| 4th Gen | 🔵 Expected | iTunesDB | Hash58 | Identical DB format to 3rd Gen |
| 5th Gen | 🔵 Expected | iTunesDB | Hash72 | Hash72 implemented, same DB structure |
| 6th Gen | 🟡 Implemented | HashedDB | HashAB | Different DB format (multitouch) |
| 7th Gen | 🟡 Implemented | HashedDB | HashAB | Different DB format (multitouch) |

### iPod Classic

| Model | Status | DB Format | Checksum | Notes |
| --- | --- | --- | --- | --- |
| 1st–5.5 Gen | ❌ | — | — | Older DB format, no hash |
| 6th Gen (80GB/160GB) | 🔵 Expected | iTunesDB | Hash58 | Same format as Nano 3rd/4th Gen |
| 7th Gen (120GB/160GB Thin) | 🔵 Expected | iTunesDB | Hash58 | Same format as Nano 3rd/4th Gen |

### iPod shuffle

| Model | Status | DB Format | Notes |
| --- | --- | --- | --- |
| 1st–4th Gen | ❌ | iTunesSD | Different format entirely |

### iPod mini

| Model | Status | Notes |
| --- | --- | --- |
| 1st/2nd Gen | ❌ | Older DB format |

## Notes
This library intentionally excludes device discovery and eject/mount logic. Host applications are responsible for identifying devices and providing a mount path.

