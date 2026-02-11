<p align="center">
  <img src="https://lexu.gg/favicon.svg" alt="Lexu Logo" width="120" height="120" />
</p>

<h1 align="center">Lexu</h1>

<p align="center">
  <strong>Your League of Legends Companion — Right in Your Pocket</strong>
</p>

<p align="center">
  <a href="https://lexu.gg">🌐 lexu.gg</a> &nbsp;·&nbsp;
  <a href="#architecture">Architecture</a> &nbsp;·&nbsp;
  <a href="#how-the-connector-works">Connector</a> &nbsp;·&nbsp;
  <a href="#security--data-privacy">Security</a> &nbsp;·&nbsp;
  <a href="#getting-started">Getting Started</a>
</p>

<br />

<p align="center">
  <img src="https://img.shields.io/badge/Electron-34.2.0-47848F?logo=electron&logoColor=white" alt="Electron" />
  <img src="https://img.shields.io/badge/React_Native-Expo_SDK_54-000020?logo=expo&logoColor=white" alt="Expo" />
  <img src="https://img.shields.io/badge/Express.js-4.x-000000?logo=express&logoColor=white" alt="Express" />
  <img src="https://img.shields.io/badge/MongoDB-8.x-47A248?logo=mongodb&logoColor=white" alt="MongoDB" />
  <img src="https://img.shields.io/badge/Socket.IO-Realtime-010101?logo=socket.io&logoColor=white" alt="Socket.IO" />
  <img src="https://img.shields.io/badge/Astro-5.x-FF5D01?logo=astro&logoColor=white" alt="Astro" />
</p>

---

## What is Lexu?

**Lexu** is a mobile companion app for League of Legends that lets you control champion select, monitor live game data, and interact with the LoL client — all from your phone.

It works by running a lightweight **desktop connector** that bridges your League of Legends client with the Lexu mobile app over your local network. No cloud relay, no account credentials, no data leaves your LAN.

> **Visit [lexu.gg](https://lexu.gg) to download and learn more.**

---

## Architecture

Lexu is built as a **multi-package monorepo** with four core components working together:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          LEXU ECOSYSTEM                                 │
│                                                                         │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐              │
│  │   lexu-astro │    │ lexu-express │    │  lexu-expo   │              │
│  │  Landing Page│    │   Backend    │    │  Mobile App  │              │
│  │   (Astro)    │    │  (Express)   │    │ (React Native│              │
│  │              │    │              │    │    + Expo)   │              │
│  └──────────────┘    └──────┬───────┘    └──────┬───────┘              │
│                             │                    │                      │
│                             │  Sync & Rights     │  LAN Connection      │
│                             │                    │  (HTTP + Socket.IO)  │
│                             │                    │                      │
│                      ┌──────┴────────────────────┴───────┐             │
│                      │         lexu-electron              │             │
│                      │     Desktop Connector (Bridge)     │             │
│                      │                                    │             │
│                      │  ┌──────────┐  ┌───────────────┐  │             │
│                      │  │ HTTP     │  │ Socket.IO     │  │             │
│                      │  │ Proxy    │  │ Server        │  │             │
│                      │  └────┬─────┘  └───────┬───────┘  │             │
│                      │       │                │          │             │
│                      └───────┼────────────────┼──────────┘             │
│                              │                │                        │
│                              ▼                ▼                        │
│                      ┌────────────────────────────────┐               │
│                      │    League of Legends Client     │               │
│                      │         (LCU API)               │               │
│                      └────────────────────────────────┘               │
└─────────────────────────────────────────────────────────────────────────┘
```

## How the Connector Works

The **Lexu Connector** (lexu-electron) is the core of the system. It acts as a secure local bridge between the League of Legends client and your mobile device — no Riot account credentials are ever needed.

### Connection Flow

```
 ┌──────────────────────────────────────────────────────────────────┐
 │                     YOUR COMPUTER (LAN)                          │
 │                                                                  │
 │  ┌─────────────────────┐         ┌─────────────────────────┐    │
 │  │  League of Legends   │         │    Lexu Connector        │    │
 │  │      Client          │◄───────►│    (Electron App)        │    │
 │  │                      │  LCU    │                           │    │
 │  │  Port: {dynamic}     │  API    │  ┌───────────────────┐   │    │
 │  │  Auth: riot:{token}  │◄──────► │  │  Local HTTP Server │   │    │
 │  │                      │  REST   │  │  Port: 42587       │   │    │
 │  │                      │         │  └─────────┬─────────┘   │    │
 │  │                      │  WS     │            │             │    │
 │  │                      │◄──────► │  ┌─────────┴─────────┐   │    │
 │  │                      │  Events │  │  Socket.IO Server  │   │    │
 │  └─────────────────────┘         │  │  (Real-time)       │   │    │
 │                                   │  └─────────┬─────────┘   │    │
 │                                   │            │             │    │
 │                                   │      QR Code with       │    │
 │                                   │      LAN IP:Port        │    │
 │                                   └────────────┬────────────┘    │
 │                                                │                 │
 └────────────────────────────────────────────────┼─────────────────┘
                                                  │
                                        Wi-Fi / LAN Only
                                                  │
                                   ┌──────────────┴──────────────┐
                                   │      Lexu Mobile App         │
                                   │         (Expo)               │
                                   │                              │
                                   │  • Scan QR or enter IP:Port  │
                                   │  • Champion Select controls  │
                                   │  • Live game monitoring      │
                                   │  • Real-time updates         │
                                   └──────────────────────────────┘
```

### Step-by-Step Process

```
    ┌─────────┐          ┌──────────┐          ┌──────────┐          ┌──────────┐
    │  Step 1  │          │  Step 2   │          │  Step 3   │          │  Step 4   │
    │          │          │           │          │           │          │           │
    │  Launch  │────────► │  Detect   │────────► │  Display  │────────► │  Control  │
    │  League  │          │  & Bridge │          │  QR Code  │          │  from     │
    │  Client  │          │           │          │           │          │  Phone    │
    │          │          │  Electron │          │  Scan or  │          │           │
    │          │          │  reads    │          │  enter    │          │  Champion │
    │          │          │  lockfile │          │  IP:Port  │          │  Select,  │
    │          │          │  & connects│         │  on phone │          │  Game     │
    │          │          │  to LCU   │          │           │          │  Monitor  │
    └─────────┘          └──────────┘          └──────────┘          └──────────┘
```

**1. Lockfile Detection** — When the League client starts, it creates a `lockfile` containing a dynamic port and authentication token. The Lexu Connector detects the `LeagueClientUx.exe` process, locates the install directory, and reads this lockfile.

**2. LCU API Bridge** — Using the lockfile credentials, the connector establishes two connections to the League Client Update (LCU) API:
  - **REST API** — For request/response operations (champion select actions, summoner info, etc.)
  - **WebSocket** — For real-time event streaming (game phase changes, lobby updates, etc.)

**3. Local Server** — The connector starts a local HTTP server + Socket.IO server on your machine. A QR code is generated containing your LAN IP and port.

**4. Mobile Connection** — The mobile app scans the QR code (or manually enters the IP:Port) and connects. All communication flows through the local network:

```
Mobile App  ──HTTP Request──►  Connector  ──Proxy──►  LCU API
Mobile App  ◄──Socket.IO────  Connector  ◄──WS─────  LCU Events
```

### Live Game Data

During an active match, the connector also taps into the **Live Client Data API** (port 2999) to provide real-time in-game statistics — player scores, gold, items, and more — streamed directly to the mobile app.

---

## Security & Data Privacy

Security is a first-class citizen in Lexu's architecture. Here's why you can trust it:

### No Riot Credentials Required

```
  ╔═══════════════════════════════════════════════════════╗
  ║              WHAT LEXU NEVER DOES                     ║
  ║                                                       ║
  ║  ✗  Ask for your Riot username or password            ║
  ║  ✗  Access your Riot account credentials              ║
  ║  ✗  Store any authentication tokens remotely          ║
  ║  ✗  Send game data to external servers                ║
  ║  ✗  Modify game files or inject code                  ║
  ╚═══════════════════════════════════════════════════════╝
```

### LAN-Only Communication

All communication between the mobile app and the connector happens **exclusively over your local network (LAN/Wi-Fi)**. Data never leaves your home network.

```
  ╔═══════════════════════════════════════════════════════╗
  ║              HOW DATA FLOWS                           ║
  ║                                                       ║
  ║  LoL Client  ◄──localhost──►  Connector               ║
  ║                                    │                   ║
  ║                              LAN / Wi-Fi Only          ║
  ║                                    │                   ║
  ║                               Mobile App               ║
  ║                                                       ║
  ║  ✓  All data stays on your local network              ║
  ║  ✓  No cloud relay or third-party routing             ║
  ║  ✓  No internet connection needed for core features   ║
  ╚═══════════════════════════════════════════════════════╝
```

### Riot's Own API — Used as Intended

Lexu uses the **LCU (League Client Update) API**, which is Riot's own local API exposed by the League client. This is the same API that:

- Riot's own client UI uses internally
- Third-party tools like Overwolf apps utilize
- Is documented and acknowledged by Riot

The connector reads the **lockfile** — a standard mechanism provided by the League client — to obtain the local port and authentication token. This is a **read-only, local-only process** that does not bypass any security measures.

### No Game Manipulation

Lexu operates strictly as a **companion tool**. It reads game state and sends actions through the official LCU API. It does not:

- Inject code into the game process
- Modify game memory or files
- Automate gameplay
- Provide unfair competitive advantages

### Data Stored Remotely

The only data sent to the Lexu backend server is:

| Data | Purpose |
|------|---------|
| Machine ID | Anonymous device identification for licensing |
| Summoner name & tag | Display purposes within the app |
| App version | Ensuring compatibility and update checks |

No gameplay data, match history, or personal information is stored on external servers.

---

## Getting Started

### Prerequisites

- **League of Legends** installed and running on your PC
- **Lexu Connector** downloaded from [lexu.gg](https://lexu.gg)
- **Lexu Mobile App** installed on your phone
- PC and phone connected to the **same Wi-Fi / LAN network**

### Quick Start

```
1.  Launch League of Legends on your PC
2.  Open the Lexu Connector — it auto-detects the League client
3.  A QR code appears on your screen
4.  Open the Lexu mobile app and scan the QR code
5.  You're connected! Start using Lexu from your phone
```

```
   PC                              Phone
   ┌────────────┐                  ┌────────────┐
   │            │                  │            │
   │  League +  │   ┌──────────┐  │   Lexu     │
   │  Lexu      │──►│ QR Code  │◄─│   Mobile   │
   │  Connector │   └──────────┘  │   App      │
   │            │                  │            │
   └────────────┘                  └────────────┘
       Same Wi-Fi / LAN Network
```

---

## License

This project is proprietary software. All rights reserved.

Visit [lexu.gg](https://lexu.gg) for more information.

---

<p align="center">
  <strong>Built with ❤️ for the League of Legends community</strong>
  <br />
  <a href="https://lexu.gg">lexu.gg</a>
</p>
