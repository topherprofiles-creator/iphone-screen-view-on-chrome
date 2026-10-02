# iphone-screen-view-on-chrome

A [Claude Code](https://claude.com/claude-code) skill. Type:

```
/iphone http://localhost:3000/dashboard
```

and your **own, already signed-in Chrome tab** switches to an iPhone 16 screen (393 x 852 at 3x, mobile, touch, iOS Safari user agent) and loads that page. It stays that way while you navigate and reload, so you can check a mobile layout in the browser you actually use, with your real session, without opening a separate emulator window.

```
/iphone http://localhost/app/dashboard.php      iPhone 16
/iphone max https://example.com                 iPhone 16 Pro Max (440 x 956)
/iphone se http://localhost:8080                iPhone SE (375 x 667)
/iphone off                                     back to desktop
```

## Requirements

- Claude Code
- The [Chrome DevTools MCP server](https://github.com/ChromeDevTools/chrome-devtools-mcp) connected to the Chrome you have open (for example with `--autoConnect`), so the skill drives your live browser rather than launching a new one

## Install

Copy the skill folder into your Claude Code skills directory:

```bash
git clone https://github.com/topherprofiles-creator/iphone-screen-view-on-chrome.git
mkdir -p ~/.claude/skills
cp -r iphone-screen-view-on-chrome/iphone ~/.claude/skills/
```

On Windows PowerShell:

```powershell
Copy-Item -Recurse .\iphone-screen-view-on-chrome\iphone $env:USERPROFILE\.claude\skills\
```

Then `/iphone <url>` is available in your next Claude Code session.

## What it does, step by step

1. Resolves the URL (adds `http://` for `localhost`, `https://` otherwise).
2. Lists the open tabs fresh every time, because tab ids change whenever Chrome reconnects.
3. Reuses the open tab on the same address, so you stay signed in. It only opens a new tab if none matches, and never touches unrelated tabs.
4. Applies the emulation first and then navigates, so the page loads as mobile from the very first request.
5. Takes one screenshot to confirm.

The emulation survives navigation, reloads and clicks. Resizing the tab's window overrides it, so the skill never does.

## Limits

This is Chrome pretending to be an iPhone, not Safari. You get the right screen size, pixel density, touch events and user agent. You do **not** get the notch or Dynamic Island, home-indicator safe-area insets, WebKit rendering, or Safari's collapsing toolbar behaviour. Anything that depends on `env(safe-area-inset-*)`, `100vh` under Safari's toolbar, or iOS-only input quirks needs a real iPhone.

## License

[MIT](LICENSE)
