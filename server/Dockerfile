# Stage 1: Build the Rust Wasm module
FROM rust:1.87.0 AS builder

# Install SpacetimeDB CLI with non-interactive mode
RUN mkdir -p /root/.spacetimedb
RUN curl -sSf https://install.spacetimedb.com | DEBIAN_FRONTEND=noninteractive sh -s -- --root-dir /root/.spacetimedb --yes
ENV PATH="/root/.spacetimedb/:${PATH}"
# Set the SpacetimeDB version explicitly to 1.1.2
RUN spacetime version install 1.1.2
RUN spacetime version use 1.1.2
# Install wasm32-unknown-unknown target
RUN rustup target add wasm32-unknown-unknown

WORKDIR /app

# Copy only Cargo.toml first (we don't have Cargo.lock yet)
COPY Cargo.toml ./
# Create a dummy lib.rs to cache dependencies
RUN mkdir src
COPY src/lib.rs ./src/
RUN cargo build --release --target wasm32-unknown-unknown

# Copy the actual source code
COPY src ./src

# Build the Wasm module
RUN cargo build --release --target wasm32-unknown-unknown

# Stage 2: Run the SpacetimeDB server
FROM rust:1.87.0

# Install SpacetimeDB CLI with non-interactive mode
RUN mkdir -p /root/.spacetimedb
RUN curl -sSf https://install.spacetimedb.com | DEBIAN_FRONTEND=noninteractive sh -s -- --root-dir /root/.spacetimedb --yes
ENV PATH="/root/.spacetimedb/:${PATH}"
# Set the SpacetimeDB version explicitly to 1.1.2
RUN spacetime version install 1.1.2
RUN spacetime version use 1.1.2
# Install wasm32-unknown-unknown target
RUN rustup target add wasm32-unknown-unknown

WORKDIR /app

# Copy the entire project structure including the built WASM
COPY --from=builder /app .

# Expose SpacetimeDB ports
EXPOSE 3000
EXPOSE 3001/udp

# The command to start the server will be provided by docker-compose
# Example: spacetimedb start --db-name $DB_NAME --module-path target/wasm32-unknown-unknown/release/spacetime_phaser_server.wasm
# The DB_NAME will come from the .env file via docker-compose