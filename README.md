# DIAC NET CHAIN (DNC)

A distributed economic protocol with peer discovery, wallet, and mining — designed for configurable, serverless networks. This repository contains a Tkinter desktop client and the core components required to run, build, and test DNC on Windows.

---

## Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Architecture](#architecture)
- [Requirements](#requirements)
- [Quick Start (Run from Source)](#quick-start-run-from-source)
- [Quick Start (Prebuilt .zip for Windows)](#quick-start-prebuilt-zip-for-windows)
- [Build Your Own Windows Executable](#build-your-own-windows-executable)
- [Usage Guide](#usage-guide)
  - [First Launch](#first-launch)
  - [Accounts and Keys](#accounts-and-keys)
  - [Wallet](#wallet)
  - [Mining](#mining)
  - [Network](#network)
- [Creating Your Own DNC Subnetwork](#creating-your-own-dnc-subnetwork)
- [How Peer Discovery Works](#how-peer-discovery-works)
- [Security Notes](#security-notes)
- [Troubleshooting](#troubleshooting)
- [Contributing and Forks](#contributing-and-forks)
- [License](#license)
- [Citation and Original Paper](#citation-and-original-paper)
- [Acknowledgments](#acknowledgments)

---

## Overview

DNC (DIAC NET CHAIN) is a fully distributed, programmable network for creating and operating digital economies without central servers, wasteful proof-of-work, or monolithic global blockchains. Every participant can hold funds, mine, validate, and synchronize value. Networks can be public, private, or isolated subnets, all discovered automatically on local networks and optionally bootstrapped via a public discovery URL.

The desktop client is implemented in Python with a clean Tkinter GUI and a simple, auditable core.

---

## Key Features

- Real P2P orientation with LAN multicast discovery and TCP health pings.
- Wallet with account creation, login via existing keys, and transaction history.
- Lightweight mining with local rewards and visible efficiency metrics.
- Configurable networks: join the central DIAC port or create your own subnet on a custom port.
- Optional Internet-wide peer discovery via a single environment variable.
- No third-party dependencies beyond the Python standard library and Tkinter.
- Open for forks, extensions, and research.
- If everyone disconnects and deletes their information, all accounts will be deleted. It could also be used as a cleaner for inactive accounts or bots.

---

## Architecture

- `diac_client.py` — launcher for the Tkinter desktop client.
- `diac_dnc_gui.py` — GUI with tabs: Home/Account, Wallet, Mining, Network.
- `diac_dnc_adapter.py` — adapter bridging GUI and core, providing:
  - TCP ping server on a chosen port (default 369) and LAN multicast discovery.
  - Real peer liveness checks (ping => pong).
  - Mining thread control and stats relay.
- `diac_dnc_client_core.py` — client core for keys, balances, transactions, local mining, peer list, and discovery URL consumption.

Default central network port: **369**.  
LAN multicast group: **239.255.124.69:39369** (used for autodiscovery among nodes on the same LAN and port).

---

## Requirements

- Windows with Python installed (tested with Python 3.13; other recent versions should work).
- Tkinter is included in the standard Python for Windows installers.
- For building an executable: PyInstaller.

---

## Quick Start (Run from Source)

1.  Download or clone this repository into a folder of your choice.
2.  Open **Command Prompt** and change to the project directory:
    ```bat
    cd C:\path\to\your\DNC\project
    ```
3.  Run the client:
    ```bat
    python diac_client.py
    ```
    If you have multiple Python versions, call the full path to your Python executable, for example:
    ```bat
    "C:\Users\<YourUser>\AppData\Local\Programs\Python\Python313\python.exe" diac_client.py
    ```

---

## Quick Start (Prebuilt .zip for Windows)

If you downloaded `DNC Program V1.zip`:

1.  Ensure Python for Windows is installed (for future extensibility; the prebuilt executable itself does not require Python to run).
2.  Extract `DNC program.zip` to a folder.
3.  Open the extracted folder, then open the `dist` folder.
4.  Double-click `DIAC NET Client` to start the application.

This build is intended for Windows only at the moment.

---

## Build Your Own Windows Executable

Below is a reproducible set of steps using PyInstaller on Windows. Replace `<YourUser>` with your Windows username.

1.  Install PyInstaller:
    ```bat
    "C:\Users\<YourUser>\AppData\Local\Programs\Python\Python313\python.exe" -m pip install pyinstaller
    ```
2.  From the project directory (cd into the folder containing the source files), build a single-file, windowed executable with a custom icon and name:
   ```bat
cd C:\Users\<YourUser>... (path to your folder where you have the .zip descompressed)
   ```
   ```bat
"C:\Users\<YourUser>\AppData\Local\Programs\Python\Python313\python.exe" ^
 -m PyInstaller --onefile --windowed --clean ^
 --icon="C:\Users\<YourName>\Documents\DIAC INFINITY 2 DNC\dnc2\diac_icon.ico" ^
 --name "DIAC NET Client" ^
 --paths . ^
 --collect-all diac_dnc ^
 --debug all ^
 diac_client.py
    ```

3.  After completion, the executable will appear under the `dist` folder:
    `dist\DIAC NET Client.exe`

Distribute by zipping the `dist` output (and any required assets like `diac_icon.ico` if you use it externally).

**Notes:**
- The above commands assume your Python is installed at the 3.13 default path. Adjust if needed.
- If `diac_icon.ico` is not present, omit the `--icon` flag or provide a valid icon path.

---

## Usage Guide

### First Launch

On first run, a dialog will ask you to select a network mode:
- **Central DIAC (port 369):** default global port. Nodes on the same LAN and port will autodiscover one another.
- **Custom subnet (custom port):** creates an isolated DNC network on the port you choose. This is useful for private experiments or closed environments.

You can change the network later from the Network tab.

### Accounts and Keys

- Create a new account or log in with existing keys.
- Keys are generated locally and never leave your machine.
- The application stores configuration under your user profile, e.g., `%USERPROFILE%\.diac_dnc\`.
- You can view your address, public key, and private key from the Home/Account tab. **Keep your private key secret.**

### Wallet

- The Wallet tab shows your DIAC balance and transaction history.
- Send DIAC to any address of the form `DNC-...`.
- Confirm the recipient and amount, then submit the transaction. Recent transactions appear in the history panel.

### Mining

- Start or stop mining from the Mining tab.
- Mining is lightweight and local. Successful cycles yield small DIAC rewards credited to your balance.
- The UI displays total rewards, estimated hourly rate, completed challenges, and efficiency.
- Mining runs in the background while the client is open.

### Network

- The Network tab displays current stats, known peers, and discovery controls.
- Add peers by IP and port, optionally with an address label.
- Use the peer discovery button if a discovery URL is configured (see below).
- LAN multicast discovery automatically finds peers on the same LAN that share your selected port.

---

## Creating Your Own DNC Subnetwork

1.  Choose **Custom subnet** when prompted on first run, or change it later from the Network tab.
2.  Pick an unused TCP port in the range 1–65535 (e.g., 5369).
3.  Ensure that systems in your subnet use the same port.
4.  For cross-network or Internet connectivity, consider firewall rules and port forwarding on your router.
5.  Share peer `IP:port` information with participants so they can add each other as peers.
6.  LAN participants will be auto-discovered via multicast when using the same port.

---

## How Peer Discovery Works

- **LAN Discovery:** All nodes using the same port on a local network will find each other automatically via LAN multicast.
- **Manual Peer Add:** To connect globally, simply share your IP address and port with others and add peers manually in the Network tab.
- **Peer-to-Peer Exchange:** After connecting to at least one peer, click “Discover Peers (P2P)” in the Network tab to automatically request and merge peer lists from all your known peers. This expands your connectivity organically without any central server or discovery URL.


## Security Notes

- **Private keys** are stored locally under your user profile.  
  Back up your keys securely and **never share them**.
- The client exposes a **TCP ping server** on the selected port for liveness checks.  
  Choose ports responsibly and apply firewall rules where appropriate.
- This software is provided **without warranty**.  
  Review the license terms before production use.

## Key Storage Location

Your DIAC wallet’s **public and private keys** are stored **locally on your computer**.  
They are **never uploaded or sent over the network**.

**Default location on Windows:**
```plaintext
C:\Users\YOUR_USERNAME\.diac_dnc\gui_config.json
```

**Default location on Linux/Mac:**
```plaintext
/home/YOUR_USERNAME/.diac_dnc/gui_config.json
```

- This folder and file are created automatically on first launch.
- To **move your wallet to another device**, just copy this file to the same location.
- **WARNING:** If you delete or lose this file, you lose access to your wallet and all your DIAC.
- Make regular backups (preferably encrypted or on a secure USB drive).

---

## Troubleshooting

- If the **GUI does not start**, verify that **Python** and **Tkinter** are properly installed.
- If **mining does not start**:
  - Ensure your account is created or logged in, and try again.
- Check logs in the console for error messages printed by the adapter or core.

## Changelog / Recent Changes

- **2025-07**
  - Fixed a critical bug with `Reputation` initialization. Now supports both signature variants (`base_path` as a `Path` object, or no argument).
  - All reputation/config data is now stored using `Path` objects for full Windows and cross-platform compatibility.
  - The README has been updated with notes on cache folders and correct troubleshooting tips.
  - Build and troubleshooting instructions verified on Windows 11 with Python 3.13.
  - Improved robustness: the app now auto-creates the `~/.diac_dnc/` folder if missing.
  - More informative error messages if required modules or files are missing at launch.

## Troubleshooting (Updated)

- If you see `TypeError: unsupported operand type(s) for /: 'str' and 'str'`, make sure you are using the latest `diac_dnc_adapter.py`, which passes `Path` (not `str`) to `Reputation`.
- You can safely delete `__pycache__` folders if you encounter weird import errors, then recompile.
- Make sure your Python installation includes Tkinter (default for most Windows installers).


## New Features and Improvements (2025-07)

### Enhanced Peer-to-Peer & Network Functionality
- **Live Peer Health Checking:**  
  The app now includes direct TCP ping checks to verify peer liveness, making network status more reliable and visible in the Network tab.
- **Real Peer Discovery Exchange:**  
  Peer discovery now includes a real peer list exchange and automatic merging, so adding a single peer can rapidly expand your network without any centralized server.
- **Manual Peer Addition:**  
  Users can add new peers by IP and port directly from the Network tab, even labeling addresses for easier management.

### Robust Reputation and Security System
- **Reputation Engine Integration:**  
  Each peer is now tracked locally with a score, visible and adjustable from the Security tab, enabling future slashing and rewards.
- **Robust Cross-Platform Data Storage:**  
  Reputation and configuration data is always stored in your user profile directory (not the current folder), for safety and portability.
- **Safe Path Handling:**  
  All file and directory paths now use Python's `Path` objects to avoid Windows/Linux bugs and ensure full compatibility.

### Wallet and Key Management
- **Account Import and Export:**  
  Users can log in using existing public/private keys, making account migration and backup seamless.
- **Key Visibility:**  
  The client allows you to securely view (and copy) your address, public key, and private key from the Home/Account tab.

### Mining Engine and Metrics
- **Start/Stop Mining from UI:**  
  Mining can now be started and stopped from the Mining tab, with visual feedback of current mining status.
- **Mining Metrics:**  
  The app displays total rewards, hourly rates, completed challenges, and calculated efficiency—all live!

### Proof of Storage Demo
- **Local Self-Challenge:**  
  A Proof-of-Storage demo allows users to load a file fragment and verify its integrity, previewing future distributed storage capabilities.

### Guardians and Recovery
- **Guardian Registration:**  
  Users can register up to three guardians (2-of-3) for key recovery, all accessible and managed through the Security tab.
- **Simulated Recovery Flow:**  
  The GUI demonstrates guardian-assisted account recovery, allowing safe public key rotation in a multi-guardian scenario.

### UI and Usability
- **Responsive Tabs:**  
  The GUI now enables/disables tabs based on login status, preventing accidental access before account setup.
- **Configurable Network Mode:**  
  Easily switch between the global "central" network (port 369) and any custom subnet with just a few clicks—no need to restart.
- **Automatic Data Directory Creation:**  
  The client now automatically creates required folders (such as `.diac_dnc`) under your user directory if missing.

### Troubleshooting & Error Handling
- **Detailed Error Messages:**  
  All major actions now show user-friendly error messages, making debugging easier for new users.
- **Clear Startup Requirements:**  
  The README and codebase now guide you through missing dependencies (e.g., Tkinter) and common mistakes.

---

**These features collectively make DNC more robust, user-friendly, and secure—ready for both power users and newcomers.**


---

## Contributing and Forks

Forks are welcome!  
You are free to improve, update, and publish better versions of the code.

- Please submit **pull requests** with clear descriptions and rationale for your changes.
- For larger proposals, open an **issue** to discuss design choices before implementation.

**Suggested contribution areas:**
- Additional platforms and installers
- Discovery backends and NAT traversal helpers
- Extended wallet and token logic
- Performance and robustness improvements
- Documentation and examples

---

## License

This project is licensed under the MIT License. See the [LICENSE](./LICENSE) file for details.

Copyright (c) 2025  
**Kaoru Aguilera Katayama**

---

## Citation and Original Paper

For theoretical background and the broader framework, see the original paper:

**[https://doi.org/10.17605/OSF.IO/AYHPJ](https://doi.org/10.17605/OSF.IO/AYHPJ)**

If you build on this work, please **cite the paper and reference this repository**.

---

## Acknowledgments

Thanks Satoshi Nakamoto for inspiring me to create DNC

