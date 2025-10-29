# Product

This project provides a low-latency push-to-talk dictation tool powered by OpenAI's Whisper (local or remote). It is designed to let users quickly convert short speech segments into typed text using a simple press-to-record workflow.

Why this project exists
- Improve rapid transcription for interactive workflows (coding, note-taking, terminal input) where short, accurate dictation is more useful than continuous streaming.
- Provide an easy-to-run, local-first option using FasterWhisper for users with CUDA-capable hardware while supporting a remote OpenAI API fallback for systems without GPU.
- Keep the integration lightweight and focused on usability (push-to-talk), low latency, and predictable behavior.

Problems it solves
- Replaces manual typing for short, focused text entry tasks.
- Avoids confusing delayed incremental transcription by transcribing after each push-to-talk session.
- Offers a secure local option for users who prefer not to send audio to an external service.

How it should work (high-level)
- User presses a configured modifier key to start recording and releases it to stop.
- The audio is downsampled and sent to a transcription engine (local FasterWhisper or OpenAI remote).
- The resulting text is pasted/typed into the active application using clipboard paste (Ctrl+Shift+V) or emulated key events.
- Optional context-grab and command-word handling can modify output or trigger actions (enter key, etc.).

Target users
- Developers and power users who want quick dictation integrated into their desktop workflow.
- Linux users who can leverage evdev for robust push-to-talk handling.
- Users with moderate privacy/security needs who prefer a local transcription option.

Primary non-goals
- Real-time continuous streaming transcription with sub-second rolling updates.
- Full GUI application or cross-platform parity in every feature (focus is Linux-first).
- Headline-level editing, punctuation fixes, or large-document transcription workflows.

Key user stories
- As a developer, I want to dictate a short commit message by holding a key so I can keep my hands near the keyboard.
- As a terminal user, I want to paste dictated text into my terminal reliably without breaking special characters.
- As a privacy-conscious user, I want the option to run transcription locally on my GPU.

Acceptance criteria (high-level)
- Press-and-hold starts audio capture; release stops capture and triggers transcription.
- Local engine path works with FasterWhisper for supported models (e.g., large-v3); remote path uses OpenAI's transcription API.
- Typed output reliably arrives in the focused application, including special characters (via clipboard paste).
- Reasonable guardrails: ignore too-short recordings, recover from engine startup delays, and provide an auto-off option.
