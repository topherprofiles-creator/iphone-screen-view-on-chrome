---
name: iphone
description: Opens a URL in the owner's main Chrome tab emulated as an iPhone (default iPhone 16, 393x852 at 3x, mobile + touch + iOS Safari user agent) and leaves it that way. Use when the user types /iphone <url>, or asks to see a page on an iPhone / mobile screen. `/iphone off` returns the tab to desktop.
---

# iphone

`/iphone <url>` puts the page in the owner's **own, logged-in Chrome** into an iPhone 16 viewport and keeps it there. It is a viewing aid for checking mobile layouts, not a test harness.

## Steps

1. **Work out the URL.** Use the argument. No scheme: `http://` for `localhost` / `127.0.0.1` / `*.test`, otherwise `https://`. No argument: reuse the URL of the tab you last worked in, or ask for one if there is none.
2. **Use the owner's live Chrome via the `mcp__chrome-devtools__*` tools.** Never kill, relaunch, copy profiles, or pass `isolatedContext`: the point is the owner's real session (already signed in).
3. **Page ids change whenever the browser reconnects**, so always call `list_pages` first and never reuse an id from earlier in the conversation.
4. **Pick the tab.** Reuse an open tab whose URL is on the same origin as the target (same host and port); that is "my main tab". Open a new one with `new_page` only if none matches. Do not touch unrelated tabs.
5. **Measure the real window, then fit the phone inside it.** The emulated height must never exceed the visible window, otherwise the bottom of the phone is cut off and can never be scrolled into view (a 495px-tall window was showing only 58% of an 852px phone: bottom nav buttons missing, page "not scrolling" like desktop). Open a throwaway tab (`new_page` with `about:blank`, no `isolatedContext`), read `innerHeight` with `evaluate_script`, then `close_page` it. Use `height = min(device height, innerHeight)`. **Never read it from a tab you have already emulated**: that reports the emulated size, not the window. `/iphone full` skips this and uses the true device height.
6. **Emulate, then navigate** (emulation first so the page loads as mobile from the first request):
   - `emulate` with `viewport: "393x<height from step 5>x3,mobile,touch"` and
     `userAgent: "Mozilla/5.0 (iPhone; CPU iPhone OS 18_0 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/18.0 Mobile/15E148 Safari/604.1"`
   - `navigate_page` with `type: "url"` to the target. If the tab is already on it, `type: "reload"`.
7. **Confirm with one `take_screenshot`** and reply with a single line (which tab, which device). Do not describe the page unless asked.

## Keep it that way

- **Never call `resize_page` on that tab afterwards.** It overrides the emulation and drops the tab back to a desktop-sized window. Navigating, reloading and clicking are fine: the emulation persists.
- If the owner later asks for a screenshot or check of that page, just use it as is.

## Devices

| Argument | Viewport |
|---|---|
| (none) / `16` | `393x852x3,mobile,touch` |
| `16-pro-max` / `max` | `440x956x3,mobile,touch` |
| `se` | `375x667x2,mobile,touch` |

The heights above are each device's full height: the height actually used is `min(that, window innerHeight)` unless `full` is given (step 5).

A device word may come before or after the URL (`/iphone se http://localhost/...`). Keep the same user agent string for all of them.

## Turn it off

`/iphone off` (or "back to desktop"): `emulate` with `userAgent: ""` and `viewport: "1440x900x1"`, then reload the tab.

## Limits worth knowing

DevTools emulation gives the right screen size, pixel density, touch and user agent, but it is **not Safari**: no notch or Dynamic Island, no home-indicator safe-area insets, and Chromium's rendering and form controls rather than WebKit's. Anything that depends on `env(safe-area-inset-*)`, `100vh` behaviour under Safari's collapsing toolbar, or iOS-only input quirks needs a real iPhone.
