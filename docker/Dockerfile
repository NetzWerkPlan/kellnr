# ----------- Builder stage -----------
FROM rust:slim-bookworm AS builder

# Install musl-tools and build dependencies for static linking
RUN apt-get update && apt-get install -y musl-tools perl clang make gcc pkg-config curl

# Install nodejs and npm for UI build
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && apt-get install -y nodejs

# Set environment variables for static linking
ENV OPENSSL_STATIC=1
ENV SQLITE3_STATIC=1

# Add musl target
RUN rustup target add x86_64-unknown-linux-musl

WORKDIR /app
COPY . .

# Build the UI
WORKDIR /app/ui
RUN npm install
RUN npm run build

# Build the release binary for musl (static)
WORKDIR /app
RUN cargo build --release --target x86_64-unknown-linux-musl --features vendored-openssl

# ----------- Distroless stage -----------
FROM gcr.io/distroless/static-debian12:nonroot

WORKDIR /opt/kellnr

# Copy the statically linked binary
COPY --from=builder /app/target/x86_64-unknown-linux-musl/release/kellnr ./kellnr
COPY --from=builder /app/config/default.toml ./config/default.toml
COPY --from=builder /app/ui/dist ./static

EXPOSE 8000

ENTRYPOINT ["/opt/kellnr/kellnr"]
