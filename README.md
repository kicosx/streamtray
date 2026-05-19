# StreamTray

A lightweight Qt6 system tray app for monitoring and watching Twitch streams on Linux. Track favorites, browse your followed channels, open chat windows, and get desktop notifications when channels go live — all from the system tray.

> **No Twitch account required to get started.** You can add channels to your favorites and monitor streams right away. A Twitch account is only needed for the Following tab, chat, and desktop notifications.

![Version](https://img.shields.io/badge/version-2.0.5-blue)
![Platform](https://img.shields.io/badge/platform-Linux-green)
![Qt](https://img.shields.io/badge/Qt-6-41cd52)

> **Disclaimer:** This is an unofficial, independent project. It is not affiliated with, endorsed by, or sponsored by Twitch Interactive, Inc. or Amazon.com, Inc. "Twitch" is a trademark of Twitch Interactive, Inc.

---

## Screenshots & Features

### Main Application

<p align="center">
  <img src="BilderTray/StreamTray.png?raw=true" alt="StreamTray Main Window" width="480"/>
</p>

StreamTray is a lightweight Twitch desktop companion that lets you monitor, manage, and watch your favorite streams without leaving your desktop.

| | |
|---|---|
| **Stream management** | Start and stop streams with a double-click |
| **Multi-stream support** | Watch multiple channels at the same time |
| **Multi-chat support** | Open independent chat windows per channel |
| **Live status** | See who's online at a glance from the main window |

---

### Settings & Player Configuration

<p align="center">
  <img src="BilderTray/StreamTraySettings.png?raw=true" alt="StreamTray Settings" width="480"/>
</p>

Tailor the app to your workflow through the built-in Settings panel.

| | |
|---|---|
| **Video player choice** | Switch between MPV and VLC Media Player |
| **Stream quality** | Select best, 1080p60, 720p, and more |
| **Auto-refresh interval** | Choose 1, 2, 5, or 10 minutes — or disable |
| **Customizable experience** | Every setting persists across restarts |

---

### Multiple Streams & Chats

<p align="center">
  <img src="BilderTray/StreamMultipleChats.png?raw=true" alt="Multiple Streams and Chats" width="640"/>
</p>

Open as many streams or chat windows as you want — each one runs independently, so you can follow multiple communities at once.

| | |
|---|---|
| **Parallel streams** | Watch several channels simultaneously |
| **Independent chats** | Each chat window is fully separate |
| **No limit** | Open as many streams and chats as you like |
| **Flexible layout** | Arrange windows freely across your desktop |

---

### System Tray Controls

<p align="center">
  <img src="BilderTray/StreamTrayIcon.png?raw=true" alt="StreamTray System Tray Icon" width="320"/>
</p>

Everything is accessible straight from the system tray — no need to open the main window.

| | |
|---|---|
| **Online streamer overview** | See live channels instantly from the tray menu |
| **Open streams from tray** | Launch a stream with one click |
| **Open chats from tray** | Jump into chat without opening the main window |
| **Login / logout** | Manage your session directly from the tray |
| **Always available** | Sits quietly in the taskbar until you need it |

---

## Features

- System tray icon with live/offline status for all favorites
- Favorites list with rich stream info (game, viewer count, title)
- **Following tab** — browse your Twitch followed channels, add to favorites
- **Desktop notifications** when a favorite goes live
- **Integrated chat windows** with emote support and adjustable opacity
- **Auto-refresh** via Twitch Helix API (configurable interval)
- **Twitch OAuth login** — fully automatic on `.deb` installs

---

## Download

| Format | Distro | Link |
|--------|--------|------|
| `.deb` | Ubuntu, Linux Mint, Debian | [GitHub Releases](https://github.com/kicosx/streamtray/releases) |
| AppImage | Any Linux (Arch, Fedora, ...) | [GitHub Releases](https://github.com/kicosx/streamtray/releases) |
| Binary | Any Linux | [GitHub Releases](https://github.com/kicosx/streamtray/releases) |
| Bazzite | Bazzite / immutable Fedora | [Bazzite setup guide](BAZZITE_SETUP.md) |

---

## Installation

### Option 1 — .deb package (Ubuntu / Linux Mint) ✅ Recommended

```bash
sudo dpkg -i streamtray_2.0.5-1_amd64.deb
sudo apt-get install -f   # install any missing dependencies
```

The package automatically:
- Installs the binary to `/usr/bin/streamtray`
- Adds a system-wide autostart entry so StreamTray starts on login
- Grants permission to bind port 80 for seamless OAuth login

**Autostart** is enabled by default. To disable it for your user only:

```bash
cp /etc/xdg/autostart/streamtray.desktop ~/.config/autostart/
```

Then open `~/.config/autostart/streamtray.desktop` and set `Hidden=true`.

---

### Option 2 — AppImage (any distro)

```bash
chmod +x StreamTray-2.0.5-x86_64.AppImage
./StreamTray-2.0.5-x86_64.AppImage
```

**Autostart setup:**

```bash
mkdir -p ~/.config/autostart
cat > ~/.config/autostart/streamtray.desktop << EOF
[Desktop Entry]
Type=Application
Name=StreamTray
Exec=/full/path/to/StreamTray-2.0.5-x86_64.AppImage
Icon=streamtray
Comment=Twitch stream monitor
X-GNOME-Autostart-enabled=true
Hidden=false
NoDisplay=false
EOF
```

Replace `/full/path/to/` with the actual location of the AppImage file.

**Optional — enable fully automatic OAuth login** (otherwise the clipboard fallback is used, see [Twitch Login](#twitch-login)):

```bash
sudo setcap 'cap_net_bind_service=ep' /full/path/to/StreamTray-2.0.5-x86_64.AppImage
```

---

### Option 3 — Raw binary (any distro)

```bash
chmod +x streamtray-2.0.5-x86_64
sudo cp streamtray-2.0.5-x86_64 /usr/local/bin/streamtray
```

**Autostart setup:**

```bash
mkdir -p ~/.config/autostart
cat > ~/.config/autostart/streamtray.desktop << EOF
[Desktop Entry]
Type=Application
Name=StreamTray
Exec=streamtray
Icon=streamtray
Comment=Twitch stream monitor
X-GNOME-Autostart-enabled=true
Hidden=false
NoDisplay=false
EOF
```

**Optional — enable fully automatic OAuth login:**

```bash
sudo setcap 'cap_net_bind_service=ep' /usr/local/bin/streamtray
```

---

## Twitch Login

StreamTray uses Twitch OAuth to show your followed channels, fetch live status via the Helix API, and connect to chat.

### Automatic (`.deb` install, or after running `setcap`)

1. Click **Login** in the app
2. Authorize in the browser that opens
3. The app captures the token automatically — the browser shows "Login successful"
4. You can close the browser tab

### Clipboard fallback (AppImage / raw binary without `setcap`)

1. Click **Login** — browser opens to Twitch authorization
2. Authorize on Twitch — the browser lands on an error page (this is normal)
3. Press **Ctrl+A** then **Ctrl+C** on the URL in the address bar
4. The app detects the token in your clipboard and logs in automatically

---

## Usage

### Watching Streams

- **Double-click** a favorite to start/stop the stream (via streamlink + mpv)
- Right-click the **tray icon** for quick access to all online streamers
- Click **Refresh** to check status immediately

### Following Tab

Switch to the **Following** tab to see all channels you follow on Twitch (requires login). Click the **☆** button on any channel to add it to your Favorites.

### Chat

Click the **Chat** button next to any channel to open a dedicated chat window. Chat windows support:
- Adjustable background opacity (text stays fully visible)
- Resize from all edges and corners
- Your own messages appear immediately after sending

### Settings

Click **Settings** to configure:
- Stream quality (best, 1080p60, 720p, etc.)
- Auto-refresh interval (1 / 2 / 5 / 10 minutes) or disable

---

## Building from Source

### Dependencies

**Ubuntu / Linux Mint:**
```bash
sudo apt install cmake qt6-base-dev qt6-base-dev-tools build-essential libcap2-bin
```

**Arch / CachyOS / Bazzite:**
```bash
sudo pacman -S cmake qt6-base base-devel libcap
```

### Build

```bash
git clone https://github.com/kicosx/streamtray.git
cd streamtray
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j$(nproc)
sudo cp build/streamtray /usr/local/bin/
sudo setcap 'cap_net_bind_service=ep' /usr/local/bin/streamtray
```

### Build .deb

```bash
dpkg-buildpackage -us -uc -b
sudo dpkg -i ../streamtray_*.deb
```

### Build AppImage

Requires `linuxdeploy` and `linuxdeploy-plugin-qt`:

```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release && cmake --build build -j$(nproc)

mkdir -p AppDir/usr/bin AppDir/usr/share/applications AppDir/usr/share/icons/hicolor/256x256/apps
cp build/streamtray AppDir/usr/bin/
cp resources/icons/tray-default.png AppDir/usr/share/icons/hicolor/256x256/apps/streamtray.png

QMAKE=/usr/lib/qt6/bin/qmake6 linuxdeploy \
  --appdir AppDir \
  --plugin qt \
  --library /usr/lib/x86_64-linux-gnu/libssl.so.3 \
  --library /usr/lib/x86_64-linux-gnu/libcrypto.so.3 \
  --output appimage
```

> **Note:** Always pass `libssl.so.3` and `libcrypto.so.3` explicitly. linuxdeploy blacklists them by default but they are required for Twitch API and chat on all distros.

---

## Performance

A single status refresh makes one HTTPS request to the Twitch Helix API and costs:

| Metric | Value |
|--------|-------|
| TCP packets | 121 |
| Sent (request + TLS) | ~55 KB |
| Received (API response) | ~13 KB |
| Total per refresh | ~68 KB |
| CPU spike | ~1% briefly |
| CPU at idle | ~0.3% |

A single refresh costs about 68 KB of traffic and a brief 1% CPU blip gone in under a second. At the default 2-minute interval that's roughly **2 MB/hour** — negligible even on a slow connection.

The sent size is larger than received because of TLS handshake overhead and the HTTP request URL containing all channel names as query parameters (`?user_login=x&user_login=y&...`). The response is compact JSON — only live channels are returned, so if few are live it stays small.

---

## Files

| Path | Description |
|------|-------------|
| `~/.config/streamtray/favorites.json` | Saved favorites |
| `~/.config/streamtray.conf` | App settings (quality, auth, intervals) |
| `~/.local/share/streamtray/bin/` | Auto-downloaded streamlink/mpv |
| `~/.cache/streamtray/emotes/` | Cached emote images |

---

## Requirements

- Linux x86_64
- `streamlink` and `mpv` for watching streams:
  ```bash
  sudo apt install streamlink mpv       # Ubuntu/Mint
  sudo pacman -S streamlink mpv         # Arch/CachyOS
  ```
- Twitch account for login, chat, and Following tab (optional)

---

## Privacy

StreamTray stores the following data locally on your machine:

- **Configuration file:** `~/.config/streamtray.conf` contains your settings, favorite channels list, and (if logged in) your Twitch OAuth token
- **No data is sent to third parties.** All communication is directly with Twitch's official API
- **No telemetry or analytics.** The app does not collect or transmit usage data

To remove all stored data, delete `~/.config/streamtray.conf`.

---

## License

Copyright (c) 2024-2026 kicos. All rights reserved.

This software is proprietary. No part may be copied, modified, or distributed without the prior written permission of the copyright owner.

This software dynamically links against Qt6, which is licensed under the GNU Lesser General Public License v3 (LGPL v3). See [THIRD_PARTY_LICENSES.md](THIRD_PARTY_LICENSES.md) for details.
