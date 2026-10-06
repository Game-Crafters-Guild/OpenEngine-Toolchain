# OpenEngine Toolchain

Prebuilt host binaries of the two tools the OpenEngine shader cook (`Tools/ShaderCook/shadercook.py`) runs to turn the engine's GLSL into WGSL for the WebGPU backend:

| Tool | What it does in the cook | Upstream | License |
|---|---|---|---|
| naga | Translates SPIR-V (from glslc) to WGSL. | `naga-cli` 29.0.4 on crates.io, from [gfx-rs/wgpu](https://github.com/gfx-rs/wgpu) | MIT OR Apache-2.0 ([LICENSES/naga.txt](LICENSES/naga.txt)) |
| tint | Validates the emitted WGSL; the cook's conformance gate. | [Dawn](https://dawn.googlesource.com/dawn) at commit `36cf1fae0cd8a81a4fb4580751648b80b2e6255c` | BSD-3-Clause ([LICENSES/tint.txt](LICENSES/tint.txt)) |

Neither project publishes a prebuilt binary for these hosts, and a multi-megabyte binary per host does not belong in git history, so the binaries are assets of a release of this repository. The repository itself holds only this README and the license notices.

## Version pins

Both pins are load-bearing; bump each only together with the engine's manifest.

- **naga 29.0.4.** naga 29.x is the WGSL frontend inside the engine's pinned wgpu-native v29.0.1.1, so WGSL this translator emits is WGSL the desktop WebGPU backend validates. Bump naga and wgpu-native together, never separately.
- **tint at Dawn `36cf1fae0`.** Tint is the WGSL frontend inside Chrome: what it accepts is what ships. It is the cook's second gate after naga; it errors on uniformity violations naga only warns about.

## Release `shadercook-toolchain-v1`

| Host | Asset | Size (bytes) | sha256 |
|---|---|---|---|
| windows-x64 | `naga-29.0.4-windows-x64.exe` | 5151232 | `1251ee41550a56474808855e424927ad3ee3820a2f92c1789dc23dd009a7d90e` |
| windows-x64 | `tint-36cf1fae0-windows-x64.exe` | 3202048 | `58623e83b49240cd05daaaf00174e760660d467934c68d71b420925bb09b425f` |
| macos-arm64 | `naga-29.0.4-macos-arm64` | 5397856 | `8312eba5c64fa2e264ad8872bbf775aadefb973112cf9b0f1ce6101be944c855` |
| macos-arm64 | `tint-36cf1fae0-macos-arm64` | 4245256 | `fa3974be525e7abe04fd746abd74213203623fc1972a19d98f3264fda87086d2` |

linux-x64 and macos-x64 have no hosted build yet. To add a host, build both tools with the recipes below, upload them to the release as `naga-29.0.4-<host>` and `tint-36cf1fae0-<host>`, and record their asset id, size and sha256 in the engine's manifest.

## Build recipes

- **naga:** `cargo install naga-cli --version '^29' --locked`
- **tint:** shallow clone of Dawn at the pinned commit, `tools/fetch_dawn_dependencies.py`, a Release CMake build with Ninja of the target `tint_cmd_tint_cmd` with all GPU backends off.

## How the engine consumes the release

The engine's `Tools/ShaderCook/toolchain/manifest.json` names this repository, the release tag, and per host each asset's id, size and sha256. `python Tools/ShaderCook/toolchain/setup.py` downloads the running host's assets through the GitHub API by asset id, verifies each against its sha256, and places them in `Tools/ShaderCook/toolchain/<host>/` (gitignored), where the cook looks for them. The repository is public, so the download needs no GitHub account or token. A file that already hashes correctly is left alone, so rerunning the script is cheap.
