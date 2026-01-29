# TheBridge - Configuration Guide

## Wireless Configuration

Copy the config template and update credentials:

```cpp
#define THEBRIDGE_WIFI_SSID "YOUR_SSID"        // Your 2.4GHz Wi-Fi network
#define THEBRIDGE_WIFI_PASS "YOUR_PASSWORD"    // Wi-Fi password
```

`include/config.h` is gitignored; keep your local credentials there.

## USB Mounting & File Access

The device mounts the iPod's USB filesystem to `/usb` in the VFS. Key paths:
- iPod music database: `/usb/iPod_Control/Music/`
- iTunes metadata: `/usb/iPod_Control/iTunes/`
- Artwork: `/usb/iPod_Control/Artwork/`

## API Endpoints

See `API.md` for the full endpoint reference and examples.

## mDNS Discovery

The device advertises as:
- **Hostname:** `ipodbridge.local` (runtime identifier)
- **Service:** `_http._tcp`
- **Port:** 80

On macOS, accessible via:
```bash
ping ipodbridge.local
curl http://ipodbridge.local/status
```

## Memory-Conscious Design

This implementation prioritizes low memory usage:
- **HTTP stack size:** 6 KB
- **TCP/IP task:** 3 KB
- **Max open sockets:** 2
- **FAT allocation unit:** 4 KB
- **USB block size:** 512 bytes

## USB Device Requirements

The iPod Nano must support:
- USB Mass Storage Class (MSC)
- FAT32 filesystem
- 30-pin or Lightning connector (via adapter)
- 5V power when connected

## Compilation & Upload

```bash
cd /Users/harry/SyncPod
pio run -t upload -e seeed_xiao_esp32s3
pio device monitor -e seeed_xiao_esp32s3
```

## Troubleshooting

### USB Device Not Detected
1. Check USB connector contact
2. Verify 5V power is supplied to both ESP32-S3 and iPod
3. Monitor serial output for USB enumeration logs

### File Write Failures
1. Ensure iPod is not in "Do Not Disconnect" mode
2. Check available space on iPod
3. Verify `/usb/iPod_Control/Music/` directory exists

### Wi-Fi Connection Issues
1. Verify SSID/password are correct
2. Ensure 2.4GHz Wi-Fi (ESP32-S3 does not support 5GHz)
3. Check signal strength near device
