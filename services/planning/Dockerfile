# ---- Stage 1 : build / install des dépendances ----
# alpine plutôt que slim (debian) : python:3.12-slim pèse ~160 Mo à lui seul
# une fois l'interpréteur installé, ce qui dépasse déjà le budget de 200 Mo
# de l'image finale. alpine (musl) tient la contrainte, au prix de wheels
# potentiellement recompilées depuis les sources (gcc ci-dessous).
FROM python:3.12-alpine AS build

WORKDIR /app

RUN apk add --no-cache gcc musl-dev libffi-dev

# --user installe dans ~/.local, ce qui se copie proprement dans le stage
# runtime sans traîner les outils de compilation ni le cache pip.
COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

COPY app/ ./app/

# ---- Stage 2 : runtime ----
FROM python:3.12-alpine AS runtime

LABEL org.opencontainers.image.source="https://github.com/chesaad/devhub-campus"

WORKDIR /app

RUN addgroup -g 1001 -S app && adduser -u 1001 -S app -G app

# --chown ici évite une 2e couche qui recopierait /home/app/.local en entier
# (un `RUN chown -R` après un COPY double le poids de tout ce qu'il touche).
COPY --from=build --chown=app:app /root/.local /home/app/.local
COPY --from=build --chown=app:app /app/app ./app

ENV PATH=/home/app/.local/bin:$PATH \
    PORT=8080 \
    LOG_LEVEL=info \
    PYTHONUNBUFFERED=1

USER 1001

EXPOSE 8080

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8080"]
