# Tech

This file documents the development setup, dependencies, and technical constraints for the project.

## Languages and runtime
- Python 3.11+ recommended

## Key dependencies
- faster-whisper (for local transcription using Whisper models)
- openai (for remote transcription API)
- sounddevice (audio capture)
- numpy
- pyperclip
- pynput (for cross-platform key handling)
- evdev (Linux input device handling and UInput)
- flask (for local engine server)

## Development setup
1. Create virtual environment:
   - python3 -m venv venv --copies
2. Activate and install:
   - venv/bin/python -m pip install -r requirements_local.txt (for local)
   - venv/bin/python -m pip install -r requirements_remote.txt (for remote)
3. If running locally with CUDA:
   - Ensure correct CUDA libs are available; `run_dictation_auto_off.sh` sets LD_LIBRARY_PATH to expected locations.
   - Add user to input group: sudo usermod -aG input __YOUR_USER_NAME__ and re-login.

## Runtime considerations
- Local GPU mode requires a CUDA-capable GPU with >= 4GB VRAM for reasonable performance with large models.
- Remote OpenAI API has network latency; expect ~1s delay as of 2024.
- On Wayland, pynput may not detect modifier keys in terminal windows; use evdev approach on Linux where possible.

## Testing & debugging
- Use `bash run_dictation_auto_off.sh en` for local testing.
- Use `bash run_dictation_remote.sh en` for remote testing.
- Log helpful messages when recording starts/stops and when transcription returns.

## Files to review when making changes
- `dictation.py`
- `dictation_auto_off.py`
- `engine.py`
- `run_dictation_auto_off.sh`
- `run_dictation_remote.sh`