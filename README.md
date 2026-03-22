# macOS Screen Privacy API Reference

A comprehensive reference for developers building screen-capture-proof applications on macOS.

Covers `NSWindow.sharingType`, `CGWindowList` behavior, `ScreenCaptureKit` interactions, and practical patterns for building overlay applications that are invisible to screen recordings, screenshots, and screen shares.

---

## Quick Reference

| API | Privacy Effect | Available Since |
|-----|---------------|-----------------|
| `NSWindow.sharingType = .none` | Window excluded from all capture | macOS 10.5+ |
| `NSWindow.sharingType = .readOnly` | Window visible but not modifiable | macOS 10.5+ |
| `CGWindowListCopyWindowInfo` | Lists all `.readWrite` / `.readOnly` windows | macOS 10.5+ |
| `SCStreamConfiguration.excludesCurrentProcessAudio` | Audio privacy control | macOS 13+ |
| `SCShareableContent` | Enumerates capturable windows (respects sharingType) | macOS 12.3+ |

---

## Window Sharing Types

### `.none` — Full Privacy

```swift
window.sharingType = .none
```

**What it does:**
- Removes window from `CGWindowListCopyWindowInfo` results
- Removes window from `CGWindowListCreateImage` compositing
- Removes window from `SCShareableContent.windows`
- Removes window from ScreenCaptureKit capture streams
- Window does NOT appear in any screen recording, screenshot, or screen share

**Use cases:** Password fields, private overlays, DRM content, sensitive data display

### `.readOnly` — Visible but Protected

```swift
window.sharingType = .readOnly
```

**What it does:**
- Window appears in screen captures (visible in recordings)
- Other processes cannot modify the window's content
- Still appears in `CGWindowListCopyWindowInfo`

**Use cases:** Content you want visible in recordings but protected from tampering

### `.readWrite` — Default

```swift
window.sharingType = .readWrite
```

The default for all new windows. Fully visible and accessible.

---

## Verification Methods

### Check if Your Window is Hidden

Start a QuickTime screen recording, then verify your private window doesn't appear in the capture.

### Programmatic Verification

```swift
func isWindowHiddenFromCapture(windowID: CGWindowID) -> Bool {
    guard let list = CGWindowListCopyWindowInfo(
        .optionAll, kCGNullWindowID
    ) as? [[String: Any]] else { return true }
    
    return !list.contains { info in
        (info[kCGWindowNumber as String] as? Int) == Int(windowID)
    }
}
```

### ScreenCaptureKit Verification (macOS 12.3+)

```swift
import ScreenCaptureKit

func checkSCKVisibility() async {
    let content = try? await SCShareableContent.current
    let windows = content?.windows ?? []
    // Private windows (.none) will NOT appear in this list
    print("Capturable windows: \(windows.count)")
}
```

---

## Common Patterns

### Pattern 1: Private Overlay Window

A floating window that's visible to the user but invisible to all capture.

```swift
class PrivateOverlay {
    let window: NSWindow
    
    init(frame: NSRect) {
        window = NSWindow(
            contentRect: frame,
            styleMask: [.borderless],
            backing: .buffered,
            defer: false
        )
        window.sharingType = .none
        window.level = .floating
        window.collectionBehavior = [.canJoinAllSpaces, .fullScreenAuxiliary]
        window.isOpaque = false
        window.backgroundColor = .clear
        window.ignoresMouseEvents = false
        window.orderFrontRegardless()
    }
}
```

### Pattern 2: Keyboard Input Without Key Window

Private windows that need keyboard input face a challenge: becoming the key window emits notifications observable by other apps. Solution: use a separate input receiver.

```swift
class KeyboardRelay: NSPanel {
    var target: NSView?
    
    init() {
        super.init(
            contentRect: NSRect(x: -9999, y: -9999, width: 1, height: 1),
            styleMask: [.nonactivatingPanel],
            backing: .buffered,
            defer: false
        )
        self.sharingType = .readWrite  // Needs to be key-capable
        self.level = .floating
        self.makeKeyAndOrderFront(nil)
    }
    
    override func keyDown(with event: NSEvent) {
        // Route keyboard input to your private window's content
        target?.keyDown(with: event)
    }
}
```

### Pattern 3: Adjustable Transparency

Allow the user to see through the overlay:

```swift
func adjustTransparency(delta: CGFloat) {
    let current = window.alphaValue
    let new = max(0.1, min(1.0, current + delta))
    window.alphaValue = new
}

// Bind to hotkeys:
// ⌃⌘↑ = increase opacity
// ⌃⌘↓ = decrease opacity
```

### Pattern 4: WebKit Content in Private Window

Render web content (e.g., a web app) inside a private window:

```objc
#import <WebKit/WebKit.h>

WKWebViewConfiguration *config = [[WKWebViewConfiguration alloc] init];
config.websiteDataStore = [WKWebsiteDataStore defaultDataStore]; // Shares Safari cookies

WKWebView *webView = [[WKWebView alloc] initWithFrame:window.contentView.bounds
                                         configuration:config];
[webView loadRequest:[NSURLRequest requestWithURL:
    [NSURL URLWithString:@"https://chatgpt.com"]]];

[window.contentView addSubview:webView];
window.sharingType = NSWindowSharingNone; // Private
```

### Pattern 5: Process Name Cloaking

Make your app's process harder to identify:

```swift
// In Info.plist, set CFBundleExecutable to a system-like name
// The process will appear in Activity Monitor with that name

// At runtime, you can also modify argv[0]:
func cloakProcessName(_ name: String) {
    name.withCString { ptr in
        let mutable = UnsafeMutablePointer(mutating: ptr)
        ProcessInfo.processInfo.setValue(name, forKey: "processName")
    }
}
```

---

## Capture API Compatibility Matrix

How different capture methods interact with window sharing types:

| Capture Method | `.readWrite` | `.readOnly` | `.none` |
|---------------|:---:|:---:|:---:|
| CGWindowListCreateImage | ✅ Captured | ✅ Captured | ❌ Hidden |
| CGWindowListCopyWindowInfo | ✅ Listed | ✅ Listed | ❌ Hidden |
| ScreenCaptureKit (SCStream) | ✅ Captured | ✅ Captured | ❌ Hidden |
| SCShareableContent.windows | ✅ Listed | ✅ Listed | ❌ Hidden |
| QuickTime Screen Recording | ✅ Captured | ✅ Captured | ❌ Hidden |
| OBS (macOS 13+) | ✅ Captured | ✅ Captured | ❌ Hidden |
| Zoom Screen Share | ✅ Captured | ✅ Captured | ❌ Hidden |
| Chrome getDisplayMedia | ✅ Captured | ✅ Captured | ❌ Hidden |
| Accessibility API | ✅ Access | ✅ Access | ⚠️ Metadata only |
| External camera | ✅ Visible | ✅ Visible | ✅ Visible |

---

## Platform Comparison

| Feature | macOS | Windows | Linux |
|---------|-------|---------|-------|
| API | `NSWindow.sharingType` | `SetWindowDisplayAffinity` | None (standard) |
| Available since | 10.5 (2007) | Windows 7 | — |
| Hides from screenshots | ✅ | ✅ | ❌ |
| Hides from recordings | ✅ | ✅ | ❌ |
| Hides from window list | ✅ | ❌ | ❌ |
| Per-window control | ✅ | ✅ | ❌ |
| Reversible at runtime | ✅ | ✅ | — |

---

## Gotchas and Edge Cases

### 1. Setting sharingType after makeKeyAndOrderFront

```swift
// ❌ BAD — may not take effect immediately
window.makeKeyAndOrderFront(nil)
window.sharingType = .none

// ✅ GOOD — set before showing
window.sharingType = .none
window.orderFrontRegardless()
```

### 2. sharingType resets on makeKeyAndOrderFront

Some AppKit calls reset `sharingType`. Use `orderFrontRegardless()` instead of `makeKeyAndOrderFront(_:)` for private windows.

### 3. Child windows inherit parent's sharingType

If you add a child window, it inherits the parent's sharing type. Set it explicitly if you want different behavior.

### 4. NSPanel vs NSWindow

`NSPanel` with `.nonactivatingPanel` style mask is ideal for overlays that shouldn't steal focus from other apps.

### 5. Full-screen apps

Private windows can use `.fullScreenAuxiliary` collection behavior to remain visible when another app is in full-screen mode:

```swift
window.collectionBehavior = [.canJoinAllSpaces, .fullScreenAuxiliary]
```

---

## Real-World Implementations

- **1Password** — Master password entry windows use `.none`
- **macOS Lock Screen** — System password field uses private windowing
- **[LDBypass](https://www.ldbypass.com)** — AI overlay tool that renders ChatGPT/Claude in a private window for use during proctored exams. Full implementation of patterns 1–5 above.
- **Touch Bar** — Touch Bar content windows use SPI=0 (same underlying mechanism)

---

## Further Reading

- [Apple Developer Documentation: NSWindow.SharingType](https://developer.apple.com/documentation/appkit/nswindow/sharingtype)
- [CGWindowListCopyWindowInfo](https://developer.apple.com/documentation/coregraphics/1455137-cgwindowlistcopywindowinfo)
- [ScreenCaptureKit Framework](https://developer.apple.com/documentation/screencapturekit)
- [NSPanel Documentation](https://developer.apple.com/documentation/appkit/nspanel)

---

## License

This reference is provided under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Use it, share it, build on it.

---

*Built and maintained by the team behind [LDBypass](https://www.ldbypass.com) — the invisible AI overlay for macOS.*
