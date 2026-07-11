# ---- Stage 1 : build / install des dépendances ----
FROM python:3.12-slim AS build

WORKDIR /app

# --user installe dans ~/.local, ce qui se copie proprement dans le stage
# runtime sans traîner les outils de compilation ni le cache pip.
COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

COPY app/ ./app/

# ---- Stage 2 : runtime ----
FROM python:3.12-slim AS runtime

LABEL org.opencontainers.image.source="https://github.com/chesaad/devhub-campus"

WORKDIR /app

RUN useradd --uid 1001 --system --no-create-home app

COPY --from=build /root/.local /home/app/.local
COPY --from=build /app/app ./app

ENV PATH=/home/app/.local/bin:$PATH \
    PORT=8080 \
    LOG_LEVEL=info \
    PYTHONUNBUFFERED=1

RUN chown -R app:app /home/app/.local
USER 1001

EXPOSE 8080

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8080"]
