FROM python:3.12.13-slim-bookworm@sha256:d384a24aebd583543abb00fa47b2a434ec3a96559c6f244cc3cfcbe9e136a798

WORKDIR /app

# Candidate/control-plane image: no bundled scheduler.
# supercronic/cron is intentionally outside this image path until a separate
# runtime scheduler phase is approved and scanned.

# 从官方镜像拷贝 uv 二进制（CI/control-plane 禁止 latest；digest pinned）
COPY --from=ghcr.io/astral-sh/uv:0.11.3@sha256:90bbb3c16635e9627f49eec6539f956d70746c409209041800a0280b93152823 /uv /uvx /bin/
ENV UV_LINK_MODE=copy

# 先安装依赖（利用 Docker 层缓存）
COPY pyproject.toml uv.lock ./
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --locked --no-install-project --no-dev

# 再复制项目代码并安装项目本身
COPY docker/manage.py .
COPY trendradar/ ./trendradar/
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --locked --no-dev && \
    rm -f /bin/uv /bin/uvx

ENV PATH="/app/.venv/bin:$PATH"

# 复制 entrypoint.sh 并强制转换为 LF 格式；最后降权运行
COPY docker/entrypoint.sh /entrypoint.sh.tmp
RUN sed -i 's/\r$//' /entrypoint.sh.tmp && \
    mv /entrypoint.sh.tmp /entrypoint.sh && \
    chmod 0555 /entrypoint.sh && \
    chmod +x manage.py && \
    mkdir -p /app/config /app/output && \
    groupadd --system --gid 10001 trendradar && \
    useradd --system --uid 10001 --gid trendradar --home-dir /app --shell /usr/sbin/nologin trendradar && \
    chown -R trendradar:trendradar /app

ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    CONFIG_PATH=/app/config/config.yaml \
    FREQUENCY_WORDS_PATH=/app/config/frequency_words.txt \
    RUN_MODE=once

USER trendradar

ENTRYPOINT ["/entrypoint.sh"]
