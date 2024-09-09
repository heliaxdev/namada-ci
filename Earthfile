VERSION 0.8

namada:
  FROM rust:1.78.0-bookworm

  WORKDIR /__w/namada/namada

  ARG toolchain=1.78.0
  ARG nightly_toolchain=nightly-2024-05-15
  ARG rocksdb_version=8.10.0
  ARG gaia_version=19.1.0
  ARG cometbft_version=0.37.9
  ARG wasm_opt_version=118

  RUN apt-get update -y
  RUN apt-get install -y protobuf-compiler 
  RUN apt-get install -y build-essential 
  RUN apt-get install -y clang-tools clang
  RUN apt-get install -y libudev-dev
  RUN apt-get install -y libssl-dev
  RUN apt-get install -y pkg-config
  RUN apt-get install -y gcc
  RUN apt-get install -y parallel
    
  RUN rustup toolchain install $toolchain-x86_64-unknown-linux-gnu --no-self-update --component clippy,rustfmt,rls,rust-analysis,rust-docs,rust-src,llvm-tools-preview
  RUN rustup target add wasm32-unknown-unknown
  RUN rustup toolchain install $nightly_toolchain-x86_64-unknown-linux-gnu --no-self-update --component clippy,rustfmt,rls,rust-analysis,rust-docs,rust-src,llvm-tools-preview,rustc-codegen-cranelift-preview
  RUN rustup target add --toolchain $nightly_toolchain wasm32-unknown-unknown
  RUN rustup default $toolchain-x86_64-unknown-linux-gnu

  # download masp artifacts
  RUN mkdir -p /masp/.masp-params
  RUN curl -o /masp/.masp-params/masp-spend.params -L https://github.com/anoma/masp-mpc/releases/download/namada-trusted-setup/masp-spend.params\?raw\=true
  RUN curl -o /masp/.masp-params/masp-output.params -L https://github.com/anoma/masp-mpc/releases/download/namada-trusted-setup/masp-output.params?raw=true
  RUN curl -o /masp/.masp-params/masp-convert.params -L https://github.com/anoma/masp-mpc/releases/download/namada-trusted-setup/masp-convert.params?raw=true

  # install cargo binstall 
  RUN curl -L --proto '=https' --tlsv1.2 -sSf https://raw.githubusercontent.com/cargo-bins/cargo-binstall/main/install-from-binstall-release.sh | bash

  # install cargo nextest
  RUN cargo binstall cargo-nextest --no-confirm

  # install sccache
  RUN cargo binstall sccache --no-confirm

  # install cargo cache
  RUN cargo binstall cargo-cache --no-confirm

  # install cargo about
  RUN cargo binstall cargo-about --no-confirm

  # install llvm-cov
  RUN cargo binstall cargo-llvm-cov --no-confirm
  RUN cargo +$nightly_toolchain install cargo-llvm-cov --locked

  # install rustfilt (for fuzzing)
  RUN cargo install rustfilt

  # install fuzz
  RUN cargo install cargo-fuzz

  # download mold
  RUN curl -o mold.tar.gz -LO https://github.com/rui314/mold/releases/download/v2.32.1/mold-2.32.1-x86_64-linux.tar.gz
  RUN tar --strip-components 2 -xvzf mold.tar.gz mold-2.32.1-x86_64-linux/bin/mold
  RUN mv mold /usr/local/bin
  RUN chmod +x /usr/local/bin/mold

  # download gaia 
  RUN curl -o gaiad -LO https://github.com/cosmos/gaia/releases/download/v${gaia_version}/gaiad-v${gaia_version}-linux-amd64
  RUN mv gaiad /usr/local/bin
  RUN chmod +x /usr/local/bin/gaiad

  # download cometbft
  RUN curl -o cometbft.tar.gz -LO https://github.com/cometbft/cometbft/releases/download/v${cometbft_version}/cometbft_${cometbft_version}_linux_amd64.tar.gz
  RUN tar -xvzf cometbft.tar.gz
  RUN mv cometbft /usr/local/bin
  RUN chmod +x /usr/local/bin/cometbft

  # download wasm-opt
  RUN curl -o binaryen.tar.gz -LO https://github.com/WebAssembly/binaryen/releases/download/version_${wasm_opt_version}/binaryen-version_${wasm_opt_version}-x86_64-linux.tar.gz
  RUN tar --strip-components 2 -xvzf binaryen.tar.gz binaryen-version_${wasm_opt_version}/bin/wasm-opt
  RUN mv wasm-opt /usr/local/bin
  RUN chmod +x /usr/local/bin/wasm-opt

  SAVE IMAGE --push ghcr.io/heliaxdev/namada-ci:namada-latest ghcr.io/heliaxdev/namada-ci:namada-main

wasm:
  FROM rust:1.78.0-bookworm

  ARG toolchain=1.78.0
  ARG wasm_opt_version=118

  WORKDIR /__w/namada/namada

  RUN apt-get update -y
  RUN apt-get install -y protobuf-compiler 
  RUN apt-get install -y parallel

  RUN rustup toolchain install $toolchain --profile minimal --no-self-update
  RUN rustup target add wasm32-unknown-unknown
  RUN rustup default $toolchain-x86_64-unknown-linux-gnu

  # install cargo binstall 
  RUN curl -L --proto '=https' --tlsv1.2 -sSf https://raw.githubusercontent.com/cargo-bins/cargo-binstall/main/install-from-binstall-release.sh | bash

  # install cargo cache
  RUN cargo binstall cargo-cache --no-confirm

  # download wasm-opt
  RUN curl -o binaryen.tar.gz -LO https://github.com/WebAssembly/binaryen/releases/download/version_${wasm_opt_version}/binaryen-version_${wasm_opt_version}-x86_64-linux.tar.gz
  RUN tar --strip-components 2 -xvzf binaryen.tar.gz binaryen-version_${wasm_opt_version}/bin/wasm-opt
  RUN mv wasm-opt /usr/local/bin
  RUN chmod +x /usr/local/bin/wasm-opt

  SAVE IMAGE --push ghcr.io/heliaxdev/namada-ci:wasm-latest ghcr.io/heliaxdev/namada-ci:wasm-main

speculos:
  FROM ghcr.io/ledgerhq/speculos

  WORKDIR /__w/namada/namada

  ARG toolchain=1.78.0
  ARG nightly_toolchain=nightly-2024-05-15
  ARG rocksdb_version=8.10.0
  ARG gaia_version=15.2.0
  ARG cometbft_version=0.37.9

  RUN apt-get update -y
  RUN apt-get install curl -y
  RUN apt-get install -y protobuf-compiler 
  RUN apt-get install -y build-essential 
  RUN apt-get install -y clang-tools clang
  RUN apt-get install -y libudev-dev
  RUN apt-get install -y libssl-dev
  RUN apt-get install -y pkg-config
  RUN apt-get install -y gcc
  RUN apt-get install -y parallel

  RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
  RUN rustup toolchain install $toolchain-x86_64-unknown-linux-gnu --no-self-update --component clippy,rustfmt,rls,rust-analysis,rust-docs,rust-src,llvm-tools-preview
  RUN rustup target add wasm32-unknown-unknown
  RUN rustup toolchain install $nightly_toolchain-x86_64-unknown-linux-gnu --no-self-update --component clippy,rustfmt,rls,rust-analysis,rust-docs,rust-src,llvm-tools-preview,rustc-codegen-cranelift-preview
  RUN rustup target add --toolchain $nightly_toolchain wasm32-unknown-unknown
  RUN rustup default $toolchain-x86_64-unknown-linux-gnu

  # download masp artifacts
  RUN mkdir -p /masp/.masp-params
  RUN curl -o /masp/.masp-params/masp-spend.params -L https://github.com/anoma/masp-mpc/releases/download/namada-trusted-setup/masp-spend.params\?raw\=true
  RUN curl -o /masp/.masp-params/masp-output.params -L https://github.com/anoma/masp-mpc/releases/download/namada-trusted-setup/masp-output.params?raw=true
  RUN curl -o /masp/.masp-params/masp-convert.params -L https://github.com/anoma/masp-mpc/releases/download/namada-trusted-setup/masp-convert.params?raw=true

  # install cargo binstall 
  RUN curl -L --proto '=https' --tlsv1.2 -sSf https://raw.githubusercontent.com/cargo-bins/cargo-binstall/main/install-from-binstall-release.sh | bash

  # install cargo nextest
  RUN cargo binstall cargo-nextest --no-confirm

  # install sccache
  RUN cargo binstall sccache --no-confirm

  # install cargo cache
  RUN cargo binstall cargo-cache --no-confirm

  # download mold
  RUN curl -o mold.tar.gz -LO https://github.com/rui314/mold/releases/download/v2.32.1/mold-2.32.1-x86_64-linux.tar.gz
  RUN tar --strip-components 2 -xvzf mold.tar.gz mold-2.32.1-x86_64-linux/bin/mold
  RUN mv mold /usr/local/bin
  RUN chmod +x /usr/local/bin/mold

  # download gaia 
  RUN curl -o gaiad -LO https://github.com/cosmos/gaia/releases/download/v${gaia_version}/gaiad-v${gaia_version}-linux-amd64
  RUN mv gaiad /usr/local/bin
  RUN chmod +x /usr/local/bin/gaiad

  # download cometbft
  RUN curl -o cometbft.tar.gz -LO https://github.com/cometbft/cometbft/releases/download/v${cometbft_version}/cometbft_${cometbft_version}_linux_amd64.tar.gz
  RUN tar -xvzf cometbft.tar.gz
  RUN mv cometbft /usr/local/bin
  RUN chmod +x /usr/local/bin/cometbft

  COPY ./artifacts/app_s2.elf .

  SAVE IMAGE --push ghcr.io/heliaxdev/namada-ci:speculos-latest ghcr.io/heliaxdev/namada-ci:speculos-main
