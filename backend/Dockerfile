# Stage 1 — Билдер
FROM python:3.12-slim AS builder
WORKDIR /app
COPY app.py .
RUN python -m py_compile app.py

# Stage 2 — Рантайм

FROM python:3.12-slim AS runtime
RUN useradd --no-create-home --shell /bin/false appuser
WORKDIR /app
COPY --from=builder /app/app.py .
USER appuser

# EXPOSE 8080  намеренно закомментировано

CMD ["python", "app.py"]