# ClickWheel

ClickWheel is a macOS app for managing and syncing music to legacy iPods (Classic, Nano, Shuffle). It reads your Music library, builds an iTunesDB on-device, and supports both wired sync and optional wireless sync through an ESP32 bridge.

## Highlights

- USB iPod detection using VID/PID (no SysInfo parsing)
- Per-device profiles for selecting artists, albums, and playlists
- Smart sync with a local manifest to avoid unnecessary file copies
- Optional wireless sync via ESP32 bridge API

## Requirements

- macOS 13 or later
- Xcode 15 or later
- Access to the Music app library (iTunesLibrary framework)

## Repository layout

This app relies on two local Swift packages:

- `libsyncsupport` – low-level sync engine and iTunesDB writing
- `libpodbridgesupport` – ESP32 bridge discovery + API client

Expected local paths (can be adjusted in Xcode if needed):

```
/Users/harry/Github/libsyncsupport
/Users/harry/Github/libpodbridgesupport
```

## Build

1. Open `ClickWheel.xcodeproj` in Xcode.
2. Ensure both local packages resolve.
3. Build and run (⌘B, ⌘R).

## Usage

1. Connect your iPod via USB or ensure it is connected to the ESP32 bridge.
2. Add the device when prompted.
3. Choose or create a sync profile and select content to sync.
4. Click **Sync Now**.
5. Eject wired devices when finished.

## Wireless sync (ESP32 bridge)

ClickWheel discovers the bridge over Bonjour (`_ipodbridge._tcp`) and polls for mounted USB devices. The ESP32 must expose these endpoints:

- `GET /status`
- `GET /device`
- `GET /find?path=...`
- `POST /upload?path=...`
- `DELETE /delete?path=...`

## Architecture overview

- `DeviceManager` – USB detection, device state, profiles, connection status
- `MusicLibraryManager` – Music app library ingestion
- `LibSyncSupport` – sync engine and iTunesDB writer
- `LibPodBridgeSupport` – bridge discovery, polling, and API client
- `WirelessSyncService` – stages files locally and uploads via bridge

## Notes

- ClickWheel does not modify your Music library.
- Sync operations are staged and written atomically to minimize corruption risk.

## License

Provided as-is for educational and personal use.

