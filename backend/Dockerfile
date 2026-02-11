# ============================
# STAGE 1: Python Builder
# ============================
FROM python:3.11-slim AS builder

ENV MUSIC21_DISABLE_CORPUS_LOADING=true
ENV MUSIC21_DISABLE_DOWNLOADS=true

WORKDIR /build

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    libsndfile1 \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .

RUN pip install --no-cache-dir --upgrade pip \
    && pip install --no-cache-dir \
       torch==2.7.1+cpu \
       --index-url https://download.pytorch.org/whl/cpu \
       --target=/python-deps \
    && pip install --no-cache-dir \
       -r requirements.txt \
       --target=/python-deps \
    \
    && rm -rf /root/.cache /root/.local /root/.music21 \
    && find /python-deps -type d -name ".cache" -exec rm -rf {} +

# ============================
# STAGE 2: Runtime (LEAN)
# ============================
FROM python:3.11-slim

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
ENV MUSIC21_DISABLE_CORPUS_LOADING=true
ENV MUSIC21_DISABLE_DOWNLOADS=true

WORKDIR /app

RUN apt-get update && apt-get install -y --no-install-recommends \
    ffmpeg \
    fluidsynth \
    libsndfile1 \
    && rm -rf /var/lib/apt/lists/*

RUN mkdir -p /app/data

COPY --from=builder /python-deps /usr/local/lib/python3.11/site-packages
COPY app/ /app/app
COPY config.json /app/data/config.json

RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser

EXPOSE 8000

CMD ["python", "-m", "uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
