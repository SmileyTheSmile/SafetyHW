# syntax=docker/dockerfile:1

FROM golang:1.21.1 AS build-stage
WORKDIR /app

# Always needed, never changes => should be done first for caching purposes
RUN go install github.com/jackc/tern/v2@latest 
# Copy package management files to the WORKDIR
COPY go.mod go.sum ./
# Install dependencies
RUN go mod download
# Copy the rest of the project to the WORKDIR
COPY ./. ./

# Build the binary to copy it to the slim image
RUN CGO_ENABLED=0 GOOS=linux go build -o api ./cmd/app/api.go


FROM alpine:latest AS build-release-stage

RUN apk --no-cache add  \
        gcompat         \
        libstdc++       \
        bash

RUN addgroup --system nonroot
RUN adduser --system nonroot --ingroup nonroot

COPY --from=build-stage ./app/db ./db
COPY --from=build-stage ./go/bin/tern ./tern
COPY --from=build-stage ./app/api ./api

EXPOSE 8080
EXPOSE 5432

USER nonroot:nonroot