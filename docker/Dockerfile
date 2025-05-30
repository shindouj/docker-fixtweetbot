# Step 1: Builder

FROM python:3.13-alpine AS builder
WORKDIR /usr/local/app

RUN apk add --no-cache git linux-headers cargo rust
RUN git clone https://github.com/Kyrela/FixTweetBot.git .

RUN python -m venv /usr/local/venv && . /usr/local/venv/bin/activate
RUN /usr/local/venv/bin/pip install --no-cache-dir -r requirements.txt && \
    /usr/local/venv/bin/pip install --no-cache-dir cryptography

# Step 2: Final image

FROM python:3.13-alpine
WORKDIR /usr/local/app

ENV USR=app
ENV GRP=$USR
ENV UID=1000
ENV GID=1000

ENV PATH="/usr/local/venv/bin:$PATH" 
ENV DISCORE_CONFIG=/usr/local/app/docker.config.yml
ENV DATABASE_DRIVER="mysql"

# needed for console logging
ENV PYTHONUNBUFFERED=1 
ENV PYTHONDONTWRITEBYTECODE=1

RUN apk add --no-cache netcat-openbsd bash

RUN addgroup \
    --gid "$GID" \
    $GRP \
    && \
    adduser \
    --disabled-password \
    --no-create-home \
    --home "$(pwd)" \
    --uid "$UID" \
    --ingroup "$GRP" \
    $USR

COPY --from=builder /usr/local/venv /usr/local/venv
COPY --from=builder /usr/local/app /usr/local/app

RUN chown -R $USR:$GRP .

COPY docker/docker-entrypoint.sh /
RUN chmod +x /docker-entrypoint.sh

USER $USR

ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["python", "main.py"]
