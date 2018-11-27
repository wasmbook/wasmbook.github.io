# Contributing to `wasm-bindgen`

This section contains instructions on how to get this project up and running for
development.

## Prerequisites

1. Rust Nightly. [Install Rust]. Once Rust is installed, run

    ```shell
    rustup default nightly
    rustup target add wasm32-unknown-unknown
    ```

[install Rust]: https://www.rust-lang.org/en-US/install.html

2. The tests for this project use Node. Make sure you have node >= 10 installed,
   as that is when WebAssembly support was introduced. [Install Node].

[Install Node]: https://nodejs.org/en/
