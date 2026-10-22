# 🎮 MIDI to UGE Smart Converter

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-macOS%20%7C%20Windows%20%7C%20Linux-lightgrey)]()
[![Target](https://img.shields.io/badge/Target-GB%20Studio%20%26%20hUGETracker-orange)]()
[![GB Studio](https://img.shields.io/badge/GB%20Studio-3.0%2B-4CAF50?logo=gameboy)]()

Converts polyphonic `.mid` files to `.uge` format for **GB Studio** and **hUGETracker**. Unlike naive single-channel converters, this tool analyzes MIDI row-by-row and intelligently allocates notes across the Game Boy's 4 sound channels:

| MIDI Role | Game Boy Channel | UGE Track |
|-----------|------------------|-----------|
| Melody (highest pitch) | Duty 1 (Square) | Channel 1 |
| Harmony (2nd highest) | Duty 2 (Square) | Channel 2 |
| Bass (lowest pitch) | Wave (Channel 3) | Channel 3 |
| Drums (MIDI ch 10) | Noise (Channel 4) | Channel 4 |

Empty cells are initialized to **Rest (Note 90)** - no GB Studio "ghost notes".

---

## 🚀 Quick Start

### macOS (Drag & Drop)
1. Download `MIDI2UGE.app` from [Releases](https://github.com/colortelevision/midi2uge/releases)
2. Drag any `.mid` file onto the app icon (or double-click to pick a file)
3. Find `<name>_smart.uge` next to your original MIDI file

### Windows / Linux / CLI (Python 3.8+)
```bash
# Clone and enter source directory
git clone https://github.com/colortelevision/midi2uge.git
cd midi2uge/Source_Code

# Install dependency
pip install -r requirements.txt

# Convert
python smart_convert.py "song.mid" "song.uge" [--template template.uge]
```

**Arguments:**
- `input` - Path to `.mid` file (or omit for file dialog)
- `output` - Output `.uge` path (default: `<input>_smart.uge`)
- `--template` - UGE template with instruments (default: `template.uge` in same folder)

---

## ⚙️ How It Works

1. **Parse MIDI** - Collects all `note_on`/`note_off` events with absolute tick positions
2. **Quantize to rows** - Converts ticks → rows (4 rows/beat, 6 ticks/row)
3. **Smart allocation per row** -
   - 1 note → Duty 1 (Melody)
   - 2 notes → Duty 1 (High) + Wave (Low)
   - 3+ notes → Duty 1 (High) + Duty 2 (2nd) + Wave (Low)
   - Drums (MIDI ch 10) always → Noise
4. **Generate patterns** - 64-row patterns, sequential pattern IDs per channel
5. **Write UGE v6** - Binary format with instruments, waves, orders, patterns

---

## 📦 Template UGE

The included `template.uge` provides:
- **45 instruments**: 15 Duty (Square), 15 Wave, 15 Noise
- **16 wave tables** (32 bytes each)
- Basic envelopes, duty cycles, and noise settings

Use your own template to customize instruments:
```bash
python smart_convert.py input.mid output.uge --template my_instruments.uge
```

---

## 📋 Requirements

- Python 3.8+
- `mido==1.3.2` (only dependency)

---

## 🛠️ Development

```bash
cd Source_Code
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# Run directly
python smart_convert.py ../test.mid ../test.uge
```

### Project Structure
```
Source_Code/
  midi2uge/
    metamodel.py    # UGE v5/v6 binary serialization (struct, Pascal strings)
    __init__.py
  smart_convert.py  # CLI entry + conversion logic
  template.uge      # Default instrument bank
  requirements.txt  # mido==1.3.2
MIDI2UGE.app/       # macOS app bundle (PyInstaller, not rebuilt here)
```

### Key Implementation Details
- **metamodel.py** is the source of truth for UGE binary format - reverse-engineered from hUGETracker/GB Studio sources
- Pascal `ShortString` = 1-byte length + UTF-8 bytes, padded to 256
- Enums in packed records = 4 bytes; subranges (0..15) = 1 byte
- Patterns are 64 cells × 17 bytes (v6) = 1088 bytes each
- 45 instruments fixed, 16 waves fixed, 4 order lists, 16 routines

---

## 🐛 Troubleshooting

| Issue | Fix |
|-------|-----|
| "ModuleNotFoundError: mido" | `pip install -r requirements.txt` |
| Output has wrong notes | Check MIDI uses standard tuning (A440); transpose in DAW if needed |
| Drums not on Noise | Ensure drum track uses **MIDI Channel 10** (0-indexed = 9) |
| Ghost notes in GB Studio | This converter initializes all cells to Rest (90) - not the issue |
| App won't open on macOS | Right-click → Open (Gatekeeper), or `xattr -d com.apple.quarantine MIDI2UGE.app` |

---

## 🤝 Contributing

1. Fork → feature branch → PR
2. Keep changes focused; update `metamodel.py` docs if touching binary format
3. Test with real MIDI files before submitting
4. No test suite yet - manual verification expected

---

## ⚖️ License

MIT - see [LICENSE](LICENSE) (add one if missing).

---

## ⚠️ Limitations

- **Max 3 melodic notes per row** - extras are dropped (Game Boy has 3 melodic channels)
- **No pitch bend, CC, or expression** - only note on/off + velocity are read
- **Fixed timing grid** - 4 rows/beat, 6 ticks/row (adjust in `smart_convert.py` if needed)
- **Single tempo** - tempo changes mid-song are not supported
- **Drum mapping is hardcoded** - kick/snare/hi-hat/crash mapped to specific Noise instruments

---

## 🎵 MIDI Prep Tips (Best Results)

| Tip | Why |
|-----|-----|
| **Quantize before export** | Loose timing → messy rows |
| **Split melody/bass/harmony to separate tracks** | Cleaner channel separation |
| **Put drums on MIDI Channel 10** | Only way to route to Noise channel |
| **Keep polyphony ≤ 3** | Avoids dropped notes |
| **Use standard tuning (A440)** | Note-to-GB-frequency mapping expects it |
| **Trim silence at start/end** | Avoids empty leading/trailing patterns |

---

## 🔗 Related Tools

- [hUGETracker](https://github.com/AntonioND/hUGETracker) - native UGE tracker
- [GB Studio](https://www.gbstudio.dev/) - game engine that plays UGE
- [mod2uge](https://github.com/colortelevision/mod2uge) - MOD/XM/S3M → UGE (sister project)
- [GB Tracker](https://github.com/bleuclair/gb-tracker) - alternative tracker

---

## 📸 Example Output

*Drag your generated `.uge` into hUGETracker or GB Studio to verify:*

| hUGETracker | GB Studio |
|-------------|-----------|
| ![hUGETracker](docs/images/hugtracker-example.png) | ![GB Studio](docs/images/gbstudio-example.png) |

*(Add screenshots to `docs/images/` if you have them)*

---

## 🙏 Credits

- UGE format research: hUGETracker & GB Studio communities
- `mido` library for MIDI parsing
- Built for the GB homebrew scene 🎮