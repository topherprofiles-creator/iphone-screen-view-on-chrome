---
name: iphone
description: Opens a URL in your main Chrome tab emulated as an iPhone (default iPhone 16, 393x852 at 3x, mobile + touch + iOS Safari user agent) and leaves it that way. Use when the user types /iphone <url>, or asks to see a page on an iPhone / mobile screen. `/iphone off` returns the tab to desktop.
---

# iphone

`/iphone <url>` puts the page in **your own, logged-in Chrome** into an iPhone 16 viewport and keeps it there. It is a viewing aid for checking mobile layouts, not a test harness.

## Steps

1. **Work out the URL.** Use the argument. No scheme: `http://` for `localhost` / `127.0.0.1` / `*.test`, otherwise `https://`. No argument: reuse the URL of the tab you last worked in, or ask for one if there is none.
2. **Use your live Chrome via the `mcp__chrome-devtools__*` tools.** Never kill, relaunch, copy profiles, or pass `isolatedContext`: the point is your real session (already signed in).
3. **Page ids change whenever the browser reconnects**, so always call `list_pages` first and never reuse an id from earlier in the conversation.
4. **Pick the tab.** Reuse an open tab whose URL is on the same origin as the target (same host and port); that is your main tab. Open a new one with `new_page` only if none matches. Do not touch unrelated tabs.
5. **Emulate, then navigate** (emulation first so the page loads as mobile from the first request):
   - `emulate` with `viewport: "393x852x3,mobile,touch"` and
     `userAgent: "Mozilla/5.0 (iPhone; CPU iPhone OS 18_0 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/18.0 Mobile/15E148 Safari/604.1"`
   - `navigate_page` with `type: "url"` to the target. If the tab is already on it, `type: "reload"`.
6. **Confirm with one `take_screenshot`** and reply with a single line (which tab, which device). Do not describe the page unless asked.

## Keep it that way

- **Never call `resize_page` on that tab afterwards.** It overrides the emulation and drops the tab back to a desktop-sized window. Navigating, reloading and clicking are fine: the emulation persists.
- If the user later asks for a screenshot or check of that page, just use it as is.

## Devices

| Argument | Viewport |
|---|---|
| (none) / `16` | `393x852x3,mobile,touch` |
| `16-pro-max` / `max` | `440x956x3,mobile,touch` |
| `se` | `375x667x2,mobile,touch` |

A device word may come before or after the URL (`/iphone se http://localhost/...`). Keep the same user agent string for all of them.

## Turn it off

`/iphone off` (or "back to desktop"): `emulate` with `userAgent: ""` and `viewport: "1440x900x1"`, then reload the tab.

## Limits worth knowing

DevTools emulation gives the right screen size, pixel density, touch and user agent, but it is **not Safari**: no notch or Dynamic Island, no home-indicator safe-area insets, and Chromium's rendering and form controls rather than WebKit's. Anything that depends on `env(safe-area-inset-*)`, `100vh` behaviour under Safari's collapsing toolbar, or iOS-only input quirks needs a real iPhone.
