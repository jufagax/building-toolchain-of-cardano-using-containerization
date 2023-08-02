# Use an official Haskell image with GHC (Glasgow Haskell Compiler) and Cabal pre-installed
FROM haskell:8.10.4 as builder

# Set a working directory
WORKDIR /build

# Install required system dependencies
RUN apt-get update && apt-get install -y \
    automake \
    build-essential \
    curl \
    git \
    libffi-dev \
    libgmp-dev \
    libncurses-dev \
    libnuma-dev \
    libssl-dev \
    libtinfo-dev

# (Optional) If you want to use the pre-installed libsodium, ensure it's present in your host system,
# and mount it into the container during build using the following line:
#COPY CARDANO-BUILD

# Clone the cardano-node repository
RUN git clone https://github.com/input-output-hk/cardano-node

# Build and install cardano-node and cardano-cli
RUN cd cardano-node \
    && git fetch --all --tags \
    && git checkout 8.1.2   # Replace <CARDANO_NODE_VERSION> with the desired version of cardano-node
RUN cd cardano-node \
    && cabal configure -O0 --with-compiler=ghc-8.10.4 \
    && echo "package cardano-crypto-praos" >> cabal.project.local \
    && echo "  flags: -external-libsodium-vrf" >> cabal.project.local \
    && cabal build -j cardano-node cardano-cli \
    && cp -p dist-newstyle/build/x86_64-linux/ghc-8.10.4/cardano-node-8.1.2/x/cardano-node/build/cardano-node/cardano-node /usr/local/bin/ \
    && cp -p dist-newstyle/build/x86_64-linux/ghc-8.10.4/cardano-cli-8.1.2/x/cardano-cli/build/cardano-cli/cardano-cli /usr/local/bin/

# Use a smaller base image for the final container
FROM ubuntu:20.04

# Copy the built binaries from the builder stage
COPY --from=builder /usr/local/bin/cardano-node /usr/local/bin/cardano-node
COPY --from=builder /usr/local/bin/cardano-cli /usr/local/bin/cardano-cli

# Expose any necessary ports (e.g., if the Cardano Node requires network connectivity)

# Set the entrypoint to run the cardano-node command by default
ENTRYPOINT ["cardano-node"]
