# opusenc-wasm

The opusenc (from [opus-tools](https://github.com/xiph/opus-tools)) build for wasm web workers.

## Building

1. Clone repository:
    ```
    git clone --depth 1 --recurse-submodules https://github.com/n3tael/opusenc-wasm.git
    ```

2. Apply patch for [lossless2opus](https://github.com/n3tael/lossless2opus) (optional):
    ```
    cd external/opus-tools && git apply ../../progress.patch
    ```

3. Create `build` directory:
    ```
    mkdir build && cd build
    ```

4. Configure project:
    ```
    emcmake cmake ..
    ```

5. Build!
    ```
    ninja
    ```

6. `opusenc.js` and `opusenc.wasm` will appear in `build` directory.

## License

Unlicense.