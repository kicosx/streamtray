# StreamTray - Technical Documentation

## Overview

StreamTray is a modern C++/Qt6 system tray application for managing Twitch streams using streamlink and mpv, with integrated Twitch chat support.

**Version:** 2.0.5

---

## Architecture

### Technology Stack
- **Language**: C++17
- **GUI Framework**: Qt6 (Widgets + Network)
- **Build System**: CMake 3.16+
- **Target OS**: Linux (Ubuntu, Mint, Bazzite, CachyOS, Fedora)

### Component Diagram
```
┌─────────────────────────────────────────────────────────────────────┐
│                           StreamTray                                 │
├─────────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │   TrayIcon   │  │  MainWindow  │  │  ChatWindow  │               │
│  │   (System    │  │  (Main GUI)  │  │  (Chat GUI)  │               │
│  │    Tray)     │  │              │  │  [multiple]  │               │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘               │
│         │                 │                 │                        │
│  ┌──────┴─────────────────┴─────────────────┴──────┐                │
│  │              Core Services (Singletons)          │                │
│  ├──────────────────────────────────────────────────┤                │
│  │  FavoritesManager    │  Load/save favorites      │                │
│  │  StreamManager       │  Start/stop streams       │                │
│  │  StatusChecker       │  Check online status      │                │
│  │  DependencyManager   │  Download dependencies    │                │
│  │  Settings            │  App configuration        │                │
│  ├──────────────────────────────────────────────────┤                │
│  │              Chat Services (Singletons)          │                │
│  ├──────────────────────────────────────────────────┤                │
│  │  TwitchAuth          │  OAuth authentication     │                │
│  │  EmoteManager        │  Emote/badge caching      │                │
│  ├──────────────────────────────────────────────────┤                │
│  │              Per-Window Instances                │                │
│  ├──────────────────────────────────────────────────┤                │
│  │  TwitchChat          │  IRC client per channel   │                │
│  └──────────────────────────────────────────────────┘                │
└─────────────────────────────────────────────────────────────────────┘
```

---

## File Structure

```
/home/kicos/streamtray/
├── CMakeLists.txt              # CMake build configuration
├── README.md                   # Quick start guide
├── DOCUMENTATION.md            # This file
├── release/                    # Built packages
│   ├── StreamTray-x.x.x-x86_64.AppImage
│   └── streamtray_x.x.x_amd64.deb
├── resources/
│   ├── icons/
│   │   ├── tray-default.png
│   │   └── tray-active.png
│   └── streamtray.qrc
└── src/
    ├── main.cpp                # Application entry point
    ├── TrayIcon.cpp/h          # System tray icon
    ├── MainWindow.cpp/h        # Main window GUI
    ├── ChatWindow.cpp/h        # Chat window GUI (NEW)
    ├── FavoritesManager.cpp/h  # Favorites management
    ├── StreamManager.cpp/h     # Stream process control
    ├── StatusChecker.cpp/h     # Online status checking
    ├── DependencyManager.cpp/h # Dependency downloads
    ├── Settings.cpp/h          # Application settings
    ├── TwitchAuth.cpp/h        # OAuth authentication (NEW)
    ├── TwitchChat.cpp/h        # IRC chat client (NEW)
    └── EmoteManager.cpp/h      # Emote caching (NEW)
```

---

## Class Reference

### Settings (Singleton)
Manages application settings using QSettings.

```cpp
Settings& Settings::instance();

// Stream settings
QString quality() const;           // Stream quality (best, 720p, etc.)
void setQuality(const QString&);

int refreshInterval() const;       // Auto-refresh interval in seconds
void setRefreshInterval(int);

bool firstRun() const;             // First run flag
void setFirstRun(bool);

// Paths
QString binPath() const;           // ~/.local/share/streamtray/bin
QString streamlinkPath() const;    // Path to streamlink binary
QString mpvPath() const;           // Path to mpv binary
QString emoteCachePath() const;    // ~/.cache/streamtray/emotes

// Chat settings (NEW)
qreal chatOpacity() const;         // 0.1 - 1.0
void setChatOpacity(qreal);

bool chatAlwaysOnTop() const;
void setChatAlwaysOnTop(bool);

// Twitch authentication (NEW)
QString twitchOAuthToken() const;
void setTwitchOAuthToken(const QString&);

QString twitchUsername() const;
void setTwitchUsername(const QString&);

bool isLoggedIn() const;
void clearTwitchAuth();

// Signals
void settingsChanged();
void chatSettingsChanged();
void authChanged();
```

### TwitchAuth (Singleton) - NEW
Handles Twitch OAuth authentication via external browser.

```cpp
TwitchAuth& TwitchAuth::instance();

void startLogin();                  // Opens browser for OAuth
void setManualToken(const QString&); // Process pasted token/URL
void logout();
void validateToken();

bool isLoggedIn() const;
QString username() const;
QString oauthToken() const;
static QString clientId();

// Signals
void loginStarted();
void loginSuccess(const QString& username);
void loginFailed(const QString& error);
void loggedOut();
void tokenValidated(bool valid);
void tokenInputRequired();          // Show token input dialog
```

**OAuth Flow (automatic — `.deb` install or after `setcap`):**
1. App starts a local HTTP server on port 80 (`cap_net_bind_service` capability required)
2. Browser opens to Twitch OAuth page
3. User authenticates on Twitch
4. Twitch redirects to `http://localhost` — app captures the token
5. Browser shows "Login successful" — tab can be closed

**OAuth Flow (clipboard fallback — AppImage / raw binary):**
1. Browser opens to Twitch OAuth page
2. User authenticates — browser lands on an error page
3. User copies the full URL from the address bar (Ctrl+A, Ctrl+C)
4. App detects the token in clipboard and logs in automatically

### TwitchChat - NEW
IRC client for Twitch chat (one instance per ChatWindow).

```cpp
TwitchChat(const QString& channel, QObject* parent = nullptr);

void connectToChat();
void disconnect();
void sendMessage(const QString& message);

bool isConnected() const;
QString channel() const;

// Signals
void connected();
void disconnected();
void messageReceived(const ChatMessage& message);
void connectionError(const QString& error);
void userJoined(const QString& username);
void userLeft(const QString& username);
```

**ChatMessage Structure:**
```cpp
struct ChatMessage {
    QString username;
    QString displayName;
    QString message;
    QString color;                              // User color
    QList<QPair<QString, QString>> badges;      // Badge setId/version
    QList<QPair<QString, QPair<int, int>>> emotes; // emoteId, (start, end)
    bool isAction;                              // /me message
    qint64 timestamp;
};
```

### EmoteManager (Singleton) - NEW
Caches emote and badge images.

```cpp
EmoteManager& EmoteManager::instance();

QPixmap getEmote(const QString& emoteId);
QPixmap getBadge(const QString& setId, const QString& version);

void loadGlobalEmotes();
void loadChannelEmotes(const QString& channelId);
void loadGlobalBadges();
void loadChannelBadges(const QString& channelId);

bool hasEmote(const QString& emoteId) const;

// Signals
void emoteLoaded(const QString& emoteId);
void badgeLoaded(const QString& setId, const QString& version);
void globalEmotesLoaded();
void channelEmotesLoaded(const QString& channelId);
```

**Cache Location:** `~/.cache/streamtray/emotes/`

### ChatWindow - NEW
Frameless chat window with transparency support.

```cpp
ChatWindow(const QString& channel, QWidget* parent = nullptr);

QString channel() const;

// Signals
void closed(const QString& channel);
```

**Features:**
- Frameless window (no title bar)
- Draggable via left-click
- Right-click context menu (Minimize, Close)
- Adjustable background opacity
- Always-on-top toggle
- Per-channel window position saved

### FavoritesManager (Singleton)
Handles loading, saving, and managing favorites.

```cpp
struct Favorite {
    QString name;      // Short name (e.g., "poki")
    QString channel;   // Twitch channel (e.g., "pokimane")
    QDateTime addedAt; // When added
    bool online;       // Current online status
};

FavoritesManager& FavoritesManager::instance();

QList<Favorite>& favorites();
bool addFavorite(const QString& name, const QString& channel);
bool removeFavorite(const QString& name);
Favorite* findByName(const QString& name);
void setOnlineStatus(const QString& channel, bool online);
static QString extractChannel(const QString& url);
```

### StreamManager (Singleton)
Controls stream processes via QProcess.

```cpp
StreamManager& StreamManager::instance();

void startStream(const QString& channel, const QString& quality = "best");
void stopStream(const QString& channel);
void stopAllStreams();
bool isStreamActive(const QString& channel) const;
int activeStreamCount() const;
QStringList activeChannels() const;

// Signals
void streamStarted(const QString& channel);
void streamStopped(const QString& channel);
void streamError(const QString& channel, const QString& error);
void activeCountChanged(int count);
```

### StatusChecker (Singleton)
Checks if streamers are online using streamlink --json.

```cpp
StatusChecker& StatusChecker::instance();

void checkAll();
void checkChannel(const QString&);
void startAutoRefresh();
void stopAutoRefresh();
bool isChecking() const;

// Signals
void statusChecked(const QString& channel, bool online);
void checkStarted();
void checkFinished();
```

### TrayIcon
System tray icon with context menu.

```cpp
TrayIcon(MainWindow* mainWindow, QObject* parent = nullptr);

void show();
void updateMenu();
void setStreamingState(bool active);

// Signals
void quitRequested();
```

**Context Menu:**
- Online Streamers (submenu)
- Open Chat (submenu) - NEW
- Open StreamTray
- Refresh Status
- Close All Streams
- Login/Logout - NEW
- Quit

### MainWindow
Main application window with dark theme.

```cpp
MainWindow(QWidget* parent = nullptr);

void refreshFavorites();
void updateStatusBar();
void openChat(const QString& channel);  // NEW
```

---

## Data Formats

### favorites.json
```json
{
  "favorites": [
    {
      "name": "poki",
      "channel": "pokimane",
      "addedAt": "2024-01-15T10:30:00"
    }
  ]
}
```

### streamtray.conf (QSettings)
```ini
[twitch]
token=<obfuscated>
username=yourname
userId=12345

[chat]
opacity=0.9
alwaysOnTop=true

[stream]
quality=best

[status]
refreshInterval=60
```

---

## Build Instructions

### Prerequisites

**Ubuntu / Linux Mint:**
```bash
sudo apt install cmake qt6-base-dev build-essential
```

**Arch / Bazzite / CachyOS:**
```bash
sudo pacman -S cmake qt6-base base-devel
```

### Compile
```bash
cd /home/kicos/streamtray
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build
```

### Create Packages
```bash
# See release/ folder for .deb and .AppImage
```

---

## Configuration Files

| File | Purpose |
|------|---------|
| `~/.config/streamtray/favorites.json` | Stored favorites |
| `~/.config/streamtray.conf` | QSettings config (quality, auth, chat settings) |
| `~/.local/share/streamtray/bin/` | Downloaded binaries |
| `~/.cache/streamtray/emotes/` | Cached emote images |
| `/tmp/streamtray.lock` | Single instance lock file |

---

## Chat Architecture

### IRC Connection
- Host: `irc.chat.twitch.tv`
- Port: `6697` (TLS)
- Capabilities: `twitch.tv/tags`, `twitch.tv/commands`, `twitch.tv/membership`

### Message Flow
```
TwitchChat                    Twitch IRC
    │                             │
    │──── CAP REQ ───────────────►│
    │──── PASS oauth:token ──────►│
    │──── NICK username ─────────►│
    │                             │
    │◄─── 001 Welcome ────────────│
    │                             │
    │──── JOIN #channel ─────────►│
    │                             │
    │◄─── @tags PRIVMSG ──────────│  (messages with emotes, badges)
    │                             │
    │──── PRIVMSG #channel :hi ──►│  (send message)
    │                             │
```

---

## UI Components

### Chat Window
```
┌──────────────────────────────────────┐
│ Disconnected    Opacity:[===] [Pin]  │  ← Header (draggable)
├──────────────────────────────────────┤
│                                      │
│ 11:30 [sub] Username: Hello chat!    │
│ 11:31 OtherUser: Hi there            │
│ 11:32 * SomeUser waves               │  ← /me action
│                                      │
├──────────────────────────────────────┤
│ [Login] [        Send a message...  ] [Chat] │
└──────────────────────────────────────┘
```

### Color Scheme
| Element | Color |
|---------|-------|
| Window Background | rgba(24, 24, 27, opacity) |
| Chat Background | #0e0e10 |
| Accent | #9146ff (Twitch purple) |
| Text | #efeff1 |
| Online | #00ff9d |
| Timestamp | #6b6b6b |

---

## Security Notes

### OAuth Token Storage
- Token stored with XOR obfuscation (not cryptographic security)
- Stored in `~/.config/streamtray.conf`
- Only `chat:read` and `chat:edit` scopes requested

### Client ID
- Embedded in binary (visible via `strings`)
- Client ID is designed to be public
- Register your own at https://dev.twitch.tv/console/apps

---

## Troubleshooting

### "StreamTray is already running"
```bash
rm -f /tmp/streamtray.lock
```

### Chat not connecting
1. Check if logged in (Login button)
2. Validate token hasn't expired
3. Check internet connection

### No tray icon
- Some DEs need tray extensions
- Main window always opens as fallback

---

## Version History

### 2.0.2 (Current)
- Automatic OAuth login via local port 80 server (deb install)
- Clipboard monitoring fallback for AppImage and raw binary
- postinst grants `cap_net_bind_service` capability automatically

### 2.0.1
- Desktop notifications when a favorite goes live
- Rich stream info in list (game, viewer count, title)
- Following tab: add channels to Favorites with star button

### 2.0.0
- Following tab — browse Twitch followed channels
- Custom frameless window with dark title bar
- Twitch Helix API batch status checking (replaces streamlink processes)
- Configurable auto-refresh interval
- Chat: opacity applies to background only
- Chat: resizable from all edges and corners
- Chat: own messages appear immediately

### 1.0.x
- Initial releases
- Stream management, favorites, chat windows, OAuth
