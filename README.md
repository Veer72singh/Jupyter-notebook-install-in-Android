# Jupyter Notebook on Android

A complete guide to installing and running Jupyter Notebook on Android with [Termux](https://termux.dev/). This procedure is based on the installation and troubleshooting process documented in [`JupyterGuide.pdf`](JupyterGuide.pdf).

## What You Will Get

- A local Python 3 and Jupyter Notebook environment on an Android phone or tablet
- Access to notebooks through a mobile browser
- Preventive steps for common native-package and Rust build errors
- Practical fixes for storage, networking, background-process, browser, and disk-space problems

## Requirements

- Android 7.0 (Nougat) or newer
- Android 11 or newer recommended
- At least 2-3 GB of free internal storage
- At least 3 GB RAM recommended
- Internet access for the initial installation
- Chrome, Firefox, or another modern mobile browser
- No root access is required

## Installation

### 1. Install Termux

Install Termux from the official [Termux GitHub releases](https://github.com/termux/termux-app/releases). Download the universal APK unless you know your device's exact CPU architecture.

> Do not use the deprecated Google Play Store build. If Android asks for permission to install unknown apps, allow it for the browser or file manager used to install the APK.

If you need Termux:API commands, install the Termux:API add-on from the same source as Termux. Do not mix Termux and add-ons from different sources because their package signatures may not match.

### 2. Update Termux

Open Termux and update the package index and installed packages:

```bash
pkg update -y
pkg upgrade -y
```

### 3. Grant Storage Access

Allow Termux to access shared Android storage:

```bash
termux-setup-storage
```

After permission is granted, Termux creates `~/storage`, with links to locations such as Downloads, DCIM, and Music.

### 4. Install Python

```bash
pkg install python -y
python --version
pip --version
```

### 5. Install Build Dependencies

Install the compiler and native libraries required by Jupyter dependencies:

```bash
pkg install -y build-essential clang libzmq rust cmake pkg-config
```

### 6. Upgrade Python Packaging Tools

```bash
pip install --upgrade pip setuptools wheel
```

### 7. Set the Android API Level

Set the API level before installing Jupyter or other packages with Rust components:

```bash
export ANDROID_API_LEVEL=21
export LDFLAGS="-lm -lcompiler_rt"
```

Make these settings permanent for future Termux sessions:

```bash
echo 'export ANDROID_API_LEVEL=21' >> ~/.bashrc
echo 'export LDFLAGS="-lm -lcompiler_rt"' >> ~/.bashrc
source ~/.bashrc
```

### 8. Install Precompiled Scientific Packages

Install Termux's precompiled packages before using pip. This prevents pip from attempting slow or unsupported source builds:

```bash
pkg install -y python-psutil python-numpy python-scipy python-matplotlib
```

### 9. Clear the pip Cache

```bash
pip cache purge
```

### 10. Install Jupyter Notebook

```bash
pip install jupyter
```

The classic Notebook interface is recommended for the smoothest Termux installation. JupyterLab is heavier and may require additional native or Rust build dependencies.

### 11. Verify the Installation

```bash
jupyter --version
```

The command should display versions for Jupyter Core, Notebook, ipykernel, and related packages.

### 12. Start Jupyter Notebook

```bash
jupyter notebook --no-browser --ip=127.0.0.1 --port=8888
```

The options mean:

- `--no-browser`: prevents Termux from trying to open a browser automatically.
- `--ip=127.0.0.1`: keeps the server accessible only on the same device.
- `--port=8888`: selects the local server port.

### 13. Open Jupyter in the Browser

Termux prints a URL similar to:

```text
http://127.0.0.1:8888/?token=YOUR_TOKEN
```

Copy the complete URL, including the token, and open it in Chrome, Firefox, or another browser on the same Android device. The Jupyter dashboard should appear, allowing you to create a Python 3 notebook.

> Treat the token URL like a password. Do not share it or include it in screenshots.

## Optional Setup

### Install Pandas

Prefer Termux's precompiled packages for large scientific libraries:

```bash
pkg install -y python-pandas
```

Use `pip install <package>` only when the package is not available from the Termux repositories.

### Keep Jupyter Running in the Background

Android may stop background processes to save battery. Use `tmux` and a wake lock for longer sessions:

```bash
pkg install tmux -y
tmux new -s jupyter
jupyter notebook --no-browser --ip=127.0.0.1 --port=8888
```

Detach from the session with `Ctrl+b`, then `d`. Keep the CPU awake with:

```bash
termux-wake-lock
```

Reattach later with:

```bash
tmux attach -t jupyter
```

### Create a Quick-Launch Alias

```bash
echo 'alias jn="jupyter notebook --no-browser --ip=127.0.0.1 --port=8888"' >> ~/.bashrc
source ~/.bashrc
```

You can then start Jupyter with:

```bash
jn
```

## Preventive Checklist

Before troubleshooting, confirm that:

- `pkg update -y` and `pkg upgrade -y` have been run.
- `build-essential`, `clang`, `libzmq`, and `rust` are installed.
- `echo $ANDROID_API_LEVEL` prints `21`.
- `python-psutil` and `python-numpy` were installed with `pkg`.
- `pip cache purge` has been run.
- `jupyter --version` works.

## Troubleshooting

### `Failed building wheel for psutil`

Install the precompiled Termux package, clear the pip cache, and retry:

```bash
pkg install python-psutil -y
pip cache purge
pip install jupyter
```

### `ImportError` When Starting Jupyter (`_zmq`)

Install `patchelf` and link the ZeroMQ extension against Termux's Python library:

```bash
pkg install patchelf -y
python --version
patchelf --add-needed libpython3.12.so \
$PREFIX/lib/python3.12/site-packages/zmq/backend/cython/_zmq.cpython-312-*.so
```

If your Python version is different, use the matching `libpython` and extension path shown by your installation.

### `maturin failed: Android API level not found`

```bash
export ANDROID_API_LEVEL=21
export LDFLAGS="-lm -lcompiler_rt"
pip cache purge
pip install jupyter
```

### `termux-setup-storage` Does Nothing or Shows Permission Denied

Run the command again:

```bash
termux-setup-storage
```

If no permission prompt appears, enable **Files and media** permission manually in **Android Settings > Apps > Termux > Permissions**.

### `jupyter: command not found`

Add pip's user-installed scripts directory to `PATH`:

```bash
echo 'export PATH=$PATH:$HOME/.local/bin' >> ~/.bashrc
source ~/.bashrc
jupyter --version
```

### `OSError: [Errno 99] Cannot assign requested address`

Use the loopback address:

```bash
jupyter notebook --ip=127.0.0.1 --port=8888 --no-browser
```

For optional access from another device on the same Wi-Fi network, inspect the phone's local address and bind to all interfaces:

```bash
pkg install net-tools -y
ifconfig
jupyter notebook --ip=0.0.0.0 --port=8888 --no-browser
```

Only expose Jupyter to a network when necessary.

### Jupyter Stops When the Phone Is Locked

```bash
pkg install tmux -y
tmux new -s jupyter
jn
termux-wake-lock
```

Also set **Android Settings > Battery > Termux > Unrestricted / No restrictions**.

### Blank or Black Browser Screen

The server may still be working while the browser loads its JavaScript assets. Try:

1. Hard refresh the page.
2. Clear cached images and files in the browser settings.
3. Wait for the initial assets to load.
4. Try Chrome, Firefox, or another browser.
5. Try a different port:

```bash
jupyter notebook --no-browser --ip=127.0.0.1 --port=9999
```

### SSL Certificate Errors from pip or curl

```bash
pkg install ca-certificates -y
update-ca-certificates
```

### `setlocale` Warning

This warning is cosmetic and does not prevent Jupyter from working. To configure the locale:

```bash
echo 'export LANG=en_US.UTF-8' >> ~/.bashrc
echo 'export LC_ALL=en_US.UTF-8' >> ~/.bashrc
source ~/.bashrc
```

### `No Space Left on Device`

Clean Termux and pip caches:

```bash
pkg clean
apt autoremove -y
pip cache purge
```

## One-Shot Setup Script

Run these commands line by line the first time so that any failure is visible:

```bash
pkg update -y && pkg upgrade -y
termux-setup-storage
pkg install -y build-essential clang libzmq rust cmake pkg-config
export ANDROID_API_LEVEL=21
echo 'export ANDROID_API_LEVEL=21' >> ~/.bashrc
pkg install -y python-psutil python-numpy python-scipy python-matplotlib
pip install --upgrade pip setuptools wheel
pip cache purge
pip install jupyter
jupyter --version
echo 'alias jn="jupyter notebook --no-browser --ip=127.0.0.1"' >> ~/.bashrc
source ~/.bashrc
```

Start the server after setup with:

```bash
jn
```

## Best Practices and Security

- Set `ANDROID_API_LEVEL` before running package installation commands.
- Install precompiled Termux packages before using pip for heavy scientific dependencies.
- Update Termux before installing large packages.
- Use `tmux` for Jupyter sessions and other long-running processes.
- Back up notebooks and projects to GitHub or `~/storage/shared`; uninstalling Termux removes its private app storage.
- Termux does not require `sudo` for normal package management.
- Keep `--ip=127.0.0.1` unless LAN access is specifically required.
- Never share Jupyter token URLs.
- Monitor battery temperature and available storage during sustained workloads.

## Useful Commands

| Command | Purpose |
| --- | --- |
| `pkg search <term>` | Search available Termux packages |
| `pkg list-installed` | List installed packages |
| `pkg clean` | Clear downloaded package archives |
| `ls -la` | List files, including hidden files |
| `cd <dir>` | Change directory |
| `pwd` | Print the current directory |
| `mkdir -p <dir>` | Create a directory and its parents |
| `df -h` | Show available disk space |
| `du -sh <dir>` | Show directory size |
| `python --version` | Show the Python version |
| `pip list` | List installed Python packages |
| `pip show <package>` | Show package details |
| `pip cache purge` | Clear pip's download cache |
| `python -m venv myenv` | Create a Python virtual environment |
| `source myenv/bin/activate` | Activate a virtual environment |
| `deactivate` | Exit the active virtual environment |
| `termux-open-url <url>` | Open a URL in Android's default browser |
| `termux-info` | Print Termux diagnostic information |
| `termux-wake-unlock` | Release a Termux wake lock |

## Reference

The complete source procedure is available in [`JupyterGuide.pdf`](JupyterGuide.pdf).
