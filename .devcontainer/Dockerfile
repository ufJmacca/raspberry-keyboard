FROM --platform=linux/arm64 debian:bookworm-slim

RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    ffmpeg \
    libasound2-dev \
    patchelf \
    && rm -rf /var/lib/apt/lists/*
    