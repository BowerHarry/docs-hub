# LibPodBridgeSupport

LibPodBridgeSupport is a small Swift package that encapsulates ESP32 iPod bridge discovery and API access. It provides Bonjour/mDNS resolution, polling orchestration, and a typed HTTP client so host apps can respond to “a wireless device is connected” without knowing network details.

## Features

- Bonjour discovery for `_ipodbridge._tcp`
- ESP32 API client (`/status`, `/device`, `/find`, `/upload`, `/delete`)
- Polling orchestration with simple callbacks
- Host override support (e.g. fixed IP on a local network)

## Usage

```swift
import LibPodBridgeSupport

var clientConfig = BridgeClientConfiguration()
clientConfig.baseURL = URL(string: "http://ipodbridge.local")!
clientConfig.logger = { print("🌐 \($0)") }

let monitorConfig = BridgeMonitorConfiguration(
    pollInterval: 5.0,
    clientConfiguration: clientConfig,
    callbackQueue: .main,
    logger: { print("📡 \($0)") }
)

let monitor = BridgeMonitor(configuration: monitorConfig)
monitor.onUpdate = { state in
    // state.isReachable, state.isUsbMounted, state.identity
}
monitor.start()
```

Upload usage (typically after a staging step):

```swift
try await monitor.client.upload(
    localURL: fileURL,
    remotePath: "/usb/iPod_Control/Music/F00/song.m4a",
    overwrite: true,
    mkdirs: true
)
```

## Public API summary

- `BridgeClient` for ESP32 API calls and retries
- `BridgeDiscovery` for Bonjour/mDNS resolution
- `BridgeMonitor` for polling orchestration and state callbacks
- `BridgeDeviceIdentity`, `BridgeStatus`, `BridgeConnectionState`

## Notes and future direction

The package is intentionally independent of any sync engine. Host apps decide what to do after a bridge reports a mounted device. A future extension will add BLE-based proximity detection and Wi-Fi provisioning, but that is not implemented here.

