FROM --platform=linux/arm64 debian:bookworm-slim

RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    ffmpeg \
    libasound2-dev \
    patchelf \
    pipx \
    && rm -rf /var/lib/apt/lists/*

RUN pipx install uv

ENV PATH="/root/.local/bin:${PATH}"