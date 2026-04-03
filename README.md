# macOS Screen Privacy & Proctoring Software Guide

A comprehensive guide to how proctoring software monitors your Mac, what each platform detects, and how macOS screen privacy APIs work.

![macOS](https://img.shields.io/badge/macOS-12%2B-000?logo=apple&logoColor=white)
![Updated](https://img.shields.io/badge/Updated-April%202026-3fb950)

---

## Proctoring Software Overview

### What Do Proctors Monitor?

| Capability | LockDown Browser | Proctorio | ProctorU | Honorlock | ExamSoft | SEB |
|---|---|---|---|---|---|---|
| Lock to single app | Yes | No | No | No | Yes | Yes |
| Screen recording | Yes | Yes | Yes | Yes | Yes | No |
| Window list monitoring | Yes | No | Yes | No | Yes | Yes |
| Webcam recording | No | Yes | Yes | Yes | No | No |
| Browser extension | No | Yes | No | Yes | No | No |
| Clipboard monitoring | Yes | Yes | No | No | Yes | Yes |

### Respondus LockDown Browser

LockDown Browser is a custom Chromium-based browser that restricts macOS to a single-app environment. It uses `CGWindowListCopyWindowInfo` to enumerate visible windows, monitors for screen capture, and disables app-switching.

**Platform**: macOS 10.15+, Windows 10+, iPadOS 14+
**Detection level**: Application-level (cannot see below window server)

### Proctorio

Chrome extension that monitors via `chrome.tabCapture` and `chrome.desktopCapture` APIs. Records webcam/mic via WebRTC. Cannot access macOS system-level APIs.

**Platform**: Chrome browser (all OS)
**Detection level**: Browser-level only

### ProctorU / Meazure Learning

Browser extension + optional native helper. Screen shares via WebRTC `getDisplayMedia`. Native helper can enumerate windows.

**Platform**: Chrome + optional native helper
**Detection level**: Browser-level + optional application-level

### Honorlock

Chrome extension using `desktopCapture` API. Monitors for phone use via AI webcam analysis. Cannot access macOS window server.

**Platform**: Chrome browser
**Detection level**: Browser-level only

### ExamSoft / Examplify

Native macOS application. Creates kiosk-mode, enumerates windows via CGWindowList, monitors screen capture activity.

**Platform**: macOS 10.15+, Windows 10+, iPadOS
**Detection level**: Application-level

### Safe Exam Browser (SEB)

Open-source kiosk browser. Locks system to single window, monitors running processes, configurable window list checking.

**Platform**: macOS 10.15+, Windows 10+, iPadOS
**Detection level**: Application-level (configurable)

---

## macOS Screen Privacy APIs

### CGWindowList

Primary window enumeration API. `CGWindowListCopyWindowInfo` returns all on-screen windows with ID, name, owner PID, bounds, and sharing type. Windows can be excluded by setting sharing type via CoreGraphics SPI.

### Screen Capture APIs

- `CGDisplayStream` - low-level display stream capture
- `CGWindowListCreateImage` - screenshot of specific windows
- `SCScreenshotManager` (ScreenCaptureKit) - modern capture framework
- `AVCaptureScreenInput` - AVFoundation screen recording

All respect the window sharing type.

### Window Sharing Type

`CGSSetWindowSharingState` controls window visibility in capture:

| Value | Behavior |
|---|---|
| 0 (None) | Excluded from capture and window lists |
| 1 (ReadOnly) | Visible in lists but not capturable |
| 2 (ReadWrite) | Normal - fully visible and capturable |

This is the same mechanism macOS uses for system UI elements.

---

## Resources

- [LDBypass](https://ldbypass.com) - Invisible AI overlay for macOS
- [Discord Community](https://discord.gg/saMaxDk5)
- [Apple ScreenCaptureKit Docs](https://developer.apple.com/documentation/screencapturekit)

*Maintained by the LDBypass team. Last updated April 2026.*
