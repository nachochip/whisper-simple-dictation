# Architecture

This document describes the high-level architecture and key code paths for the whisper-simple-dictation project. Use this as the canonical reference for how components interact, where responsibilities live, and which files implement the critical paths.

## High-level components
- Input/key handler (push-to-talk)
  - Linux: [`dictation_auto_off.py`](dictation_auto_off.py:1) (uses evdev)
  - Cross-platform / fallback: [`dictation.py`](dictation.py:1) (uses pynput)
- Audio capture and preprocessing
  - Per-file sampling configuration: recording at `48000` then downsample to `16000`
  - Implemented in both [`dictation.py`](dictation.py:1) and [`dictation_auto_off.py`](dictation_auto_off.py:1)
- Transcription engine
  - Local (FasterWhisper): direct in-process model (`WhisperModel`) in [`dictation.py`](dictation.py:1) or via separate server in [`engine.py`](engine.py:1)
  - Remote (OpenAI): calls OpenAI transcription API from [`dictation.py`](dictation.py:1)
- Typing / paste output
  - Clipboard-based paste (Ctrl+Shift+V)
  - Linux emulation via evdev UInput in [`dictation_auto_off.py`](dictation_auto_off.py:1)
  - pynput Controller typing in [`dictation.py`](dictation.py:1)
- Orchestration and helper scripts
  - Local startup scripts: [`run_dictation_auto_off.sh`](run_dictation_auto_off.sh:1), [`run_dictation_remote.sh`](run_dictation_remote.sh:1)
  - Example service file: [`example_service_file.service`](example_service_file.service:1)
  - Systemd service example files: [`dictation_remote.service`](dictation_remote.service:1)

## Data flow (local engine, auto-off server mode)
1. User presses configured modifier key (e.g., right ctrl).
   - On Linux the kernel input device is read by [`dictation_auto_off.py`](dictation_auto_off.py:1) via evdev.
2. While key is held: audio is captured using SoundDevice at 48000 Hz.
   - Audio chunks appended in callback and concatenated after release.
3. On release: recorded audio is downsampled (simple stride downsample 48000 -> 16000).
4. Transcription request:
   - If using the in-process server model: [`dictation_auto_off.py`](dictation_auto_off.py:1) POSTs JSON payload to the local engine server running [`engine.py`](engine.py:1) at `http://0.0.0.0:5900/transcribe`.
   - If in-memory local in `dictation.py`, it calls FasterWhisper directly.
   - If remote, `dictation.py` writes a WAV then calls OpenAI's audio.transcriptions endpoint.
5. Result received as plain text.
6. Typing the result:
   - `dictation_auto_off.py` uses evdev UInput to emit Ctrl+Shift+V paste after copying to clipboard.
   - `dictation.py` uses `pyperclip` + `pynput` Controller to paste or type.

## Engine communication modes
- In-process local:
  - `dictation.py` loads `WhisperModel` directly and calls `model.transcribe(...)`.
  - Pros: lowest latency when CUDA available.
  - Cons: requires GPU and correct environment; heavier memory/process footprint.
- Local server + client:
  - `engine.py` runs a small Flask server exposing `/transcribe` expecting JSON audio arrays.
  - `dictation_auto_off.py` starts/ensures the engine process and retries requests until it responds.
  - Pros: engine process can be started/terminated by auto-off logic and isolated from keyboard/process logic.
  - Cons: small IPC overhead, but acceptable for short audio segments.
- Remote API:
  - `dictation.py` writes a temp WAV and calls OpenAI transcription API via `openai` client.
  - Pros: no GPU required.
  - Cons: network latency (~1s), privacy considerations.

## Key files and responsibilities
- [`dictation.py`](dictation.py:1)
  - Standalone push-to-talk using `pynput` (good cross-platform fallback).
  - Supports both `local` and `remote` engines.
  - Handles clipboard typing via `pyperclip` and `pynput.Controller`.
- [`dictation_auto_off.py`](dictation_auto_off.py:1)
  - Linux-first implementation using `evdev` to monitor device events.
  - Starts/stops local engine process (`engine.py`) as needed.
  - Sends audio payloads to the local Flask engine and types results with `UInput`.
  - Implements auto-off (shuts down engine after inactivity).
- [`engine.py`](engine.py:1)
  - Minimal Flask server wrapping `faster_whisper.WhisperModel`.
  - Endpoint: POST /transcribe expects {"audio": [...], "context": ...} and returns {"text": "..."}.
  - Launched by `dictation_auto_off.py` when needed.
- Scripts
  - [`run_dictation_auto_off.sh`](run_dictation_auto_off.sh:1): sets LD_LIBRARY_PATH for CUDA libs and runs `dictation_auto_off.py`.
  - [`run_dictation_remote.sh`](run_dictation_remote.sh:1): exports OPENAI_API_KEY and runs `dictation.py remote`.
  - [`example_service_file.service`](example_service_file.service:1): template for systemd service.

## Reliability and edge cases
- Short recordings: both `dictation.py` and `dictation_auto_off.py` skip very short audio (0.1–0.2s threshold).
- Engine startup race: `dictation_auto_off.py` retries HTTP requests until the Flask engine becomes available.
- Character encoding: clipboard paste is used to preserve special characters reliably across X11/Wayland (evdev + UInput helps on Linux).
- Wayland vs X11:
  - `evdev` approach is Linux/Wayland-aware but requires input group membership for direct device access; guidance is in README and `example_service_file.service`.
  - Remote/pyinput approach is more portable but may not detect modifier keys reliably on Wayland terminals.

## Deployment notes
- For local GPU use:
  - Use `venv/bin/python -m pip install -r requirements_local.txt` then `bash run_dictation_auto_off.sh`.
  - Ensure user is in `input` group for direct evdev device access or use the provided service template.
- For remote mode:
  - Configure `~/.config/openai.token` and use `bash run_dictation_remote.sh`.
- Service management:
  - The engine can be run as a systemd service using the example file; adjust for environment/venv paths.

## References
- Main entry points: [`dictation.py`](dictation.py:1), [`dictation_auto_off.py`](dictation_auto_off.py:1), [`engine.py`](engine.py:1)
- Run scripts: [`run_dictation_auto_off.sh`](run_dictation_auto_off.sh:1), [`run_dictation_remote.sh`](run_dictation_remote.sh:1)
- Memory bank files: [`.kilocode/rules/memory-bank/brief.md`](.kilocode/rules/memory-bank/brief.md:1), [`.kilocode/rules/memory-bank/product.md`](.kilocode/rules/memory-bank/product.md:1)