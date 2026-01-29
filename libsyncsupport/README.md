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

Status legend: ✅ supported, ⏳ in progress, ❌ not supported.

| iPod Family | Model | Status | Notes |
| --- | --- | --- | --- |
| iPod mini | 1st Gen | ❌ | |
| iPod mini | 2nd Gen | ❌ | |
| iPod classic | 1st Gen | ❌ | |
| iPod classic | 2nd Gen | ❌ | |
| iPod classic | 3rd Gen | ❌ | |
| iPod classic | 4th Gen (Mono) | ❌ | |
| iPod classic | 4th Gen (Color/Photo) | ❌ | |
| iPod classic | 5th Gen (Video) | ❌ | |
| iPod classic | 5.5 Gen (Video) | ❌ | |
| iPod classic | 6th Gen | ❌ | |
| iPod classic | 7th Gen | ❌ | |
| iPod shuffle | 1st Gen | ❌ | |
| iPod shuffle | 2nd Gen | ❌ | |
| iPod shuffle | 3rd Gen | ❌ | |
| iPod shuffle | 4th Gen | ❌ | |
| iPod nano | 1st Gen | ❌ | |
| iPod nano | 2nd Gen | ✅ | |
| iPod nano | 3rd Gen | ❌ | |
| iPod nano | 4th Gen | ❌ | |
| iPod nano | 5th Gen | ❌ | |
| iPod nano | 6th Gen | ❌ | |
| iPod nano | 7th Gen | ❌ | |

## Notes
This library intentionally excludes device discovery and eject/mount logic. Host applications are responsible for identifying devices and providing a mount path.

