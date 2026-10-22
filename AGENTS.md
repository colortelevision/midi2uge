# midi2uge — MIDI to UGE Converter

## Project Overview
Python tool that converts polyphonic `.mid` files to `.uge` format for GB Studio / hUGETracker. Smartly allocates notes across 4 Game Boy channels: Melody→Duty 1, Harmony→Duty 2, Bass→Wave, Drums (ch 10)→Noise.

## Structure
```
Source_Code/
  midi2uge/
    metamodel.py    # UGE binary format (de/serialization, v5/v6)
    __init__.py
  smart_convert.py  # CLI entry point & conversion logic
  template.uge      # Default instrument bank (binary)
  requirements.txt  # mido==1.3.2
MIDI2UGE.app/       # macOS app bundle (drag-and-drop)
```

## Developer Commands

### Run conversion (CLI)
```bash
cd Source_Code
python smart_convert.py "input.mid" "output.uge" [--template template.uge]
```
- If `--template` omitted, uses `template.uge` in same dir
- If no args, opens file dialog (tkinter)

### Install dependencies
```bash
cd Source_Code
pip install -r requirements.txt
```

### Run as macOS app
Drag `.mid` file onto `MIDI2UGE.app` (or double-click to pick file). Output: `<name>_smart.uge` next to input.

## Architecture Notes
- **No tests, no lint, no typecheck** — pure Python script
- **Virtual env** at `.venv/` (repo root)
- **metamodel.py** handles UGE v5/v6 binary format exactly (struct packing, Pascal ShortString, enums as 4 bytes in packed records)
- **smart_convert.py** does row-by-row MIDI analysis → allocates up to 3 melodic notes/row across 3 channels + drums on ch 4
- Template UGE provides 45 instruments (15 Duty, 15 Wave, 15 Noise) + 16 waves
- Output patterns: 64 rows each, initialized to Rest (note=90) to avoid GB Studio ghost notes

## Key Constraints
- Python 3.8+
- Only dependency: `mido==1.3.2`
- No build step — runs directly from source
- macOS app is a PyInstaller/bundle artifact (not rebuilt from this repo)
- UGE format is reverse-engineered; `metamodel.py` is the source of truth for binary layout