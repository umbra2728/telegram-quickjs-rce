# telegram-quickjs-rce

RCE in Telegram Desktop via bot widgets. Works on **Windows x64** and **macOS arm64**.

Telegram Desktop 7.4.0–7.4.2 runs bot widgets in a sandbox powered by QuickJS. A use-after-free in the generator prologue ([quickjs-ng #1910](https://github.com/quickjs-ng/quickjs/issues/1910)) gives arbitrary read/write inside the widget sandbox. The PoC escapes the sandbox and launches the Calculator from the Telegram Desktop process.

Both PoCs use the same UAF primitive, only the final call differs per OS:

| File | OS | Payload | SHA-256 |
|---|---|---|---|
| `poc-win.html` | Windows x64 | `WinExec` → `calc.exe` | `3e2cbc65f8dcb29e2ae78879554f72ad7f3bf48509467d43d60e1e2f346d3347` |
| `poc-macos.html` | macOS arm64 | `system("open -a Calculator")` | `ac512ff672ac894ed074c126b1f331ae2d739ddf0dfcb6f37541ba78ecfd4f2b` |

Tested on Telegram Desktop 7.4.2 (official builds, both platforms).

## Usage

Send the PoC to yourself as a widget via the Bot API (`sendWidget`), then open the chat in a vulnerable Telegram Desktop:

```bash
curl -s -X POST "https://api.telegram.org/bot<TOKEN>/sendWidget" \
  -H "Content-Type: application/json" \
  -d "$(python3 -c 'import json; print(json.dumps({"chat_id": <YOUR_ID>, "html": open("poc-win.html").read()}))')"
```

When you open the message in the chat, the Calculator opens and Telegram Desktop keeps running.

## Credits

- Original research and Windows PoC: [bikini/telejam](https://github.com/bikini/telejam)
- macOS port: [@umbra2728](https://github.com/umbra2728)
