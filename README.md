# Robert Remote

Browser remote and choreography playground for the ALLNET **"Robert" dancing robot**. No app, no install: open the page, click Connect, make it dance.

**→ Live:** https://lda.io/robert-remote/

Needs **Chrome or Edge** (Windows, macOS, Android, ChromeOS). Firefox and iOS browsers have no Web Bluetooth. Close the phone app first, the robot accepts one connection at a time.

## Features

- Hold-to-drive arrows (also arrow keys / WASD, space = stop)
- LED color, speed, eye blink
- Every built-in move: 93 dances, 11 hand moves, 35 leg moves
- Choreography tab: write short JavaScript scripts, with presets

```js
color('red'); speed(1)
await forward(800)
await dance()          // random dance
await hand(3)          // hand move #3
await blink(true); await wait(3000); await blink(false)
```

Also scriptable from DevTools via `robot`.

## Protocol

Reverse engineered from the Android app `cn.app.robert` 1.5.4.

- BLE device name `Robert`, service `0xFFC0`, write characteristic `0xFFC1`
- Every command is one 17-byte packet:

```
AA AA CC 32 01 | A A T S C E 02 02 | 01 01 | 55 55
```

| Field | Meaning |
|---|---|
| A | action: 1-93 dances, 100-110 hand, 200-235 leg (no 231), **77 = stop**. Driving (seen from Robert's face) = 1 forward, 2 back, 3 right, 4 left |
| T | beats 2/4/8/12, `FF` = keep going until next command |
| S / C / E | speed 0 fastest > 1 > 2 > 3 slowest · color 1 dark blue, 2 blue, 3 green, 4 yellow, 5 red, 6 purple, 7 white · eyes 0 steady, 4 slow blink |

The robot stores no programs; the app streams one move at a time.

Unofficial project, not affiliated with ALLNET.
