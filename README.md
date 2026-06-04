# weewx-temperaturnu t

A lightweight WeeWX extension that automatically uploads weather station temperature data to [Temperatur.nu](https://www.temperatur.nu/).

![License](https://img.shields.io/badge/License-GPLv3-blue.svg)
![Python](https://img.shields.io/badge/Python-2.7%2B%20%7C%203.x-blue.svg)
![WeeWX](https://img.shields.io/badge/WeeWX-3.8.0%2B-blue.svg)

## Overview

**weewx-temperaturnu** is a WeeWX extension that automatically monitors archive records from your weather station and uploads temperature data to [Temperatur.nu](https://www.temperatur.nu/), a Swedish weather data collection service.

The extension performs automatic temperature conversion to Celsius regardless of your station's unit system, ensuring compatibility with stations using Fahrenheit, Celsius, or other unit conventions.

## Features

- ✅ **Automatic Temperature Uploads** - Sends temperature to Temperatur.nu for each new archive record
- ✅ **Intelligent Unit Conversion** - Automatically converts temperature to Celsius using WeeWX's built-in `weewx.units.to_METRICWX()` helper
- ✅ **Universal Compatibility** - Works with US, Metric, and MetricWX unit systems
- ✅ **Zero Dependencies** - Uses only Python standard library (no external pip packages required)
- ✅ **Python 2 & 3 Compatible** - Supports Python 2.7+ and Python 3.x
- ✅ **WeeWX v3.8.0+** - Compatible with all modern WeeWX versions
- ✅ **Flexible Installation** - Support for both WeeWX v4 (wee_extension) and v5 (weectl) installers
- ✅ **Secure Debug Logging** - Comprehensive logging with automatic API key masking
- ✅ **Lightweight** - Minimal code footprint with simple, maintainable design

## Requirements

- **Python:** 2.7+ or 3.x
- **WeeWX:** v3.8.0 or later
- **API Key:** Obtain from [Temperatur.nu](https://www.temperatur.nu/nystation/)
- **External Libraries:** None (uses only Python standard library)

## Installation

### Step 1: Obtain an API Key

Visit [Temperatur.nu New Station](https://www.temperatur.nu/nystation/) to register your weather station and receive an API key.

### Step 2: Download the Extension

```bash
wget -O weewx-temperaturnu.zip https://github.com/rc-chuah/weewx-temperaturnu/releases/latest/download/weewx-temperaturnu.zip
```

### Step 3: Install

**For WeeWX v4 and earlier:**

```bash
wee_extension --install weewx-temperaturnu.zip
```

**For WeeWX v5:**

```bash
weectl extension install weewx-temperaturnu.zip
```

### Step 4: Configure

Edit `/etc/weewx/weewx.conf` and add the required configuration:

```ini
[StdRESTful]
    [[TemperaturNu]]
        apikey = YOUR_API_KEY_HERE
```

**Optional parameters:**

```ini
[StdRESTful]
    [[TemperaturNu]]
        # Your API key from temperatur.nu (REQUIRED)
        apikey = YOUR_API_KEY_HERE
        
        # Enable or disable extension (default: true)
        enabled = true
        
        # Temperatur.nu server URL (default: https://www.temperatur.nu/rapportera.php)
        server_url = https://www.temperatur.nu/rapportera.php
        
        # Upload interval in seconds (default: 60)
        post_interval = 60
        
        # Maximum backlog of records to upload (default: unlimited)
        max_backlog = 10
        
        # Data upload timeout in seconds (default: 60)
        timeout = 60
        
        # Maximum upload retry attempts (default: 3)
        max_tries = 3
```

### Step 5: Restart WeeWX

```bash
sudo systemctl restart weewx
```

Check that the extension loaded successfully:

```bash
tail -f /var/log/syslog | grep -i temperaturnu
```

Expected log output:

```
temperaturnu: version is 0.1
temperaturnu: Data will be uploaded to https://www.temperatur.nu/rapportera.php
```

## How It Works

The extension operates through the following workflow:

1. **Monitor** - Listens for new archive records from your WeeWX weather station
2. **Extract** - Retrieves the outdoor temperature field (`outTemp`) from each record
3. **Convert** - Transforms temperature to Celsius using WeeWX's unit conversion system
4. **Upload** - Sends the temperature to Temperatur.nu via an HTTP GET request

### Temperature Conversion

The extension automatically converts your weather station's temperature to Celsius, regardless of the unit system your station uses. This is accomplished through WeeWX's `weewx.units.to_METRICWX()` helper function, which intelligently handles:

- **US Units** (°F) → Celsius
- **Metric Units** (°C) → Celsius (pass-through)
- **MetricWX Units** (°C) → Celsius (pass-through)

#### Example Conversions

| Fahrenheit | Celsius | Context |
|-----------|---------|---------|
| 32°F      | 0°C     | Freezing point |
| 50°F      | 10°C    | Cool day |
| 68°F      | 20°C    | Room temperature |
| 72.5°F    | 22.5°C  | Comfortable temperature |
| 98.6°F    | 37°C    | Body temperature |
| 212°F     | 100°C   | Boiling point |

### Upload Protocol

The extension uploads temperature data to Temperatur.nu using HTTP GET with the following format:

```
GET /rapportera.php?hash=YOUR_API_KEY&t=22.5 HTTP/1.1
Host: www.temperatur.nu
```

**Parameters:**
- `hash` - Your Temperatur.nu API key
- `t` - Temperature in Celsius (rounded to 1 decimal place)

## Troubleshooting

### View Extension Logs

Check WeeWX logs for temperaturnu messages:

```bash
tail -f /var/log/syslog | grep temperaturnu
```

Or on systemd systems:

```bash
journalctl -u weewx -f | grep temperaturnu
```

### Enable Debug Logging

To see detailed debug information, enable debug mode in `/etc/weewx/weewx.conf`:

```ini
debug = 2
```

Then restart WeeWX:

```bash
sudo systemctl restart weewx
```

Debug logs will show:
- Temperature conversion values
- Upload URLs (with API key masked)
- HTTP request details
- Success/failure status

### Test the Extension Manually

Run the built-in test suite to verify correct temperature conversion:

```bash
cd /usr/share/weewx
PYTHONPATH=bin python bin/user/temperaturnu.py
```

This executes three unit system tests. Expected output:

```
================================================================================
Test 1 - Purely US Units (weewx.US)
Input: US units (°F)
================================================================================
https://www.temperatur.nu/rapportera.php?hash=test_key_12345&t=22.5

================================================================================
Test 2 - Purely Metric Units (weewx.METRIC)
Input: Metric units (°C)
================================================================================
https://www.temperatur.nu/rapportera.php?hash=test_key_12345&t=22.5

================================================================================
Test 3 - Purely MetricWX Units (weewx.METRICWX)
Input: MetricWX units (°C)
================================================================================
https://www.temperatur.nu/rapportera.php?hash=test_key_12345&t=22.5

================================================================================
EXPECTED OUTPUTS (for all three unit tests - should be identical):
================================================================================
temperature=22.5 (°C)
================================================================================
```

All three tests should produce identical results (22.5°C), confirming the conversion logic works correctly.

### Test Manually with curl

```bash
curl "https://www.temperatur.nu/rapportera.php?hash=YOUR_API_KEY&t=20"
```

### Common Issues and Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| **No uploads occurring** | API key missing or incorrect | Verify `apikey` in `/etc/weewx/weewx.conf` matches your Temperatur.nu account |
| **"No outTemp found in record"** | Temperature field missing | Ensure your weather station records the `outTemp` field (most stations do by default) |
| **Connection errors** | Network or firewall issue | Test connectivity: `curl -v https://www.temperatur.nu/rapportera.php?hash=test&t=20` |
| **Wrong temperature value** | Incorrect unit system | Check `/etc/weewx/weewx.conf` for correct `unit_system` setting (US, Metric, or MetricWX) |
| **Extension not loading** | Python path or import error | Verify WeeWX installation; check logs with `debug = 2` enabled |

## Dependencies

This extension uses **only Python's standard library**. No external packages are required.

**Python modules used:**
- `Queue` (Python 2) / `queue` (Python 3) - Thread-safe queue
- `urllib` (Python 2) / `urllib.parse` (Python 3) - URL encoding
- `sys`, `time` - System utilities
- `logging` / `syslog` - Logging (with fallback for legacy WeeWX versions)

**WeeWX modules used:**
- `weewx.restx` - REST API framework
- `weewx.units` - Temperature unit conversion
- `weewx.manager` - Managing WeeWX database
- `weeutil.weeutil` - Utility functions

## Architecture

The extension uses a thread-safe queue-based architecture inherited from WeeWX's RESTful framework:

1. **Main Thread** - Captures new archive records and queues them
2. **Background Thread** - Processes queued records and uploads to Temperatur.nu
3. **Queue** - Thread-safe FIFO buffer for decoupling data capture from uploads

This design ensures that temperature uploads don't block WeeWX's main weather station loops.

## License

Copyright © 2026 RC Chuah

Distributed under the terms of the [GNU General Public License (GPLv3)](LICENSE.md)

## Credits

- **Original Concept & Implementation:** Based on [weewx-windy](https://github.com/Jterrettaz/weewx-windy) by Jacques Terrettaz and [weewx-temperaturnu](https://github.com/LapplandsCohan/weewx-temperaturnu) by Konrad Skeri Ekblad
- **Modified for Temperatur.nu Integration:** RC Chuah

## Related Projects

- **WeeWX:** https://www.weewx.com/
- **Temperatur.nu:** https://www.temperatur.nu/
- **weewx-windy:** https://github.com/Jterrettaz/weewx-windy
- **weewx-temperaturnu (original):** https://github.com/LapplandsCohan/weewx-temperaturnu

## Documentation

- **WeeWX Docs:** https://www.weewx.com/docs/
- **WeeWX Extensions:** https://www.weewx.com/docs/utilities/weectl.htm
- **Temperatur.nu API:** https://www.temperatur.nu/info/rapportera-till-temperatur-nu/
