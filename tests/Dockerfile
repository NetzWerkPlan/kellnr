##################################################
#  Docker image for integration tests of Kellnr  #
#                                                #
#  Build with:                                   #
#  docker build -t kellnr-test:local ../         #
##################################################

#################
#  Build image  #
#################

# ----------- Builder stage -----------
FROM rust:slim-bookworm AS builder

ARG DEBIAN_FRONTEND=noninteractive
ENV TZ=Europe/Berlin

# Install musl-tools and build dependencies for static linking
RUN apt-get update && apt-get install -y musl-tools perl clang make gcc pkg-config curl

# Set environment variables for static linking
ENV OPENSSL_STATIC=1
ENV SQLITE3_STATIC=1

# Add musl target
RUN rustup target add x86_64-unknown-linux-musl

# Install latest nodejs version with nvm
ENV NVM_DIR="/root/.nvm"
ENV NODE_VERSION="23.2.0"
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash \
  && . "$NVM_DIR/nvm.sh" \
  && nvm install $NODE_VERSION \
  && nvm alias default $NODE_VERSION \
  && nvm use $NODE_VERSION
ENV NODE_PATH="$NVM_DIR/versions/node/$NODE_VERSION/lib/node_modules"
ENV PATH="$NVM_DIR/versions/node/v$NODE_VERSION/bin:$PATH"

COPY . /sources

# Build the UI
WORKDIR /sources/ui
RUN npm install
RUN npm run build

# Build kellnr (static musl)
WORKDIR /sources
RUN cargo build --release --target x86_64-unknown-linux-musl --features vendored-openssl

# ----------- Distroless stage -----------
FROM gcr.io/distroless/static-debian12:nonroot

WORKDIR /opt/kellnr

COPY --from=builder /sources/target/x86_64-unknown-linux-musl/release/kellnr ./kellnr
COPY --from=builder /sources/config/default.toml ./config/default.toml
COPY --from=builder /sources/ui/dist ./static
COPY --from=builder /sources/tests/ca.crt /usr/local/share/ca-certificates/ca.crt

EXPOSE 8000

ENTRYPOINT ["/opt/kellnr/kellnr"]
