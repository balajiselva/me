FROM python:3.13-slim-bookworm AS builder

WORKDIR /mediafusion

# Install build dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends git curl build-essential && \
    pip install --upgrade pip && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Create mediafusion user
RUN groupadd -r mediafusion && \
    useradd --no-log-init -r -g mediafusion -m -d /home/mediafusion mediafusion

RUN chown -R mediafusion:mediafusion /mediafusion
USER mediafusion

# Add Python/uv to PATH
ENV PATH="/home/mediafusion/.local/bin:$PATH" \
    PYTHONUNBUFFERED=1

# Copy dependency files first for caching
COPY --chown=mediafusion:mediafusion pyproject.toml uv.lock /mediafusion/

# Install uv and dependencies
COPY --from=ghcr.io/astral-sh/uv:0.6.3 /uv /uvx /bin/
RUN --mount=type=cache,target=/mediafusion/.cache/uv \
    uv sync --frozen --no-install-project --compile-bytecode

# ---------------- Runtime Stage ----------------
FROM python:3.13-slim-bookworm AS runtime

ARG VERSION
WORKDIR /mediafusion

# Install runtime dependencies (add ffmpeg/git if needed)
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Create user
RUN groupadd -r mediafusion && \
    useradd --no-log-init -r -g mediafusion -m -d /home/mediafusion mediafusion

# Copy virtualenv from builder
COPY --from=builder --chown=mediafusion:mediafusion /mediafusion/.venv /mediafusion/.venv

# Add venv to PATH
ENV PATH="/mediafusion/.venv/bin:$PATH" \
    VERSION=${VERSION}

# Copy project files
COPY --chown=mediafusion:mediafusion . /mediafusion

# Ensure startup script is executable
RUN chmod +x /mediafusion/deployment/startup.sh

USER mediafusion

EXPOSE 8000
EXPOSE 9191

CMD ["/mediafusion/deployment/startup.sh"]
