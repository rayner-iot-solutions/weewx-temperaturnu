# weewx-temperaturnu Changelog

All notable changes to this project will be documented in this file.

## [0.1] - 2026-06-04

### Initial Release

**Features:**
- Automatic temperature uploads to Temperatur.nu via HTTP GET
- Automatic temperature conversion to Celsius using `weewx.units.to_METRICWX()`
- Support for multiple unit systems (US, Metric, MetricWX)
- Temperature rounding to 1 decimal place
- Full Python 2.7+ and Python 3.x compatibility
- WeeWX v3.8.0 and later support
- Support for both WeeWX v4 (wee_extension) and v5 (weectl) installers
- Zero external dependencies - uses only Python standard library
- Comprehensive debug logging with API key masking for security
- Built-in self-test functionality with three unit system tests

**Technical Implementation:**
- Based on weewx-windy and weewx-temperaturnu templates
- Extends `weewx.restx.StdRESTbase` for REST API integration
- Implements thread-safe queue-based architecture
- Handles both modern (weeutil.logger) and legacy (syslog) logging

**Testing:**
- Includes unit tests for US, Metric, and MetricWX systems
- All three unit systems correctly convert 72.5°F / 22.5°C
- Manual testing via `python bin/user/temperaturnu.py`

**Credits:**
- RC Chuah (Author)
- Based on work by Matthew Wall, Jacques Terrettaz, and Konrad Skeri Ekblad
