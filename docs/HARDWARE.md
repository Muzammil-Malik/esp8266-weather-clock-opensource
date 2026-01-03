# Hardware Documentation

## Device Specifications

### Original Product
- **Name**: ESP8266 Mini Weather Clock Kit
- **Model**: TJ-56-654
- **Source**: [AliExpress Link](https://pt.aliexpress.com/item/1005008333782531.html)
- **Price**: ~$12 USD
- **Dimensions**: 40mm x 40mm x 43mm

### Components

#### ESP-01S WiFi Module
- **Chip**: ESP8266EX
- **Flash**: 1MB (8Mbit)
- **RAM**: 80KB total (32KB instruction, 48KB data)
- **CPU**: 80MHz (can be overclocked to 160MHz)
- **WiFi**: 802.11 b/g/n (2.4GHz only)
- **GPIO**: 2 usable pins (GPIO0, GPIO2)
- **Voltage**: 3.3V (NOT 5V tolerant!)

#### Display Module
- **Model**: GM009605v4.3
- **Type**: OLED (Organic LED)
- **Resolution**: 128x64 pixels
- **Size**: 0.96 inches diagonal
- **Controller**: SSD1306 or SH1106 compatible
- **Interface**: I2C
- **I2C Address**: 0x3C (default), 0x3D (fallback)
- **Colors**: Monochrome (white on black)

#### Power Supply
- **Input**: 5V via Micro-USB
- **Regulator**: Onboard 3.3V LDO (on main PCB)
- **Current**: ~80-120mA typical

#### Case
- **Material**: Transparent acrylic
- **Pieces**: 6 (top, bottom, 4 sides)
- **Assembly**: Brass standoffs and M2.5 screws

## Pinout

### ESP-01S Pin Configuration

```
┌─────────────────┐
│  ESP-01S Module │
├─────────────────┤
│                 │
│  [antenna]      │
│                 │
│  3V3 │ │ GND    │
│   TX │ │ GPIO0  │  ← I2C SDA (custom mapping!)
│   RX │ │ GPIO2  │  ← I2C SCL (custom mapping!)
│   EN │ │ GND    │
│                 │
└─────────────────┘
```

### Pin Functions

| Pin | Standard Use | This Project |
|-----|--------------|--------------|
| 3V3 | Power (3.3V) | Power |
| GND | Ground | Ground |
| TX | UART TX | Serial debug output |
| RX | UART RX | Serial input (flashing) |
| GPIO0 | General I/O | **I2C SDA** (data line) |
| GPIO2 | General I/O | **I2C SCL** (clock line) |
| EN | Chip Enable | Pulled high (always on) |

**⚠️ Important**: This project uses **non-standard I2C pin mapping**!
- Typical ESP8266: SDA=GPIO4, SCL=GPIO5
- **This device**: SDA=GPIO0, SCL=GPIO2

### I2C Connection

```
ESP-01S          OLED Display
─────────────────────────────
3V3      →       VCC
GND      →       GND
GPIO0    →       SDA
GPIO2    →       SCL
```

### Programming Connection (FTDI)

```
FTDI Adapter     ESP-01S
──────────────────────────
3V3      →       3V3
GND      →       GND
TX       →       RX
RX       →       TX
GND      →       GPIO0  (boot mode - connect only during programming)
```

**Programming Mode:**
1. Connect GPIO0 to GND
2. Power on the ESP-01S
3. Remove GPIO0-GND connection
4. Upload firmware
5. Power cycle to run new code

## PCB Layout

The main PCB (TJ-56-654) contains:
- ESP-01S socket (8-pin header)
- OLED display connector (4-pin header)
- 3.3V voltage regulator (AMS1117-3.3)
- Micro-USB connector for power
- Bypass capacitors

## Memory Map

### Flash Memory (1MB)
```
0x00000000 - 0x00010000  : Bootloader (64KB)
0x00010000 - 0x0007C000  : Firmware (~470KB max for OTA)
0x0007C000 - 0x00080000  : EEPROM emulation (16KB)
0x00080000 - 0x000FA000  : OTA partition (~470KB)
0x000FA000 - 0x000FB000  : WiFi config (4KB)
0x000FB000 - 0x00100000  : System reserved (20KB)
```

### RAM Layout
```
Total: 80KB
├── IRAM (Instruction): 32KB
│   ├── Used: ~62KB (94%)  ← Critical!
│   └── Free: ~4KB
└── DRAM (Data): 48KB
    ├── Heap: ~40KB free
    ├── Stack: ~4KB
    └── Globals: ~4KB
```

## Power Consumption

| Mode | Current | Power @3.3V |
|------|---------|-------------|
| Active (WiFi on) | 80-120mA | 264-396mW |
| Display on | +15mA | +50mW |
| Deep sleep | ~20µA | ~66µW |

**Note**: This firmware does not use deep sleep (clock is always-on).

## Hardware Modifications

### Optional Improvements

1. **External antenna**: Solder U.FL connector for better WiFi range
2. **Temperature sensor**: Add DHT22 or BME280 to GPIO (requires software changes)
3. **Buttons**: Add physical buttons for display control (requires free GPIO)
4. **Battery backup**: Add 18650 cell + TP4056 charger for UPS functionality

### Pin Availability

ESP-01S has very limited GPIO:
- **GPIO0**: Used for I2C SDA (can't use for other purposes)
- **GPIO2**: Used for I2C SCL (can't use for other purposes)
- **TX/RX**: Can be repurposed (breaks serial console)

For additional peripherals, consider upgrading to ESP-12F or ESP32.

## Troubleshooting

### Display not working
- Check I2C address: Try 0x3C and 0x3D
- Verify pin mapping: SDA=GPIO0, SCL=GPIO2
- Check power: Display needs 3.3V
- Test with I2C scanner (`/api/i2c-scan`)

### WiFi connection fails
- ESP8266 only supports 2.4GHz (not 5GHz)
- Check power supply: Weak USB port can cause brownouts
- Some routers don't like ESP8266 - try different channel

### Bootloop/crashes
- Check IRAM usage: Must be <95%
- Verify flash mode: Should be "DIO" not "QIO"
- Bad power supply: Use quality USB cable

### Can't flash firmware
- GPIO0 must be LOW during boot for programming mode
- Some FTDI adapters need DTR/RTS wiring for auto-reset
- Baud rate: Try 115200 (default) or 57600 if errors

## Datasheets

- [ESP8266EX Datasheet](https://www.espressif.com/sites/default/files/documentation/0a-esp8266ex_datasheet_en.pdf)
- [ESP-01S Pinout](https://components101.com/wireless/esp8266-pinout-configuration-features-datasheet)
- [SSD1306 OLED Controller](https://cdn-shop.adafruit.com/datasheets/SSD1306.pdf)

## Safety Warnings

⚠️ **Do NOT connect 5V to ESP-01S GPIO pins** - they are NOT 5V tolerant!

⚠️ **Use 3.3V FTDI adapter** - 5V will permanently damage the ESP8266

⚠️ **Check polarity** - Reversing power can destroy the module

⚠️ **ESD sensitive** - Touch grounded metal before handling board
