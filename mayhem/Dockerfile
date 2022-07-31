# Build Stage
FROM ghcr.io/evanrichter/cargo-fuzz:latest as builder

## Add source code to the build stage.
ADD . /src
WORKDIR /src

RUN echo building instrumented harnesses && \
    bash -c "pushd testsuite/aptos-fuzzer/fuzz && cargo +nightly -Z sparse-registry fuzz build && popd" && \
    mv testsuite/aptos-fuzzer/fuzz/target/x86_64-unknown-linux-gnu/release/fuzz_runner /fuzz_runner && \
    mv testsuite/aptos-fuzzer/fuzz/target/x86_64-unknown-linux-gnu/release/fuzzer_builder /fuzzer_builder && \
    echo done

RUN echo building non-instrumented harnesses && \
    export RUSTFLAGS="--cfg fuzzing -Clink-dead-code -Cdebug-assertions -C codegen-units=1" && \
    bash -c "pushd testsuite/aptos-fuzzer/fuzz && cargo +nightly -Z sparse-registry build --release && popd" && \
    mv testsuite/aptos-fuzzer/fuzz/target/release/fuzz_runner /fuzz_runner_no_inst && \
    mv testsuite/aptos-fuzzer/fuzz/target/release/fuzzer_builder /fuzzer_builder_no_inst && \
    echo done

# Package Stage
FROM rustlang/rust:nightly

COPY --from=builder /fuzz_runner /fuzz_runner_no_inst /
COPY --from=builder /fuzzer_builder /fuzzer_builder_no_inst /
