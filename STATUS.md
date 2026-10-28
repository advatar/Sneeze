# Local macOS build

- [x] Build the Release dependency set, including both Wasmtime architectures.
- [x] Make the OpenXR static loader independent of system JsonCpp.
- [x] Use native Metal for Vox on Apple platforms without host Vulkan linkage.
- [x] Rebuild the affected dependencies and Sneeze's tools and tests.
- [x] Run local core tests and verify integration with Rubidium.

The initial build compiled `libSneeze.a`, but SneezeTest and SignMsf failed to link
because OpenXR selected Homebrew JsonCpp. Vox also selected Homebrew's ARM-only
Vulkan loader for a universal build. Build recipes must select dependencies that
work for every requested architecture.

The Wasmtime test-fixture submodule download stalled. Stopping that download and
retrying `scripts/build-macos.sh --all` used the existing source checkout and
successfully built Wasmtime for arm64 and x86_64.

GitHub issue creation was attempted, but `advatar/Sneeze` has issues disabled
(HTTP 410).

Build fixes:

- OpenXR uses vendored JsonCpp. On Apple it also skips Vulkan discovery, whose
  Homebrew include directory otherwise shadows the vendored JSON headers.
- Vox uses Metal on Apple in both its dependency build and consumer discovery.
  This removes the ARM-only system Vulkan loader from universal executable links.

Validation on 2026-10-28:

- Rebuilt OpenXR and Vox with `scripts/build-macos.sh --only <dep> --rebuild`.
  The final OpenXR retry used `--only openxr-sdk` after correcting header lookup.
- `CMAKE_BUILD_PARALLEL_LEVEL=12 CARGO_BUILD_JOBS=12 ./scripts/build-macos.sh`
  completed successfully, including SneezeTest, SignMsf, and GenCerts.
- Rubidium's Release build completed against these dependencies. Its running
  app initialized Metal on Apple M5 Max, instantiated WASM modules, and loaded
  glTF scene content.
- Wasm, Chrono, SPIR-V, UI, compute, Vox, and console tests passed (244 checks).
- The JWS suite aborted on malformed input with `Invalid input: too much fill`.
  The storage suite crashed at startup. glTF passed 167 checks and failed 2
  because sample GLB fixtures were missing. These failures remain unresolved;
  the test suite is not fully passing.

Logs are retained in `builds/macos-universal/logs/`. No simulator tests were run.
