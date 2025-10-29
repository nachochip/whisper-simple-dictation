# Project Brief

This project is a simple dictation tool using OpenAI's Whisper for speech-to-text transcription. It allows users to press a key to start recording, release to stop, and have the transcribed text typed out via simulated keypresses.

## Core Functionality
- Push-to-talk recording with key press/release
- Local Whisper execution (requires CUDA, 4GB+ VRAM) or remote OpenAI API
- Automatic text typing via clipboard paste (Ctrl+Shift+V)
- Support for multiple languages and Whisper models
- Linux-focused with evdev for local input handling

## Key Components
- `dictation.py` - Main dictation logic
- `engine.py` - Whisper integration (local/remote)
- `dictation_auto_off.py` - Local version with auto-off functionality
- Various run scripts for different configurations

## Technologies
- Python 3
- Whisper (OpenAI/FasterWhisper)
- evdev (Linux input handling)
- pynput (cross-platform fallback)
- OpenAI API (remote option)

## Current Status
- Functional push-to-talk dictation
- Local and remote execution modes
- Basic language and model selection
- Service setup for background running

## Goals
- Reliable, low-latency speech-to-text dictation
- Cross-platform compatibility where possible
- Simple installation and usage