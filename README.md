# Gamescope for Devuan Excalibur (Debian Trixie base)

Optimized [Gamescope](https://github.com/ValveSoftware/gamescope) (micro-compositor) packages for Devuan Excalibur, built for modern CPUs.

## Requirements

- **Distribution:** Devuan Excalibur (stable)
- **Architecture:** amd64
- **CPU:** x86-64-v3 (AVX2) or x86-64-v4 (AVX-512) - pick the matching variant

**Packages will not run** on CPUs without AVX2/AVX-512. Check support:

```bash
grep -o 'avx[0-9_]*' /proc/cpuinfo | sort -u
```

If the output is empty - **do not install these packages**.

## Package

<details>
<summary>Package details</summary>

| Package     | Purpose                                                      |
|-------------|--------------------------------------------------------------|
| `gamescope` | Micro-compositor providing scaling and fullscreen options    |

Debug packages (`*-dbgsym`) are **not included** in releases - they do not affect performance and only take up space.
</details>

## Installation

### 1. Download

```bash
mkdir -p ~/gamescope-opt && cd ~/gamescope-opt
gh release download --repo argonforge/gamescope-builds --pattern '*.zst'
tar --zstd -xf <archive name>.tar.zst
```

Or download the `.zst` file manually from the [Releases](../../releases) page.

### 2. Back up current package

```bash
sudo mkdir -p /root/gamescope-backup
sudo cp /var/cache/apt/archives/gamescope_*.deb /root/gamescope-backup/ 2>/dev/null || true
```

### 3. Install

```bash
cd ~/gamescope-opt
sudo apt install ./packages/*.deb
```

The `./` prefix is required - otherwise `apt` will look for packages in repositories.

### 4. Hold version

To prevent `apt upgrade` from reverting to the stock package. See [Hold & Rollback](#hold--rollback).

### 5. Verify

```bash
gamescope --version
gamescope -- glxgears
```

## Hold & Rollback

```bash
# Hold
sudo apt-mark hold gamescope

# If the compositor becomes unstable, freezes, or crashes:
# Unhold
sudo apt-mark unhold gamescope

# Reinstall stock version
sudo apt install --reinstall gamescope

sudo reboot
```

## Expected Performance

| Component                          | Gain   | Comment                                              |
|------------------------------------|--------|------------------------------------------------------|
| Micro-compositor overhead          | 1-3%   | Noticeable only in CPU-bound scenarios                |
| Nested compositing overhead        | ~1%    | Compared to running the game without Gamescope        |
| Overall FPS in GPU-bound games     | ~0%    | Bottleneck is GPU and memory bandwidth, not compositor |

**Honest note:** Gamescope is a compositor, not a driver or translation layer. Its own CPU cost is small, and optimizing it yields a few percent in CPU-bound scenarios at best. The main FPS gains in games come from the graphics stack (Mesa, DXVK, VKD3D-Proton), not from Gamescope itself.

Source workflow: [`.github/workflows/build-devuan-stable.yml`](.github/workflows/build-devuan-stable.yml).

## Companion projects

For a complete optimized graphics stack on **AMD Zen (x86-64-v3/v4)**:

| Project                                                                    | Purpose                           |
|----------------------------------------------------------------------------|-----------------------------------|
| [`mesa-builds`](https://github.com/argonforge/mesa-builds)                 | Mesa (radeonsi, RADV)             |
| [`dxvk-builds`](https://github.com/argonforge/dxvk-builds)                 | DXVK (D3D9/10/11 -> Vulkan)       |
| [`vkd3d-proton-builds`](https://github.com/argonforge/vkd3d-proton-builds) | VKD3D-Proton (D3D12 -> Vulkan)    |
| [`wine-builds`](https://github.com/argonforge/wine-builds)                 | Wine WoW64 (Clang)                |

## Important

- Packages are built **only for Devuan Excalibur**. Installing on Daedalus (oldstable) or other releases may break due to library version mismatches (`libwlroots-0.18`, `libdrm 2.4.124`, `pixman 0.44`, `wayland-server 1.23.1`).
- Packages are **not signed**. Verify integrity using SHA-256 from the release description.
- **Do not install v4 packages** if you are unsure about AVX-512 support. Use v3 if your CPU has only AVX2.
- Runtime performance of Gamescope is limited by the micro-compositor's CPU-side overhead. The main FPS gains come from the graphics stack.
- The author is not responsible for any system issues. Always have a Live USB ready for recovery.

## License

The build scripts and GitHub Actions workflows in this repository
are licensed under the MIT License. See LICENSE file.

The Gamescope source code and Debian packaging files are distributed
under their respective licenses - see the gamescope source package
for details. The compiled .deb packages in Releases are
redistributions of Gamescope under its original license.
