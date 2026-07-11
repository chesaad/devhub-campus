# ---- Stage 1 : build du binaire Go ----
FROM golang:1.22-alpine AS build

WORKDIR /src

COPY go.mod ./
RUN go mod download

COPY cmd/ ./cmd/
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o /out/notif ./cmd

# ---- Stage 2 : runtime minimal ----
# distroless:nonroot plutôt que scratch : on garde /etc/passwd (uid 65532
# "nonroot" déjà déclaré) et les certs CA, sans le poids d'alpine.
FROM gcr.io/distroless/static:nonroot AS runtime

LABEL org.opencontainers.image.source="https://github.com/chesaad/devhub-campus"

COPY --from=build /out/notif /notif

ENV PORT=8080 \
    LOG_LEVEL=info

EXPOSE 8080

# USER nonroot:nonroot déjà fourni par l'image distroless
ENTRYPOINT ["/notif"]
