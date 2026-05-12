# Installing Rust (`rustup`) on Windows When the Network Is Restricted

This guide is written for situations where **normal access to Rust’s official infrastructure is unreliable or blocked** (for example, during severe filtering or international connectivity issues). The flow below uses a **public mirror** for `rustup` metadata and toolchain downloads, and documents a **manual rescue step** when even the mirror hostname fails to resolve until you seed one small file.

The examples use **PowerShell** on **64-bit Windows**, starting from the **MSVC** installer triple (`x86_64-pc-windows-msvc`). **Step 6** shows how to set **GNU** (`x86_64-pc-windows-gnu`) as the default toolchain afterward. You can adapt URLs and triples if you use a different architecture.

---

## What you are doing (in one paragraph)

You download the official **rustup installer** from a mirror, point `rustup` at that mirror with two environment variables, run the installer, and—if DNS to the mirror fails—**manually place** the missing `channel-rust-stable.toml.sha256` file into the **exact temporary path** `rustup` printed in its error message. After that, you run the installer again and let it finish downloading the toolchain.

---

## Prerequisites

- **Windows** with permission to install under your user profile (default locations: `%USERPROFILE%\.rustup` and `%USERPROFILE%\.cargo`).
- A way to obtain files **out of band** when needed (another network, VPN, mobile hotspot, friend’s machine, USB stick, etc.). If a hostname never resolves, **something** must still be able to reach the mirror long enough to download the small metadata files, or you must copy them from another machine.
- **PowerShell** (used in the commands below).

---

## Step 1 — Download `rustup-init.exe` from a mirror

Download the Windows MSVC installer from the Tsinghua mirror (replace the URL only if you use another mirror you trust):

```text
https://mirrors.tuna.tsinghua.edu.cn/rustup/rustup/dist/x86_64-pc-windows-msvc/rustup-init.exe
```

Save it somewhere convenient (for example `F:\rustup-init.exe`).

---

## Step 2 — Point `rustup` at the mirror (current session)

In **PowerShell**, set these variables **in the same session** before running the installer. They tell `rustup` where to fetch channel metadata and toolchain packages:

```powershell
$env:RUSTUP_DIST_SERVER = "https://mirrors.tuna.tsinghua.edu.cn/rustup"
$env:RUSTUP_UPDATE_ROOT = "https://mirrors.tuna.tsinghua.edu.cn/rustup/rustup"
```

Optional but recommended for repeat installs: add the same values as **user** environment variables in Windows (System Properties → Environment Variables) so you do not have to set them every time you open a new terminal.

---

## Step 3 — Run the installer

From the directory where you saved the file:

```powershell
.\rustup-init.exe
```

### Visual C++ prerequisites prompt

You may see:

> Rust requires a linker and Windows API libraries…

The installer offers:

1. Quick install via Visual Studio Community
2. Manual prerequisites
3. **Don’t install the prerequisites** (often chosen if you will use the **GNU** ABI / another linker story, or you will install MSVC build tools yourself later)

In the walkthrough this document is based on, **option `3`** was chosen. If you need the **MSVC** toolchain in a serious way, plan to install **Visual Studio Build Tools** or **Visual Studio** with the C++ workload when you have connectivity.

### Installation mode

When asked:

```text
1) Proceed with standard installation (default - just press enter)
2) Customize installation
3) Cancel installation
```

Choose **`1`** for a normal stable toolchain unless you know you need custom components.

---

## Step 4 — If you hit a DNS / “No such host” error

You might see something like:

```text
error: could not download file from 'https://mirrors.tuna.tsinghua.edu.cn/rustup/dist/channel-rust-stable.toml.sha256' to 'C:\Users\USER\.rustup\tmp\u0l7a9ubhzddx122_file': error downloading file: ... dns error: No such host is known. (os error 11001)
```

That means **`rustup` could not download** that small metadata file. Two important details:

1. **Download URL** — Take the URL from the error line, for example:  
   `https://mirrors.tuna.tsinghua.edu.cn/rustup/dist/channel-rust-stable.toml.sha256`
2. **Destination path** — The path after `to '` is **random per run** (the `tmp\..._file` name changes). You must use **the path from your own error message**, not a path copied from a blog post.

### What to do

1. On **any machine or browser session** that **can** reach the mirror, download **`channel-rust-stable.toml.sha256`** from that exact URL.
2. Copy the file onto the Windows machine.
3. Rename or save it so it matches the **full filename** `rustup` expected in the error (including the random prefix and the `_file` suffix), e.g.  
   `C:\Users\YOURNAME\.rustup\tmp\u0l7a9ubhzddx122_file`  
   Create the `tmp` folder under `%USERPROFILE%\.rustup` if it does not exist.
4. Run the installer again **with the same mirror environment variables** (Step 2):

   ```powershell
   $env:RUSTUP_DIST_SERVER = "https://mirrors.tuna.tsinghua.edu.cn/rustup"
   $env:RUSTUP_UPDATE_ROOT = "https://mirrors.tuna.tsinghua.edu.cn/rustup/rustup"
   .\rustup-init.exe
   ```

If `rustup` warns about an existing **`settings.toml`**, that is normal after a partial run; continue and let it complete the toolchain download.

---

## Step 5 — Confirm success

Close and reopen PowerShell (or open a new window) so **`PATH`** picks up `%USERPROFILE%\.cargo\bin`, then:

```powershell
cargo --version
```

Example of a successful install:

```text
cargo 1.95.0 (f2d3ce0bd 2026-03-21)
```

You can also verify:

```powershell
rustc --version
rustup --version
```

---

## Step 6 — Optional: switch the default toolchain to GNU (`x86_64-pc-windows-gnu`)

If you first installed the **MSVC** host triple (for example via `rustup-init.exe` for `x86_64-pc-windows-msvc`) but want the **GNU** toolchain as default—so you can link with **MinGW** instead of the Visual C++ build tools—set the **same mirror variables** in PowerShell, then install or refresh toolchains and set GNU as default.

1. **Same session as your `rustup` commands** (or set these permanently in user environment variables):

   ```powershell
   $env:RUSTUP_DIST_SERVER = "https://mirrors.tuna.tsinghua.edu.cn/rustup"
   $env:RUSTUP_UPDATE_ROOT = "https://mirrors.tuna.tsinghua.edu.cn/rustup/rustup"
   ```

2. **Ensure the default stable toolchain is present** (if you already have MSVC stable, this step may report that the existing install is in use):

   ```powershell
   rustup toolchain install
   ```

3. **Install GNU stable (if needed) and make it the default**:

   ```powershell
   rustup default stable-x86_64-pc-windows-gnu
   ```

   `rustup` will sync the channel from the mirror, download components (including **`rust-mingw`** for the GNU target), and set the default toolchain.

Example session (paths and versions match a real run; yours may differ slightly):

```text
PS C:\Windows\system32> $env:RUSTUP_DIST_SERVER = "https://mirrors.tuna.tsinghua.edu.cn/rustup"
>> $env:RUSTUP_UPDATE_ROOT = "https://mirrors.tuna.tsinghua.edu.cn/rustup/rustup"
PS C:\Windows\system32>
PS C:\Windows\system32> rustup toolchain install
info: using existing install for stable-x86_64-pc-windows-msvc
info: the active toolchain `stable-x86_64-pc-windows-msvc` has been installed
info: it's active because: it's the default toolchain
info: checking for self-update (current version: 1.29.0)
PS C:\Windows\system32> rustup default stable-x86_64-pc-windows-gnu
info: syncing channel updates for stable-x86_64-pc-windows-gnu
info: latest update on 2026-04-16 for version 1.95.0 (59807616e 2026-04-14)
info: downloading 7 components
        cargo installed                       11.20 MiB
       clippy installed                        4.89 MiB
    rust-docs installed                       21.20 MiB
   rust-mingw installed                        5.22 MiB
     rust-std installed                       25.11 MiB
        rustc installed                       95.74 MiB
      rustfmt installed                        2.78 MiB                                                                 info: default toolchain set to stable-x86_64-pc-windows-gnu

  stable-x86_64-pc-windows-gnu installed - rustc 1.95.0 (59807616e 2026-04-14)
```

After switching, confirm:

```powershell
rustup default
rustc --version
```

You should see **`stable-x86_64-pc-windows-gnu`** as default and a `host: x86_64-pc-windows-gnu` line from `rustc -vV`.

---

## Reference — default directories

| Purpose                                  | Default location           | Override      |
| ---------------------------------------- | -------------------------- | ------------- |
| Rustup metadata & toolchains             | `%USERPROFILE%\.rustup`    | `RUSTUP_HOME` |
| Cargo config & binaries                  | `%USERPROFILE%\.cargo`     | `CARGO_HOME`  |
| Executables (`cargo`, `rustc`, `rustup`) | `%USERPROFILE%\.cargo\bin` | (via `PATH`)  |

Uninstall: `rustup self uninstall` (reverts `PATH` changes made by rustup for the user).

---

## Why this works

- **`RUSTUP_DIST_SERVER`** — Base URL for **dist** files (channel files, toolchain manifests, component downloads for the default distribution layout).
- **`RUSTUP_UPDATE_ROOT`** — Base URL for **rustup’s own update metadata** (the `rustup` self-update channel layout).

Using a **mirror** reduces dependence on `static.rust-lang.org` when it is slow or unreachable. The **manual file copy** step works around **intermittent DNS or transport failures** for a specific small file `rustup` needs to trust the channel before downloading larger artifacts.

---

## Limitations and safety notes

- **Mirror trust** — You are trusting the mirror operator not to tamper with artifacts. Prefer **well-known university or distro mirrors** and, when possible, verify checksums or signatures using documentation from the same mirror or the Rust project.
- **If the mirror is unreachable too** — You will need another mirror, VPN, Tor, or an offline transfer strategy; this README only documents the **rustup + mirror + manual tmp seed** pattern.
- **Corporate proxies** — If you use an HTTP proxy, you may also need `HTTP_PROXY` / `HTTPS_PROXY`; that is outside the scope of this short guide.
- done
