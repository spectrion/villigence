![Logo](https://spectrion.github.io/assets/IMG_4306.png)

----------------------------

Players install Villigence once. Any game using the SDK automatically checks for it before matchmaking — no extra installs per game, ever.

---

## How it works

```
Player installs Villigence AntiCheat extension (once)
           ↓
Game loads villigence.js SDK
           ↓
SDK pings extension via postMessage
           ↓
Extension found → session starts → player can matchmake
Extension missing → overlay appears → player prompted to install
Unsupported browser → overlay appears → dev's choice (block / offline)
           ↓
Extension monitors for cheats in background
           ↓
Cheat detected → fires onCheat() → dev handles it
```

---

## Structure

```
villigence/
├── extension/     ← Sideload this once in Chrome/Edge/Brave/Firefox/Opera
│   ├── manifest.json
│   ├── background.js
│   ├── content.js
│   └── popup.html
├── sdk/
│   └── villigence.js   ← Drop into your game
└── demo/
    └── index.html      ← Working demo
```

---

## Sideload the Extension

### Chrome, Edge, Brave, Opera, Vivaldi
1. Open `chrome://extensions` (or `edge://extensions`)
2. Enable **Developer Mode** (top right)
3. Click **Load unpacked** → select the `extension/` folder
4. Done — Villigence is now active for all games

### Firefox
1. Open `about:debugging#/runtime/this-firefox`
2. **Load Temporary Add-on** → select `extension/manifest.json`

---

## Integrate into Your Game

### 1. Add the script
```html
<script src="villigence.js"></script>
```

### 2. Init before matchmaking
```javascript
Villigence.init({
  gameId:   'your-game-id',   // required
  gameName: 'Your Game',

  // Behaviour when extension not installed:
  // 'block'   → overlay shown, can't play online
  // 'offline' → overlay shown, can dismiss to offline mode
  // 'allow'   → no check (not recommended for competitive)
  noExtension: 'block',

  // Behaviour on unsupported browser (Safari etc):
  noSupport: 'offline',

  // Detection toggles
  detectSpeedHack:    true,
  detectAutoClicker:  true,
  detectDevTools:     false,   // up to you
  detectTimeManip:    true,
  detectHeadless:     true,

  // Thresholds
  clickRateThreshold: 20,      // clicks/sec
  speedHackThreshold: 1.5,     // ratio
  timeDriftThreshold: 2000,    // ms

  // What happens on detection
  onCheat:     'both',         // 'block' | 'report' | 'both'
  reportUrl:   'https://yourserver.com/api/cheats',
  reportSecret: 'your-secret',

  // Overlay customization
  accentColor: '#3dff6e',      // your game's brand color
  installUrl:  'https://villigence.dev/install',
})

.onReady(({ sessionId }) => {
  // Extension verified, session active
  enableMatchmakingUI();
})

.onMissing(() => {
  // Extension not installed — overlay already shown
})

.onUnsupported(() => {
  // Safari / unsupported browser — overlay already shown
})

.onCheat((event) => {
  // event.type:       'speed_hack' | 'auto_clicker' | 'dev_tools' | 'time_manip' | 'headless'
  // event.severity:   'low' | 'medium' | 'high'
  // event.detail:     description
  // event.action:     'block' | 'report'
  // event.totalFlags: total this session
  if (event.action === 'block') kickFromMatch();
});
```

### 3. Gate matchmaking with one line
```javascript
playButton.addEventListener('click', () => {
  if (!Villigence.canMatchmake()) return; // overlay shown automatically
  joinQueue();
});
```

---

## Report Endpoint Payload

```json
{
  "gameId": "your-game-id",
  "gameName": "Your Game",
  "violation": {
    "type": "auto_clicker",
    "severity": "high",
    "detail": "32 clicks/sec (limit: 20)",
    "value": 32,
    "timestamp": 1706000000000
  },
  "totalFlags": 3,
  "browser": "Chrome",
  "villigenceVersion": "1.0.0"
}
```

---

## Supported Browsers

| Browser  | Extensions | Villigence |
|----------|-----------|------------|
| Chrome   | ✅ | ✅ |
| Edge     | ✅ | ✅ |
| Brave    | ✅ | ✅ |
| Opera    | ✅ | ✅ |
| Vivaldi  | ✅ | ✅ |
| Firefox  | ✅ | ✅ |
| Safari   | ❌ | Offline only |
| Mobile Chrome/Safari | ❌ | Offline only |
| Kiwi (Android) | ✅ | ✅ |

---

## Mobile Games

- **Kiwi Browser** (Android) supports Chrome extensions — full Villigence support
- **Standard mobile browsers** don't support extensions — use `noSupport: 'offline'` to let mobile players play in offline/solo mode
- For native mobile apps (React Native etc) — use `Villigence.report()` for manual reporting only

----------

This Is In Active Dev And Anyone Can Use This In Their Game
