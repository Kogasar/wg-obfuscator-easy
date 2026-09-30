# ---------------------------------------------------------
# Build frontend
# ---------------------------------------------------------
FROM node:20 AS frontend

WORKDIR /frontend

# Dockerfile находится в patch/, но COPY работает
# относительно build context (корня репозитория)
COPY frontend/package*.json ./
RUN npm install

COPY frontend/ ./
RUN npm run build


# ---------------------------------------------------------
# WireGuard Obfuscator
# ---------------------------------------------------------
FROM clustermeerkat/wg-obfuscator:1.6 AS obf


# ---------------------------------------------------------
# Main container
# ---------------------------------------------------------
FROM debian:trixie-slim

WORKDIR /app

ENV PIP_BREAK_SYSTEM_PACKAGES=1
ENV PIP_ROOT_USER_ACTION=ignore

# wg-obfuscator
COPY --from=obf /app/wg-obfuscator /usr/bin/wg-obfuscator

# Backend
COPY backend /app

# Frontend, собранный на первом этапе
COPY --from=frontend /static /app/static

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        tini \
        iptables \
        iproute2 \
        procps \
        wireguard \
        python3 \
        python3-pip \
        python3-packaging && \
    pip install --no-cache-dir -r requirements.txt && \
    apt-get purge -y python3-pip python3-wheel && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    rm -rf /tmp/* /var/tmp/* && \
    find /usr/lib/python3* -type d -name __pycache__ -exec rm -r {} + 2>/dev/null || true && \
    find /usr/lib/python3* -type f -name "*.pyc" -delete 2>/dev/null || true && \
    find /usr/lib/python3* -type f -name "*.pyo" -delete 2>/dev/null || true

# Replace wg-quick with patched version
COPY --chmod=755 patch/wg-quick /usr/bin/wg-quick

HEALTHCHECK --interval=30s --timeout=5s --start-period=20s --retries=3 \
    CMD python3 -c "import os,urllib.request; p=os.getenv('WEB_PREFIX','').strip(); p=('/'+p if p and not p.startswith('/') else p).rstrip('/'); urllib.request.urlopen('http://127.0.0.1:5000'+p+'/health')"

ENTRYPOINT ["tini", "--", "./docker-entrypoint.sh"]
