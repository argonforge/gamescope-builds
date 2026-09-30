# Gamescope for Devuan Excalibur (Debian Trixie base)

Optimized [Gamescope](https://github.com/ValveSoftware/gamescope) (micro-compositor) packages for Devuan Excalibur, built for modern CPUs.

## Requirements

- **Distribution:** Devuan Excalibur (stable) or Debian Trixie (stable)
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

| Package     | Purpose                                                   |
|-------------|-----------------------------------------------------------|
| `gamescope` | Micro-compositor providing scaling and fullscreen options |

Each release is published as two variants:

| Codename        | CPU requirement     | Version suffix |
|-----------------|---------------------|----------------|
| `stable-avx2`   | x86-64-v3 (AVX2)    | `+avx2`        |
| `stable-avx512` | x86-64-v4 (AVX-512) | `+avx512`      |

Debug packages (`*-dbgsym`) are **not included** - they do not affect performance and only take up space.
</details>

## APT repository

The repository is hosted on GitHub Pages:

```
https://argonforge.github.io/gamescope-builds
```

### 1. Import the signing key

```bash
curl -fsSL https://argonforge.github.io/gamescope-builds/public.asc \
  | sudo gpg --dearmor -o /usr/share/keyrings/argonforge-gamescope.gpg
```

### 2. Add the repository

Pick the codename matching your CPU. **Only one of the two, not both.**

For x86-64-v3 (AVX2 - most x86-64 CPUs from 2013+):

```bash
echo "deb [signed-by=/usr/share/keyrings/argonforge-gamescope.gpg] https://argonforge.github.io/gamescope-builds stable-avx2 main" \
  | sudo tee /etc/apt/sources.list.d/argonforge-gamescope.list
```

For x86-64-v4 (AVX-512 - Zen 4/5):

```bash
echo "deb [signed-by=/usr/share/keyrings/argonforge-gamescope.gpg] https://argonforge.github.io/gamescope-builds stable-avx512 main" \
  | sudo tee /etc/apt/sources.list.d/argonforge-gamescope.list
```

### 3. Install

```bash
sudo apt update
sudo apt install gamescope
```

`apt` may mark the package as `DOWNGRADING` if a newer version was installed from another source. This is expected - the Argon Forge build replaces the stock package.

### 4. Hold version (recommended)

To prevent `apt upgrade` from reverting to the stock package:

```bash
sudo apt-mark hold gamescope
```

### 5. Verify

```bash
apt policy gamescope
gamescope --version
```

The version string should end with `+avx2` or `+avx512` depending on the codename you selected.

## Manual installation (alternative)

If you prefer not to add the repository:

### 1. Download

```bash
mkdir -p ~/gamescope-opt && cd ~/gamescope-opt
gh release download --repo argonforge/gamescope-builds --pattern '*.zst'

# Unpack the archive matching your CPU:
tar --zstd -xf gamescope-*-x86-64-v4.tar.zst   # AVX-512
# or
tar --zstd -xf gamescope-*-x86-64-v3.tar.zst   # AVX2
```

Or download the `.tar.zst` file manually from the [Releases](../../releases) page.

### 2. Install

```bash
sudo apt install ./packages/*.deb
```

The `./` prefix is required - otherwise `apt` will look for packages in repositories.

## Rollback

If the compositor becomes unstable, freezes, or crashes:

```bash
# Unhold
sudo apt-mark unhold gamescope

# Remove the Argon Forge repository
sudo rm /etc/apt/sources.list.d/argonforge-gamescope.list
sudo apt update

# Reinstall stock version
sudo apt install --reinstall gamescope

sudo reboot
```

If you installed manually:

```bash
sudo apt remove gamescope
sudo apt install gamescope   # from your distribution
```

## Expected Performance

| Component                      | Gain | Comment                                                |
|--------------------------------|------|--------------------------------------------------------|
| Micro-compositor overhead      | 1-3% | Noticeable only in CPU-bound scenarios                 |
| Nested compositing overhead    | ~1%  | Compared to running the game without Gamescope         |
| Overall FPS in GPU-bound games | ~0%  | Bottleneck is GPU and memory bandwidth, not compositor |

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

- Packages are built for **Debian Trixie / Devuan Excalibur** (same library base). Installing on Bookworm, Daedalus or other releases will break due to library version mismatches (`libwlroots-0.18`, `libdrm 2.4.124`, `pixman 0.44`, `wayland-server 1.23.1`).
- Packages are **signed with the Argon Forge key**. Verify the fingerprint against the one published at `https://argonforge.github.io/gamescope-builds/public.asc` before trusting the repository.
- **Do not add both codenames** (`stable-avx2` and `stable-avx512`) at the same time. Choose the one matching your CPU.
- **Do not install v4 packages** if you are unsure about AVX-512 support. Use `stable-avx2` if your CPU has only AVX2.
- Runtime performance of Gamescope is limited by the micro-compositor's CPU-side overhead. The main FPS gains come from the graphics stack.
- The author is not responsible for any system issues. Always have a Live USB ready for recovery.

## License

The build scripts and GitHub Actions workflows in this repository
are licensed under the MIT License. See LICENSE file.

The Gamescope source code and Debian packaging files are distributed
under their respective licenses - see the gamescope source package
for details. The compiled `.deb` packages in Releases and in the
APT repository are redistributions of Gamescope under its original license.
