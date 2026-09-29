# 🏦 FocusBank

![Status](https://img.shields.io/badge/status-WIP-orange?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)
![Platform](https://img.shields.io/badge/platform-Android%20%7C%20Kotlin-green?style=flat-square)
![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat-square)

> **An open-source Android app that turns focus into currency: earn coins by verifiably studying, working, or exercising — spend them on doomscrolling, or don't.**

---

## 📑 Table of Contents

- [Overview](#overview)
- [How It Works](#-how-it-works)
- [Points Economy](#-points-economy)
- [Doomscroll Blocklist](#-doomscroll-blocklist)
- [Privacy & Data](#-privacy--data)
- [Permissions Required](#-permissions-required)
- [Architecture](#️-architecture)
- [Configuration](#️-configuration)
- [Dependencies](#-dependencies)
- [Known Limitations](#-known-limitations)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#-license)

---

## Overview

FocusBank flips the usual "screen time limit" app on its head. Instead of just counting minutes and guilt-tripping you, it runs a **coin economy**: verified focus time (studying, working, exercising) earns coins, and opening a doomscroll-prone app (Instagram, TikTok, etc.) spends them. Everyone gets a small free daily allowance no matter what — beyond that, **no coins means no scroll**. There's no bypass button, no "just this once."

```
Focus session (camera-verified) → earn coins
Doomscroll app opened → spend coins (after free daily minutes run out)
Coins hit zero → blocklisted apps get hard-blocked until you earn more
```

---

## 🔍 How It Works

| Mode | Trigger | Verification |
| :--- | :--- | :--- |
| **Earning** (study / work / exercise) | User manually starts a focus session | Front camera confirms a person is actually present, with a lightweight liveness check (subtle motion/blink) so a static photo propped up can't fake it |
| **Spending** (doomscrolling) | User opens a blocklisted app | Screen/app-usage monitoring only — no camera involved here |

A **10-minute daily allowance** on blocklisted apps is always free, regardless of coin balance — the point isn't to ban social media outright, it's to make *excess* scrolling cost something.

---

## 🪙 Points Economy

> [!NOTE]
> The exact rates below are a starting proposal, not final — this is the first thing to tune based on real usage. See [Roadmap](#roadmap).

| Activity | Rate (proposed) |
| :--- | :--- |
| Verified focus (study/work/exercise) | +1 coin / minute |
| Blocklisted app usage, beyond the free 10 min/day | −2 coins / minute |
| Balance at zero | Blocklisted apps hard-blocked until next earning session |

---

## 🚫 Doomscroll Blocklist

Doomscroll apps are defined by an **explicit blocklist** (Instagram, TikTok, and similar), not by analyzing on-screen content — that would be a much bigger privacy line to cross for very little gain in accuracy.

> [!NOTE]
> A blocklist is a blunt instrument — it can't distinguish "reading a curated tutorial thread" from "endless scroll" within the same app. This is a deliberate trade-off: less invasive, but less precise. Open to discussion in [Roadmap](#roadmap).

---

## 🔐 Privacy & Data

- **Open source** — every line of code that touches the camera or your usage data is publicly auditable. If you don't trust it, read it.
- Camera frames used for presence/liveness detection are processed **on-device only** and are never stored or transmitted.
- No account, no cloud sync in v1 — your coin balance lives on your device.

> [!WARNING]
> Camera-based presence detection can still be spoofed with enough deliberate effort (e.g., a looping video instead of a static photo). The liveness check raises the bar, it doesn't make cheating impossible — see [Known Limitations](#-known-limitations).

---

## 🔑 Permissions Required

| Permission | Why |
| :--- | :--- |
| Camera | Presence/liveness check during earning sessions only — never active in the background |
| Usage Access (`PACKAGE_USAGE_STATS`) | Detect which app is currently in the foreground |
| Accessibility Service | Enforce the hard block on doomscroll apps when balance hits zero |
| Foreground Service notification | Keeps the app alive during an active focus session |

---

## 🏗️ Architecture

```
CameraX (liveness check) ──┐
                            ├──> Session Manager ──> Coin Ledger (local DB)
UsageStatsManager ──────────┘                              │
                                                             ▼
AccessibilityService ──> enforces block when balance == 0
```

---

## ⚙️ Configuration

```kotlin
const val FREE_DOOMSCROLL_MINUTES_PER_DAY = 10
const val EARN_RATE_COINS_PER_MINUTE = 1
const val SPEND_RATE_COINS_PER_MINUTE = 2

val BLOCKLISTED_APPS = listOf(
    "com.instagram.android",
    "com.zhiliaoapp.musically", // TikTok
    "com.google.android.youtube"
)
```

> [!CAUTION]
> These rates and the blocklist are placeholders for early testing, not a final balance. Expect to tune them heavily once real usage data comes in.

---

## 📦 Dependencies

- [CameraX](https://developer.android.com/training/camerax) — camera capture for presence/liveness detection
- [ML Kit Face Detection](https://developers.google.com/ml-kit/vision/face-detection) — lightweight on-device liveness signal (blink/motion), no cloud calls
- [Room](https://developer.android.com/training/data-storage/room) — local coin ledger storage

> [!IMPORTANT]
> Library versions are not pinned. If a dependency updates and breaks the build, lock versions in your package manager.

---

## ⚠️ Known Limitations

Being upfront about this, since the project is open source:

- **Presence ≠ focus.** The camera confirms a person is in front of the phone, not that they're actually studying. Someone can prop the phone up and work on a different device instead.
- **Liveness detection raises the bar, doesn't remove the gap.** A determined person can still defeat it with a looping video instead of a photo.
- **Blocklist is app-level, not content-level.** It can't tell productive use of a listed app apart from mindless scrolling within it.
- **Exercise doesn't fit the "camera facing you" model well.** A phone propped up to watch you exercise is a different setup than one on a desk while you study — this needs its own detection approach, not yet designed.
- **Nothing stops someone from just uninstalling the app**, or revoking the Accessibility permission, when their coins run out. Self-monitoring tools are opt-in by nature — there's no way around that with an app alone.

---

## Roadmap

> [!NOTE]
> None of the items below are finalized — this is a planning list for future versions, not current behavior.

| Milestone | Target | Status |
| :---: | :--- | :---: |
| M1 | Tune the final point economy (earn/spend rates, free daily minutes) based on real testing | 🔲 Planned |
| M2 | Design a separate detection approach for exercise sessions (camera-facing model doesn't fit) | 🔲 Planned |
| M3 | Community-maintained, opt-in doomscroll blocklist (beyond the hardcoded default) | 🔲 Planned |
| M4 | Stats dashboard (focus time vs. doomscroll time, streaks) | 🔲 Planned |
| M5 | Stronger liveness detection (harder to spoof with looping video) | 🔲 Planned |
| M6 | iOS port, if Screen Time API constraints allow equivalent enforcement | 🔲 Planned |

---

## Contributing

Contributions are very welcome — economy balance ideas, detection improvements, UI, docs, anything.

1. Fork the repo
2. Create a feature branch: `git checkout -b feat/your-feature`
3. Commit your changes: `git commit -m "feat: describe your change"`
4. Push and open a Pull Request against `main`

Please open an issue first for anything larger than a bug fix, so we can discuss direction before you invest time building it — especially around the point economy and blocklist, since those are the most opinionated and least settled parts of the project.

> [!IMPORTANT]
> When contributing anything touching the Accessibility Service or camera pipeline, test on a real device — emulator behavior for both differs meaningfully from real hardware.

---

## 📄 License

This project is licensed under the **MIT License**. See [LICENSE](./LICENSE) for the full text.

---
