# Project User Stories & Acceptance Criteria

This document provides a detailed breakdown of project tasks in the format of user stories, suitable for backlog management in tools like GitHub Issues. Each number corresponds to a phase or step in the `PLAN.md` document.

---

### Epic 1: Environment & Project Setup

**Story 1.1: Physical Pi Preparation**
-   **As a developer,** I want to prepare the physical Raspberry Pi with a fresh OS and network access **so that** I have a stable, known target for deployment and hardware testing.
-   **Acceptance Criteria:**
    -   [ ] Raspberry Pi OS is flashed onto the microSD card.
    -   [ ] Headless SSH and Wi-Fi are configured.
    -   [ ] The Pi is accessible over the network via SSH.
    -   [ ] The OS is fully updated using `apt update` and `apt upgrade`.

**Story 1.2: Containerized Development Environment**
-   **As a developer,** I want to define a containerized development environment using Docker and VS Code **so that** my development environment on my machine perfectly matches the Pi's ARM architecture and OS, preventing environment-related bugs.
-   **Acceptance Criteria:**
    -   [ ] A `.devcontainer` directory exists in the project root.
    -   [ ] A `Dockerfile` is created, based on an ARM Debian image, and installs all required `apt` packages (git, ffmpeg, audio libs).
    -   [ ] A `devcontainer.json` is created, configuring VS Code to use the Dockerfile and automatically install the Python and Ruff extensions.
    -   [ ] The project can be successfully opened in a VS Code Dev Container.

**Story 1.3: Project Initialization and Dependency Management**
-   **As a developer,** I want to initialize the project with `uv` for dependency management **so that** I have a fast and modern way to handle Python packages within the container.
-   **Acceptance Criteria:**
    -   [ ] The `devcontainer.json` includes a `postCreateCommand` to install `uv`.
    -   [ ] A virtual environment is created inside the dev container using `uv venv`.
    -   [ ] The virtual environment can be activated from the VS Code terminal.

---

### Epic 2: Code Quality & Git Workflow

**Story 2.1: Git Branching Strategy**
-   **As a developer,** I want a formal Git branching strategy (`main`, `develop`, `feat`) established and enforced **so that** the codebase remains stable and organized.
-   **Acceptance Criteria:**
    -   [ ] `main` and `develop` branches are created on GitHub.
    -   [ ] Branch protection rules are enabled for `main` and `develop` to require Pull Requests for merges.

**Story 2.2: Automated Code Quality Enforcement**
-   **As a developer,** I want `ruff` and `pre-commit` configured at the project's start **so that** all code is automatically linted and formatted before being committed, ensuring consistency and quality.
-   **Acceptance Criteria:**
    -   [ ] `ruff` and `pre-commit` are added as development dependencies.
    -   [ ] A `pyproject.toml` file contains configuration for `ruff`.
    -   [ ] A `.pre-commit-config.yaml` is created to run `ruff check --fix` and `ruff format`.
    -   [ ] `pre-commit install` successfully installs the hooks.
    -   [ ] A test commit on a modified file triggers the pre-commit hook successfully.

---

### Epic 3: Core Hardware Features

**Story 3.1: HID Keyboard Emulation**
-   **As a user,** I want the Raspberry Pi to function as a plug-and-play USB keyboard **so that** it can type transcribed text into any connected computer without drivers.
-   **Acceptance Criteria:**
    -   [ ] The Pi's `/boot/config.txt` and `/boot/cmdline.txt` are configured for USB gadget mode.
    -   [ ] A Python script (`src/keyboard.py`) exists that can write to `/dev/hidg0`.
    -   [ ] When the script is run on the Pi (connected via OTG port), it successfully types pre-defined text into a text editor on a host computer.

**Story 3.2: Audio Recording**
-   **As a developer,** I want to reliably capture audio from a USB microphone on the Pi **so that** I have raw audio data for the speech-to-text model.
-   **Acceptance Criteria:**
    -   [ ] The `pyaudio` library is added as a dependency.
    -   [ ] A Python script (`src/audio.py`) is created.
    -   [ ] When run on the Pi with a microphone, the script successfully records and saves a `.wav` file.
    -   [ ] The saved `.wav` file is playable and contains clear audio.

---

### Epic 4: Speech-to-Text Implementation

**Story 4.1: Whisper Model Benchmarking**
-   **As a developer,** I want to systematically benchmark different Whisper models on the Pi **so that** I can make a data-driven decision on which model offers the best tradeoff between speed and accuracy.
-   **Acceptance Criteria:**
    -   [ ] PyTorch, Whisper, and `tiktoken` are installed in the dev environment.
    -   [ ] A script exists (`transcriber.py`) that takes a model name and audio file as input.
    -   [ ] A test suite of short `.wav` files is created and committed.
    -   [ ] An automation script (`run_tests.py`) runs all tests and logs model load time, transcription time, and transcribed text to a `results.md` file.

**Story 4.2: Integrated Application Logic**
-   **As a user,** I want a single application that combines recording, transcription, and typing **so that** I can perform the full speech-to-text workflow.
-   **Acceptance Criteria:**
    -   [ ] A `main.py` script is created that imports and orchestrates the audio, transcription, and keyboard modules.
    -   [ ] The application loads the selected Whisper model once at startup.
    -   [ ] The application waits for a trigger (e.g., `input()`), then performs the record -> transcribe -> type sequence.
    -   [ ] The end-to-end flow works correctly during a hardware test on the Pi.

---

### Epic 5: Performance and Deployment

**Story 5.1: Real-Time Performance Optimization**
-   **As a user,** I want the transcription to feel near-instantaneous **so that** the device is practical to use for dictation.
-   **Acceptance Criteria:**
    -   [ ] A performance-tuning Git branch is created.
    -   [ ] Experiments with model quantization (e.g., `ctranslate2`) are conducted and benchmarked.
    -   [ ] Voice Activity Detection (VAD) is implemented to prevent processing silence.
    -   [ ] The solution providing the best performance improvement is merged into the `develop` branch.

**Story 5.2: Automated Deployment**
-   **As a developer,** I want a simple script to deploy code updates to the Pi **so that** I can iterate quickly and reliably.
-   **Acceptance Criteria:**
    -   [ ] A `deploy.sh` script is created in the repository.
    -   [ ] The script successfully pulls the latest `main` branch, and syncs Python dependencies using `uv pip sync`.

**Story 5.3: Automatic Startup Service**
-   **As a user,** I want the dictation application to start automatically on boot **so that** the device is a true standalone appliance.
-   **Acceptance Criteria:**
    -   [ ] A `systemd` service file for the application is created.
    -   [ ] The service is configured to run `main.py` on boot.
    -   [ ] After a reboot, the application is running and the device is ready to transcribe.

---

### Epic 6: Finalization

**Story 6.1: Continuous Integration Pipeline**
-   **As a developer,** I want a CI pipeline in GitHub Actions **so that** code quality is automatically verified for every pull request, preventing bad merges.
-   **Acceptance Criteria:**
    -   [ ] A `.github/workflows/ci.yml` file is created.
    -   [ ] The workflow triggers on PRs to `develop`.
    -   [ ] The workflow successfully installs dependencies and runs `ruff check`.

**Story 6.2: Comprehensive Documentation**
-   **As a future developer or user,** I want clear, comprehensive documentation **so that** I can understand, set up, and use or contribute to the project easily.
-   **Acceptance Criteria:**
    -   [ ] The `README.md` contains full setup instructions for the dev container.
    -   [ ] The `README.md` explains how to deploy and run the application on the Pi.
    -   [ ] The code is well-commented, explaining complex parts.