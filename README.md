# Gamescope (x86-64-v4 optimized)

Custom builds of [Gamescope](https://github.com/ValveSoftware/gamescope) with `-march=x86-64-v4 -O3` and related flags for AMD Zen 4 CPUs.

Built from upstream source with Clang, targeting Devuan Excalibur (Debian Trixie base). Ships as a `.deb` package.

## Requirements

- **Distribution:** Devuan Excalibur (or Debian Trixie)
- **Architecture:** amd64
- **CPU:** with **AVX-512** support (x86-64-v4)
  - AMD Zen 4 (Ryzen 7000/8000/9000)
- **GPU:** AMD (RDNA 1/2/3/4) — tested on Radeon 780M (Phoenix)

The binary will crash with `Illegal instruction` on CPUs without AVX-512. Check support:

```bash
grep -o 'avx512[a-z]*' /proc/cpuinfo | sort -u | head
```

If the output is empty — **do not install this package**.

## Installation

1. Download the `.deb` from the [Releases](../../releases) page.

2. Install:

   ```bash
   sudo apt install ./gamescope_*.deb
   ```

3. Verify:

   ```bash
   gamescope --version
   ```

## Package notes

- Built **only for Devuan Excalibur / Debian Trixie**. Installing on older releases may break due to library version mismatches (`libwlroots-0.18`, `libdrm 2.4.124`, `pixman 0.44`, `wayland-server 1.23.1`).
- **Not signed.** Verify integrity using SHA-256 from the release description.
- Debug symbols are stripped; `dbgsym`/`ddeb` packages are not produced.

## Build Architecture

- **Compiler:** Clang 19 (Devuan)
- **Build flags:** `-march=x86-64-v4, -mtune=znver4, -O3, -fno-plt, -fomit-frame-pointer, -falign-functions=32, -falign-loops=32`
- **Meson options:**
    - `-Dpipewire=enabled`
    - `-Denable_openvr_support=false`

## How It Is Built

GitHub Actions workflow runs inside a `devuan/devuan:excalibur` container:

1. Installs Clang, LLD, Meson, Ninja, `debhelper`, `ccache`, and runtime dependencies.
2. Downloads Debian packaging (`debian/`) from the pool matching the base version.
3. Clones Gamescope source at the requested tag.
4. Applies patches and `override_dh_dwz` (Clang generates DWARF-5, unsupported by `dwz`).
5. Configures with Meson, builds via `dh_auto_build`.
6. Packages via `dpkg-buildpackage -b -us -uc`.
7. Uploads the resulting `.deb` as an artifact and attaches it to a release.

Source workflow: [`.github/workflows/build.yml`](.github/workflows/build.yml).

## Companion projects

For a complete optimized graphics stack on AMD Zen 4:

| Project | Purpose |
|---|---|
| [`mesa-optimized`](https://github.com/nafigator/mesa-optimized) | Mesa (radeonsi, RADV) |
| [`dxvk-optimized`](https://github.com/nafigator/dxvk-optimized) | DXVK (D3D9/10/11 → Vulkan) |
| [`vkd3d-proton-optimized`](https://github.com/nafigator/vkd3d-proton-optimized) | VKD3D-Proton (D3D12 → Vulkan) |
| [`wine-optimized`](https://github.com/nafigator/wine-optimized) | Wine WoW64 (Clang, llvm-mingw) |

## Notes

- Not affiliated with the upstream project. Report build-specific issues in this repository's [Issues](../../issues) tracker.
- Runtime performance of Gamescope is limited by the micro-compositor's CPU-side overhead. The main FPS gains in games come from the graphics stack (Mesa, DXVK, VKD3D-Proton), not from Gamescope itself.
- The package is built against the system `libwlroots-0.18`, `libdrm 2.4.124`, `pixman 0.44`, and `wayland-server 1.23.1` from Devuan Excalibur. On a host with different library versions, the binary may fail to load.

## License

Build scripts and workflows in this repository are licensed under the MIT License.

Gamescope itself is distributed under the [BSD-2-Clause / BSD-3-Clause](https://github.com/ValveSoftware/gamescope/blob/master/LICENSE). The compiled `.deb` in Releases is a redistribution of Gamescope under its original license.
