# Project Plan: Raspberry Pi Whisper-based Speech-to-Text HID

This document outlines the step-by-step plan for creating a speech-to-text device using a Raspberry Pi, OpenAI's Whisper, and USB HID emulation.

## Development Methodology: Container-First

To ensure consistency between the development and production environments, this project will use a container-first approach. All development will be done inside a Docker container that mimics the Raspberry Pi's ARM-based Debian OS, managed via VS Code's Dev Containers extension.

---

## Phase 1: Project & Environment Setup

1.  **Hardware Preparation**:
    *   Flash a microSD card with the latest Raspberry Pi OS.
    *   Enable SSH and configure Wi-Fi for headless setup.
    *   Boot the Pi, connect via SSH, and run system updates (`sudo apt update && sudo apt upgrade`).

2.  **Docker and VS Code Dev Containers Setup (on the main development machine)**:
    *   Install Docker Desktop.
    *   Install the "Dev Containers" extension in VS Code.
    *   Create a `.devcontainer` directory in the project root.
    *   Create a `Dockerfile` to define the environment (base ARM Debian image, install `git`, `ffmpeg`, audio libraries).
    *   Create a `devcontainer.json` file to configure the container, including VS Code extensions to install (`ms-python.python`, `charliermarsh.ruff`) and a `postCreateCommand` to install `uv`.

3.  **Launch the Dev Container**:
    *   Open the project in VS Code and select "Reopen in Container".
    *   The container will build and start, providing a Pi-like development environment.

4.  **Project Initialization (inside the container)**:
    *   Create a repository on GitHub.
    *   From the dev container's terminal, initialize the project.
    *   Create a Python virtual environment using `uv venv`.

---

## Phase 2: Code Quality & Git Workflow

1.  **Git Workflow**:
    *   Define and configure the `main`, `develop`, and `feat/...` branching strategy.
    *   Protect `main` and `develop` branches in GitHub settings.

2.  **Linter/Formatter Setup**:
    *   In the dev container, install `ruff` and `pre-commit` using `uv pip install`.
    *   Configure `ruff` rules in the `pyproject.toml` file.

3.  **Pre-commit Hook**:
    *   Create a `.pre-commit-config.yaml` to run `ruff` automatically before each commit.
    *   Install the hooks with `pre-commit install`.
    *   Commit the configuration files to the repository.

---

## Phase 3: Raspberry Pi as a USB HID

1.  **Enable Gadget Mode on Pi**:
    *   Modify `/boot/config.txt` to add `dtoverlay=dwc2`.
    *   Modify `/boot/cmdline.txt` to add `modules-load=dwc2,g_hid`.

2.  **HID Script Development (in container)**:
    *   Write a Python script (`src/keyboard.py`) to simulate keystrokes by writing to the HID device file (`/dev/hidg0`).
    *   The script will map characters to HID scancodes.

3.  **Hardware Test (on Pi)**:
    *   Deploy the script to the Pi.
    *   Connect the Pi's USB-C/OTG port to a computer and run the script to test typing.

---

## Phase 4: Audio Recording

1.  **Install Libraries (in container)**:
    *   Use `uv pip install pyaudio wave` to install audio libraries.

2.  **Recording Script (in container)**:
    *   Write a script (`src/audio.py`) using `pyaudio` to record audio from a microphone and save it as a `.wav` file.

3.  **Hardware Test (on Pi)**:
    *   Deploy and run the script on the Pi with a USB microphone to verify it can record audio correctly.

---

## Phase 5: Speech-to-Text with Whisper

1.  **Install Dependencies (in container)**:
    *   Install the correct ARM-compatible PyTorch wheel.
    *   Install Whisper and `tiktoken` using `uv pip install`.

2.  **Transcription and Testing Script (in container)**:
    *   Develop an advanced script (`transcriber.py`) that accepts a model name (`tiny.en`, `base.en`, etc.) and audio file as arguments.
    *   The script must time model loading and transcription speed.
    *   Create a suite of test audio files.
    *   Create a `run_tests.py` script to automate testing across all models and audio files, logging results to `results.md`.

3.  **Analysis**:
    *   Analyze the `results.md` file to select the best model that balances performance and accuracy.

---

## Phase 6: Application Integration

1.  **Refactor Code**:
    *   Organize the functions from the previous steps into a clean `src` module.

2.  **Create Main Application (`main.py`)**:
    *   Develop the main application loop: Wait for trigger -> Record -> Transcribe -> Type.
    *   Load the chosen Whisper model only once at startup.
    *   Use a simple `input()` for the initial trigger mechanism.

3.  **Integration Test (on Pi)**:
    *   Deploy the full application to the Pi and perform an end-to-end test.

---

## Phase 7: Performance Testing and Optimization

1.  **Establish Baseline**:
    *   Measure the end-to-end performance of the integrated application.

2.  **Optimization Experiments (in `feat/performance-tuning` branch)**:
    *   **Quantization**: Investigate `ctranslate2` or `whisper.cpp` to run faster, quantized models.
    *   **VAD**: Implement Voice Activity Detection using `webrtcvad` to only process speech, reducing unnecessary processing.

3.  **Merge**:
    *   Analyze results and merge the best-performing solution into `develop`.

---

## Phase 8: Deployment and Service Setup

1.  **Deployment Script**:
    *   Create a `deploy.sh` script on the Pi to automate pulling the latest code from `main` and syncing dependencies.

2.  **Systemd Service**:
    *   Create a `systemd` service file to run the `main.py` application automatically on boot.
    *   Enable and start the service.

---

## Phase 9: CI, Documentation, and Finalization

1.  **CI Pipeline**:
    *   Set up a GitHub Actions workflow (`.github/workflows/ci.yml`) to run `ruff` on every push to `develop` and PRs.

2.  **Documentation**:
    *   Create a `README.md` with detailed setup and usage instructions for both the development environment and the final device.
    *   Clean up and comment the code.