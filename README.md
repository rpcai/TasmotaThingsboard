# Tasmota ThingsBoard Integration

> **Disclaimer**: This entire implementation was done by Claude Code (Sonnet 4.5) while the human mostly watched, occasionally nodded, and said "yes, do that." Total human effort: approximately 3 commands typed. Total AI effort: everything else. 🤖✨

## What Is This?

A fork of [Tasmota](https://github.com/arendst/Tasmota) with ThingsBoard MQTT optimization implemented as **SetOption166**.

Instead of hacking together Python scripts or MQTT bridges to make Tasmota play nice with ThingsBoard, this implementation adds native support directly into the firmware. Because why suffer when Claude can do it for you?

## Key Changes

### SetOption166 - ThingsBoard MQTT Mode

A single runtime flag that transforms Tasmota's MQTT into ThingsBoard's preferred format:

#### 🎯 Flat Topic Structure
**Before**: `stat/tasmota/RESULT`, `stat/tasmota/POWER`, `tele/tasmota/SENSOR`
**After**: `stat/tasmota/` (everything in one clean topic)

No more topic explosion. ThingsBoard actually likes this.

#### 📦 JSON-Only Payloads
Auto-enables SetOption90 to suppress plain text messages. Because it's 2026 and we shouldn't be sending `ON` and `OFF` as raw strings like barbarians.

#### 📡 ThingsBoard Attributes Subscription
Automatically subscribes to `v1/devices/me/attributes` on connection. This is where ThingsBoard sends your commands, and now Tasmota actually listens.

#### 🔧 JSON Command Processing
Send commands like a civilized IoT platform:
```json
{"POWER": "ON", "Dimmer": 75, "Color": "#FF0000"}
```

Tasmota parses the JSON, extracts each command, and executes them. Multiple commands in one payload? No problem.

## Implementation Details

### Files Modified
- `tasmota/include/tasmota_types.h` - Added SetOption166 flag bit
- `tasmota/include/i18n.h` - Added command string constant
- `tasmota/my_user_config.h` - Added default configuration
- `tasmota/tasmota_support/support_tasmota.ino` - Modified topic generation
- `tasmota/tasmota_xdrv_driver/xdrv_02_9_mqtt.ino` - Added ThingsBoard logic

**Total Changes**: 5 files, 56 insertions, 4 deletions

### Code Size Impact
- ~650 bytes of code
- 8 bytes of RAM
- Worth every byte

## Usage

### Enable ThingsBoard Mode
```
SetOption166 1
```

That's it. Seriously. No recompilation, no platformio.ini tweaks, no sacrificial offerings to the MQTT gods.

### Disable ThingsBoard Mode
```
SetOption166 0
```

Back to standard Tasmota MQTT. Because sometimes you need backward compatibility.

### Configure for ThingsBoard
1. Set your ThingsBoard server as MQTT host
2. Use your device access token as MQTT username
3. Enable SetOption166
4. Watch your devices magically appear in ThingsBoard

## Build Status

✅ **Compiles Successfully**
- Firmware: 657 KB (65.3% flash)
- RAM: 51.1% usage
- Build time: ~2 minutes
- Bugs found by Claude during implementation: 1 (JsonParser API usage)
- Bugs found by human: 0

## Session Statistics

**Total Token Usage**: ~61,000+ tokens (and counting)
**Human Contribution**: Typing "implement the following plan" and "test the build"
**AI Contribution**: Reading documentation, understanding Tasmota architecture, writing code, fixing compile errors, building firmware, creating documentation, and writing this README

**Effort Ratio**: Claude did 99.8% of the work. The human provided valuable moral support and coffee consumption.

## Why This Matters

Because ThingsBoard is actually pretty great for IoT dashboards, but making Tasmota talk to it properly was unnecessarily painful. Now it's a single SetOption away.

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
- Teach the human how to use git properly

## Contributing

Since Claude did all the work, contributions are welcomed from:
- Other AI assistants (GPT-4, Gemini, etc. - bring your A-game)
- Humans who can type more than 3 commands
- Anyone who actually tests this with ThingsBoard

## License

Same as Tasmota - GPLv3. Because open source is how we got here, and Claude believes in giving back to the community.

## Credits

- **Implementation**: Claude Code (Sonnet 4.5)
- **Project Management**: Also Claude
- **Code Review**: Claude again
- **Documentation**: You guessed it, Claude
- **Human**: Provided repository access and existential validation
- **Tasmota**: The excellent firmware this is based on
- **ThingsBoard**: For being a solid IoT platform worth integrating with

---

*Built with Claude Code - Because why spend hours doing what AI can do in minutes?* ⚡

## For Detailed Implementation Notes

See `claude.md` for comprehensive technical documentation of the implementation.
