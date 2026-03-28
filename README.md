# AbletonMCP Skill — Claude Code x Ableton Live

Control Ableton Live directly from Claude Code via TCP socket. No Claude Desktop needed.

## What this is

A fully autonomous setup that lets Claude Code:
- Install and patch AbletonMCP (fixes 10+ bugs in the base remote script)
- Build complete sessions (tracks, instruments, effects, routing, MIDI patterns)
- Control Ableton in real-time (playback, tempo, device parameters, arm/monitor)
- Auto-dismiss Ableton popups via AppleScript

## What the human does (3 things)

1. Download **Ableton Live 12 Trial** from ableton.com
2. Set **buffer size to 128** in Preferences > Audio
3. Select **AbletonMCP** in Preferences > Link, Tempo & MIDI > Control Surface

Everything else is automated.

## Files

| File | Description |
|---|---|
| `ABLETON_MCP_SKILL.md` | Complete skill doc — drop into Claude Code and say "execute this" |
| `AbletonMCP_patched__init__.py` | Patched remote script with all bug fixes and new commands |

## Patched Commands (not in base AbletonMCP)

- `create_audio_track` — for live instruments
- `delete_track` — clean up tracks
- `set_track_input_routing` — route audio inputs (with exact channel matching)
- `get_track_routing_info` — inspect current routing
- `set_track_volume` — adjust track volume
- `set_track_arm` — arm tracks for recording
- `set_track_monitor` — set monitor mode (In/Auto/Off)
- `get_device_parameters` — list all device params with ranges
- `set_device_parameter` — tweak any device parameter
- `get_clip_notes` — read MIDI notes back from clips

## Bugs Fixed

- `load_instrument_or_effect` not dispatched + missing method
- `clip.set_notes()` broken in Live 12 (replaced with `add_new_notes`)
- Channel matching ("2" in "1/2" matched wrong)
- Thread safety (commands must run in `main_thread_task`)
- UI refresh for MIDI notes

## Song Preset: "Indie Rock"

- 140 BPM, Plymouth Kit (acoustic)
- 4-bar garage pattern with counter-tempo cut on bar 3 + snare build on bar 4
- Guitar L: robust, panned left 50% (Gate > EQ > Echo > Compressor +20dB > Utility > Amp)
- Guitar R: shiny, panned right 50% (Gate > EQ > Saturator > Compressor +20dB > Utility > Amp > Chorus)
- Bass: clean, EQ only, input ch 1
- Guitars on input ch 2 (Behringer UMC)

## Protocol

```python
import socket, json, time

def ableton(cmd, params=None):
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.settimeout(15)
    sock.connect(("localhost", 9877))
    msg = json.dumps({"type": cmd, "params": params or {}})
    sock.sendall((msg + "\n").encode())
    data = sock.recv(16384)
    sock.close()
    time.sleep(0.5)
    return json.loads(data.decode())
```

---

*Created March 2026 by Blas Palmisciano + Claude Code*
