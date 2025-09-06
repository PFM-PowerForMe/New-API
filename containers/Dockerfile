FROM docker.io/library/alpine:edge AS git
ARG TAG
ENV TAG=${TAG:-main}
RUN apk update && apk add --no-cache \
	git
WORKDIR /
RUN git -c advice.detachedHead=false clone --branch $TAG --depth=1 --recurse-submodules https://github.com/pfm-powerforme/new-api.git source
WORKDIR /source/


FROM docker.io/oven/bun:latest AS frontend
ARG TAG
ENV TAG=${TAG:-main}
COPY --from=git /source/web/ /frontend/
WORKDIR /frontend/
RUN bun install
RUN DISABLE_ESLINT_PLUGIN='true' VITE_REACT_APP_VERSION=$TAG bun run build


FROM docker.io/library/golang:alpine AS backend
ARG TAG
ENV TAG=${TAG:-main}
ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux
COPY --from=git /source/ /backend/
WORKDIR /backend/
COPY --from=frontend /frontend/dist web/dist
RUN go build -ldflags "-s -w -X 'one-api/common.Version=$TAG'" -o one-api


FROM docker.io/library/alpine:latest AS runtime
RUN apk update && apk add --no-cache \
    ca-certificates tzdata ffmpeg \
    && update-ca-certificates
COPY --from=backend /backend/one-api /one-api
EXPOSE 3000
WORKDIR /data
ENTRYPOINT ["/one-api"]
