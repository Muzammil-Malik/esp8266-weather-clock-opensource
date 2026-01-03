# Project Structure

This document describes the organization of the ESP8266 Weather Clock firmware repository.

## Directory Layout

```
esp8266-weather-clock-opensource/
│
├── README.md                      # Main documentation (start here!)
├── LICENSE                        # MIT License
├── CHANGELOG.md                   # Version history
├── CONTRIBUTING.md                # Contribution guidelines
├── PROJECT_STRUCTURE.md           # This file
├── .gitignore                     # Git exclusions
│
├── src/                           # Source code
│   └── clock_ntp_ota_v1.9.ino     # Main firmware (2,096 lines)
│
├── docs/                          # Documentation
│   ├── INSTALLATION.md            # Complete installation guide
│   ├── HARDWARE.md                # Hardware specs and pinout
│   ├── v1.9_RELEASE_NOTES.md      # v1.9.0 release notes
│   └── v1.9.1_HYBRID_FIX.md       # v1.9.1 WiFi startup fix
│
├── images/                        # Photos and screenshots
│   ├── product/                   # AliExpress product photos
│   │   ├── 01-main-product.webp   # Main product shot
│   │   ├── 02-components.webp     # Kit components
│   │   ├── 03-weather-forecast.webp
│   │   ├── 04-temperature-display.webp
│   │   ├── 05-details.webp        # Transparent case details
│   │   └── 06-size.webp           # Dimensions (40x40x43mm)
│   │
│   └── build/                     # Custom firmware screenshots
│       ├── display-time.png       # Time display mode
│       ├── display-temperature.png # Weather display mode
│       └── display-sunrise-sunset.png # Solar display mode
│
└── .github/                       # GitHub-specific files
    ├── workflows/
    │   └── build.yml              # CI: Auto-build on push
    │
    └── ISSUE_TEMPLATE/
        ├── bug_report.md          # Bug report template
        └── feature_request.md     # Feature request template
```

## Key Files

### Root Level

**README.md** (11KB)
- Main project documentation
- Blog-style narrative about reverse engineering
- Security issues discovered
- Complete feature list
- Installation quickstart
- API documentation

**LICENSE** (MIT)
- Permissive open source license
- Use freely, modify, distribute

**CHANGELOG.md**
- Version history: v1.5 → v1.9.1
- Features, fixes, breaking changes
- Migration notes

**CONTRIBUTING.md**
- How to contribute
- Code style guidelines
- Testing requirements

### Source Code (`/src`)

**clock_ntp_ota_v1.9.ino**
- Main firmware file (2,096 lines)
- ESP8266 Arduino sketch
- Requires libraries:
  - Adafruit GFX & SSD1306
  - NTPClient
  - WiFiManager
  - AsyncHTTPRequest_Generic
  - ESPAsyncTCP

**Architecture:**
- Fully async (zero blocking in loop)
- State machines: WiFi, NTP, Weather
- Hybrid model: sync WiFi in setup(), async in loop()
- Memory-optimized: ICACHE_FLASH_ATTR on 26 functions

**Configuration:**
- 26-field struct stored in EEPROM
- Magic number validation
- Web-based config UI

### Documentation (`/docs`)

**INSTALLATION.md** (18KB)
- Complete step-by-step installation guide
- Arduino IDE setup
- FTDI wiring diagrams
- OTA update instructions
- Comprehensive troubleshooting

**HARDWARE.md** (8KB)
- ESP-01S specifications
- Pin mapping (SDA=GPIO0, SCL=GPIO2)
- Display module details (GM009605v4.3)
- Power requirements
- Memory layout
- Safety warnings

**v1.9_RELEASE_NOTES.md** (7KB)
- Detailed v1.9.0 changelog
- Performance improvements
- Async architecture explanation
- Memory usage comparison
- Testing checklist

**v1.9.1_HYBRID_FIX.md** (Russian, 6KB)
- Critical startup fix documentation
- WiFi synchronous vs async tradeoffs
- Timeline diagrams
- Before/after comparison

### Images (`/images`)

**Product Photos** (`/product`)
- Original AliExpress product images
- DIY kit components
- Transparent acrylic case
- Size reference (40mm cube)

**Build Photos** (`/build`)
- Custom firmware screenshots
- Three display modes:
  1. Time mode (10:34 + date)
  2. Weather mode (15.4°c + city)
  3. Sunrise/sunset mode (times + daylight duration)

### GitHub Config (`/.github`)

**Workflows**
- `build.yml`: CI pipeline
  - Auto-compile on push
  - Check firmware size < 470KB
  - Upload build artifacts
  - Attach binaries to releases

**Issue Templates**
- `bug_report.md`: Structured bug reports
- `feature_request.md`: Feature suggestions

## Build Artifacts (ignored by git)

When you compile locally, these are created:

```
build/
├── clock_ntp_ota_v1.9.ino.bin     # Flash this via OTA
├── clock_ntp_ota_v1.9.ino.elf     # Debug symbols
└── clock_ntp_ota_v1.9.ino.map     # Memory map
```

**Note**: `build/` is in `.gitignore` - artifacts not committed to repo.

## File Sizes

| File | Size | Description |
|------|------|-------------|
| `src/*.ino` | 65KB | Main source code |
| `build/*.bin` | 409KB | Compiled firmware |
| `README.md` | 45KB | Main docs |
| `docs/INSTALLATION.md` | 18KB | Install guide |
| `docs/HARDWARE.md` | 8KB | Hardware specs |

## Memory Usage

**Compiled firmware (v1.9.1):**
- Flash: 408,844 / 1,048,576 bytes (38%)
- RAM: 37,644 / 80,192 bytes (46%)
- IRAM: 61,987 / 65,536 bytes (94%) ⚠️

**Why 94% IRAM is acceptable:**
- ICACHE_FLASH_ATTR applied to all web handlers
- Stable across versions v1.8-v1.9.1
- No IRAM growth observed in testing

## Version Control

**Branches:**
- `main`: Stable releases (v1.9.1)
- `develop`: Work-in-progress features
- `feature/*`: New feature branches

**Tags:**
- `v1.9.1`: Current production release
- `v1.9.0`: Async refactoring
- `v1.8.0`: Security + stability fixes
- `v1.7.0`: Initial working firmware

## Not Included (Why)

**What's NOT in this repo:**
- Build artifacts (`.bin`, `.elf`, `.map`) - generated locally
- Backup files (`.bak`, `.bak2`) - development artifacts
- IDE configs (`.vscode/`, `.idea/`) - personal preferences
- macOS metadata (`.DS_Store`) - system files
- Secrets (`config_local.h`) - would leak credentials

These are excluded via `.gitignore`.

## How to Navigate

**For users:**
1. Start with `README.md` (overview + quickstart)
2. Follow `docs/INSTALLATION.md` (step-by-step setup)
3. Check `CHANGELOG.md` (version history)

**For developers:**
1. Read `CONTRIBUTING.md` (guidelines)
2. Study `src/clock_ntp_ota_v1.9.ino` (source code)
3. Review `docs/HARDWARE.md` (hardware constraints)
4. Check `.github/workflows/build.yml` (CI setup)

**For hardware hackers:**
1. Check `docs/HARDWARE.md` (pinout, specs)
2. View `images/product/` (original device photos)
3. Read `README.md` section "Hardware Discovery"

**For troubleshooters:**
1. Open `docs/INSTALLATION.md`
2. Jump to "Troubleshooting" section
3. Check `images/build/` for reference screenshots

## Quick Links

- **Main docs**: [README.md](README.md)
- **Install guide**: [docs/INSTALLATION.md](docs/INSTALLATION.md)
- **Hardware specs**: [docs/HARDWARE.md](docs/HARDWARE.md)
- **Changelog**: [CHANGELOG.md](CHANGELOG.md)
- **Contributing**: [CONTRIBUTING.md](CONTRIBUTING.md)

---

**Last updated**: 2026-01-03
**Repository**: https://github.com/your-username/esp8266-weather-clock-opensource
