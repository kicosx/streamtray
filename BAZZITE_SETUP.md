# StreamTray on Bazzite

Bazzite is an immutable Fedora-based OS running Wayland. StreamTray works fine on it, but needs two small adjustments compared to a standard Linux install.

---

## 1. Install dependencies

Bazzite uses `rpm-ostree` instead of `dnf` — changes require a reboot:

```bash
sudo rpm-ostree install streamlink mpv
systemctl reboot
```

---

## 2. Download

Get the latest release from the [GitHub Releases page](https://github.com/kicosx/streamtray/releases).

Choose either the **AppImage** or the **raw binary**.

---

## 3. AppImage setup

The AppImage does not bundle the Qt Wayland platform plugin. Without the fix below, the app may fail to start or **chat/main windows cannot be moved**.

Force XWayland instead:

```bash
chmod +x StreamTray-x86_64.AppImage
QT_QPA_PLATFORM=xcb ./StreamTray-x86_64.AppImage
```

**Autostart setup:**

```bash
mkdir -p ~/.config/autostart
cat > ~/.config/autostart/streamtray.desktop << EOF
[Desktop Entry]
Type=Application
Name=StreamTray
Exec=env QT_QPA_PLATFORM=xcb /full/path/to/StreamTray-x86_64.AppImage
Icon=streamtray
Comment=Twitch stream monitor
X-GNOME-Autostart-enabled=true
Hidden=false
NoDisplay=false
EOF
```

Replace `/full/path/to/` with the actual location of the AppImage file.

**Optional — fully automatic OAuth login:**

```bash
sudo setcap 'cap_net_bind_service=ep' /full/path/to/StreamTray-x86_64.AppImage
```

---

## 4. Raw binary setup

```bash
chmod +x streamtray-x86_64
sudo cp streamtray-x86_64 /usr/local/bin/streamtray
```

The binary has the same Wayland issue — force XWayland when running it directly:

```bash
QT_QPA_PLATFORM=xcb streamtray
```

**Autostart setup:**

```bash
mkdir -p ~/.config/autostart
cat > ~/.config/autostart/streamtray.desktop << EOF
[Desktop Entry]
Type=Application
Name=StreamTray
Exec=env QT_QPA_PLATFORM=xcb streamtray
Icon=streamtray
Comment=Twitch stream monitor
X-GNOME-Autostart-enabled=true
Hidden=false
NoDisplay=false
EOF
```

**Optional — fully automatic OAuth login:**

```bash
sudo setcap 'cap_net_bind_service=ep' /usr/local/bin/streamtray
```

---

## 5. Twitch login

Without `setcap`, use the clipboard fallback — see [Twitch Login](README.md#twitch-login) in the main README.
