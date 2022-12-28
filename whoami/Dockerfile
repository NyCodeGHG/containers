FROM golang:1-alpine as builder

RUN apk --no-cache --no-progress add git ca-certificates tzdata make \
    && update-ca-certificates \
    && rm -rf /var/cache/apk/*

WORKDIR /go/whoami

# Download go modules
COPY go.mod .
COPY go.sum .
RUN ls
RUN GO111MODULE=on GOPROXY=https://proxy.golang.org go mod download

COPY . .

RUN GOFLAGS="-buildvcs=false" make build

# Create the User here, because this is still an alpine image
RUN adduser whoami --disabled-password --no-create-home

# Create a minimal container to run a Golang static binary
FROM scratch

COPY --from=builder /etc/passwd /etc/passwd
COPY --from=builder /usr/share/zoneinfo /usr/share/zoneinfo
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=builder /go/whoami/whoami .

# Bind to 8080 because we are running as non root.
ENV WHOAMI_PORT_NUMBER 8080

USER whoami

ENTRYPOINT ["/whoami"]
EXPOSE 8080
