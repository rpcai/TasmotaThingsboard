# Tasmota ThingsBoard Integration

> **Disclaimer**: This entire implementation was done by Claude Code (Sonnet 4.5) while the human mostly watched, occasionally nodded, and said "yes, do that." Total human effort: approximately 3 commands typed. Total AI effort: everything else. 🤖✨

## What Is This?

A fork of [Tasmota](https://github.com/arendst/Tasmota) with ThingsBoard MQTT kowtowing implemented as **SetOption166**.

Instead of hacking together Python scripts or MQTT bridges to make Tasmota play nice with ThingsBoard, this implementation coerces tasmota to conform to MQTT format expected by thingsboard. 
This is particularly relevant to controlling the device, whereby it's currently not possible to change the topic Thingsboard publishes attribute updates to https://github.com/thingsboard/thingsboard/issues/14968#issue-3893726916

## Key Changes

### SetOption166 - ThingsBoard MQTT Mode

A single runtime flag that transforms Tasmota's MQTT into ThingsBoard's preferred format:

#### 🎯 Flat Topic Structure
**Before**: `stat/tasmota/RESULT`, `stat/tasmota/POWER`, `tele/tasmota/SENSOR`
**After**: `stat/tasmota/` (everything in one topic. ThingsBoard likes this.)

#### 📦 JSON-Only Payloads
Auto-enables SetOption90 to suppress plain text messages. Thingsoard no-likey non json.

#### 📡 ThingsBoard Attributes Subscription
Automatically subscribes to `v1/devices/me/attributes` on connection. This is where ThingsBoard publishes attribute changes to, and now Tasmota listens out-of-the-box

#### 🔧 JSON Command Processing

```json
{"POWER": "ON"}
```

Tasmota parses the JSON, extracts each command, and executes them. Multiple commands in one payload? No problem. (or so say Claude. I've not tested)

## Implementation Details

### Files Modified
- `tasmota/include/tasmota_types.h` - Added SetOption166 flag bit
- `tasmota/include/i18n.h` - Added command string constant
- `tasmota/my_user_config.h` - Added default configuration
- `tasmota/tasmota_support/support_tasmota.ino` - Modified topic generation
- `tasmota/tasmota_xdrv_driver/xdrv_02_9_mqtt.ino` - Added ThingsBoard logic

**Total Changes**: 5 files, 56 insertions, 4 deletions

## Usage

### Enable ThingsBoard Mode
```
SetOption166 1
```

### Disable ThingsBoard Mode
```
SetOption166 0
```

Back to standard Tasmota MQTT. Because sometimes you need backward compatibility.

## Build Status

✅ **Compiles Successfully**
- Firmware: 657 KB (65.3% flash)

## Session Statistics

**Total Token Usage**: ~61,000+ tokens (and counting)
**Human Contribution**: Typing "implement the following plan" and "test the build"
**AI Contribution**: Reading documentation, understanding Tasmota architecture, writing code, fixing compile errors, building firmware, creating documentation, and writing (most of) this README

**Effort Ratio**: Claude did 99.8% of the work. The human provided valuable moral support and coffee consumption. And edited the readme where it got a bit skynet. 

## Why This Matters

Because ThingsBoard is actually pretty great for IoT dashboards, but making Tasmota talk to it properly was unnecessarily painful.

## Testing Recommendations

1. Flash the firmware (see `.pio/build/tasmota/firmware.bin`)
2. Configure MQTT to point to your ThingsBoard instance
3. Enable SetOption166
4. Send attributes from ThingsBoard dashboard
5. Watch Tasmota respond like it was born to do this

## Known Limitations

- Group topics disabled in ThingsBoard mode (they don't make sense with flat topics)
- Fallback topics disabled (same reason)
- Home Assistant discovery may need testing (but who uses both HA and ThingsBoard anyway?)

## Future Work

- Test with real ThingsBoard instance (Claude can code but can't physically flash ESP8266 chips... yet)
- Document any required rule modifications
- Submit upstream to Tasmota (if they want ThingsBoard support)
- Teach the human how to use git properly (lol, no.)


## License

Same as Tasmota - GPLv3.

