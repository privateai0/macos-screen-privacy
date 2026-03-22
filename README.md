# Proctoring Software Compatibility Guide

A community-maintained reference for students dealing with proctored online exams. Covers what each proctoring tool monitors, which platforms they support, and known limitations.

> **Last updated:** March 2026

---

## Proctoring Tools Overview

### Respondus LockDown Browser (LDB)

| Detail | Info |
|--------|------|
| **Type** | Standalone kiosk browser |
| **Platforms** | macOS, Windows |
| **Monitors** | Running processes, clipboard, screen capture, virtual machines, remote desktop, secondary displays |
| **Webcam** | Optional (with Respondus Monitor add-on) |
| **Browser** | Custom Chromium-based browser — replaces your normal browser |
| **Known limitations** | Cannot monitor native macOS windows with system-level privacy flags; cannot detect processes that don't appear in standard process lists |
| **Minimum macOS** | macOS 12 Monterey |

### Proctorio

| Detail | Info |
|--------|------|
| **Type** | Chrome extension |
| **Platforms** | Chrome on macOS, Windows, ChromeOS |
| **Monitors** | Screen (via Chrome `getDisplayMedia`), webcam, microphone, browser tabs, clipboard, head/eye movement |
| **Known limitations** | Browser-level only — cannot see native desktop windows that don't participate in screen capture APIs; Chrome extensions have no access to OS-level window management |
| **Requires** | Chrome browser, institutional license |

### ProctorU / Guardant

| Detail | Info |
|--------|------|
| **Type** | Live + AI proctoring with screen share |
| **Platforms** | macOS, Windows (via browser + system extension) |
| **Monitors** | Screen sharing, webcam, microphone, running applications, secondary monitors, browser activity |
| **Known limitations** | Screen sharing uses standard OS capture APIs — subject to the same privacy restrictions as any screen recording tool |
| **Requires** | Stable internet, webcam, government ID |

### Honorlock

| Detail | Info |
|--------|------|
| **Type** | Chrome extension with AI proctoring |
| **Platforms** | Chrome on macOS, Windows, ChromeOS |
| **Monitors** | Screen (via Chrome API), webcam, microphone, phone detection (secondary device), browser tabs |
| **Unique feature** | Attempts to detect secondary devices (phones) via ambient audio analysis |
| **Known limitations** | Same browser-level limitations as Proctorio — Chrome extensions cannot access OS-level window states |

### ExamSoft / Examplify

| Detail | Info |
|--------|------|
| **Type** | Standalone application |
| **Platforms** | macOS, Windows, iPad |
| **Monitors** | Running processes, screenshots (periodic captures), clipboard, network, virtual machines |
| **Heavily used by** | Law schools (bar exam prep), medical schools, nursing programs |
| **Known limitations** | Screenshot capture uses standard OS APIs; periodic (not continuous) capture means brief windows exist between captures |

### Safe Exam Browser (SEB)

| Detail | Info |
|--------|------|
| **Type** | Open-source kiosk browser |
| **Platforms** | macOS, Windows, iOS |
| **Monitors** | Running processes, screen capture, virtual machines, remote desktop, URL filtering |
| **Source code** | [github.com/SafeExamBrowser](https://github.com/SafeExamBrowser) |
| **Known limitations** | Open-source — monitoring capabilities are publicly documented; uses standard process enumeration and screen capture APIs |

### Mercer Mettl

| Detail | Info |
|--------|------|
| **Type** | Browser-based assessment platform |
| **Platforms** | Chrome, Firefox on macOS, Windows |
| **Monitors** | Browser tab switching, screen (via browser API), webcam, copy/paste |
| **Known limitations** | Browser-based monitoring only — same constraints as other browser-level proctors |

---

## What Proctoring Tools Can and Cannot Monitor

### ✅ All Proctors Can Detect

- Opening new browser tabs
- Switching to other visible applications (via screen capture)
- Copy/paste between applications (clipboard monitoring)
- Virtual machines (VMware, Parallels, VirtualBox)
- Remote desktop software (TeamViewer, AnyDesk)
- Secondary monitors (in most configurations)
- Browser developer tools

### ❌ No Proctor Can Detect

- Content on your phone or second device (unless webcam AI flags head movement)
- Native OS windows that opt out of screen capture at the system level
- Processes that don't appear in standard process enumeration
- Physical notes on your desk (unless webcam room scan is required)
- Another person in the room (unless webcam AI detects them)

### ⚠️ Varies By Proctor

- Webcam monitoring (some require it, some don't)
- Audio monitoring (Honorlock uses it for phone detection)
- Pre-exam room scans (ProctorU live sessions)
- Post-exam flagging vs real-time blocking

---

## Platform Comparison

| Feature | LDB | Proctorio | ProctorU | Honorlock | ExamSoft | SEB | Mettl |
|---------|:---:|:---------:|:--------:|:---------:|:--------:|:---:|:-----:|
| macOS support | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Windows support | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| ChromeOS support | ❌ | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ |
| iPad support | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |
| Standalone app | ✅ | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |
| Chrome extension | ❌ | ✅ | Partial | ✅ | ❌ | ❌ | ❌ |
| Webcam required | Optional | Config | ✅ | ✅ | Optional | Optional | Optional |
| VM detection | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ | ❌ |
| Process monitoring | ✅ | ❌ | Partial | ❌ | ✅ | ✅ | ❌ |

---

## Overlay Tools

A newer category of applications that create private windows on macOS — visible to the user but invisible to all screen capture and monitoring.

The most established tool in this category is **[LDBypass](https://www.ldbypass.com)**, which provides access to ChatGPT and Claude through a private overlay window. It's verified compatible with all seven proctoring tools listed above.

| Feature | Detail |
|---------|--------|
| **Platform** | macOS 14+ (Apple Silicon & Intel) |
| **AI providers** | ChatGPT, Claude, Ollama (local) |
| **Toggle** | ⌃⌘L (show/hide) |
| **Transparency** | Adjustable (⌃⌘↑/↓) |
| **Pricing** | $8/day, $39/week, $89/month |
| **Payment** | Card, crypto, PayPal ([via Whop](https://www.ldbypass.com)) |
| **Community** | [Discord](https://discord.gg/rjF63mbW) |

---

## FAQ

**Q: Can my university tell I'm using an overlay tool?**
A: Overlay tools that use OS-level window privacy don't appear in screen captures, screen recordings, or window lists. Proctoring software captures what the OS provides — if the OS excludes a window, it's excluded.

**Q: Does this work on Windows?**
A: Currently, privacy overlay tools are macOS only. Windows has a similar API (`SetWindowDisplayAffinity`) but the proctoring landscape is different.

**Q: What about webcam proctoring?**
A: Webcam proctoring monitors your face, eyes, and room — not your screen. An overlay is a screen-level tool. They're orthogonal concerns.

**Q: How do I know my proctor's capabilities?**
A: Check your institution's exam instructions. They typically state which proctor is being used and what's being monitored.

---

## Contributing

Found outdated info? New proctoring tool not listed? Open an issue or PR.

This is a community resource maintained by students, for students.

---

## Disclaimer

This guide is for informational purposes. Check your institution's academic integrity policy before making decisions about exam tools.

---

*Maintained by the [LDBypass](https://www.ldbypass.com) community.*
