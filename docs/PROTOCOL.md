# Robert BLE protocol

How the ALLNET "Robert" dancing robot is controlled over Bluetooth Low Energy.

Reverse engineered from the Android app `cn.app.robert` 1.5.4 (decompiled with jadx), then **tested on a real robot**: commands sent from Python (bleak), results checked by eye and on camera. Everything marked *tested* was confirmed that way; everything else comes only from the app's code.

## Connection

| | |
|---|---|
| Device | advertises service `0xFFC0` (name `Robert`, not always in the advert) |
| Write | characteristic `0xFFC1` (write, write-without-response) |
| Notify | characteristic `0xFFC2`, robot → app replies (see type 13) |
| Links | **one connection at a time**: close the phone app before connecting from elsewhere |
| Audio | separate classic Bluetooth speaker; the ear buttons are play/pause and next track for that |

The app writes in chunks of at most 18 bytes, 20 ms apart. Every packet below fits in one write.

## Packet frame

```
AA AA CC | type | count | items... | 55 55
```

| Type | Direction | Item format | Status |
|---|---|---|---|
| `0x32` | app → robot | 8 bytes + `01 01` | main command, *tested* |
| `0x03` | app → robot | 3 bytes `A T S` | short command, no color/eyes, *tested*: plays the move |
| `0x0C` | app → robot | none (count 0) | stop and report beats, *tested* |
| `0x0D` | robot → app | 1 byte | beats report, *tested*; sending it to the robot does nothing (*tested*) |

## Main command (type 0x32)

17 bytes, always one item:

```
AA AA CC 32 01 | A A T S C E 02 02 | 01 01 | 55 55
```

| Byte | Field | Values |
|---|---|---|
| A (twice) | action | see below |
| T | beats | how long the move runs, see below |
| S | speed | **0 fastest > 1 > 2 > 3 slowest** (*tested*; `FF` behaves like 2) |
| C | eye color | **1 blue, 2 cyan (default), 3 green, 4 yellow, 5 red, 6 purple, 7 white** (*tested*, switches right away) |
| E | eye mode | **0 steady, 4 slow blink** (*tested*); 1-3 look the same as 0 |
| `02 02` | unknown | the app always sends `02 02` |

Robert has no lights besides his eyes.

### Actions

| A | Meaning | Status |
|---|---|---|
| 1 | walk forward (toward his face) | *tested* |
| 2 | walk back | *tested* |
| 3 | turn right (his right) | *tested* |
| 4 | turn left (his left) | *tested* |
| 1-93 | dances ("combined"); 1-4 sit in this range too, with normal beats they were not tested | 10 *tested* |
| 100-110 | hand moves | |
| 200-235 | leg moves (the app skips 231) | |
| **77** | **stop / idle**; also how the app pushes new color, eye or speed settings | *tested* |
| 94-99, 111-199, 231, 236-254 | nothing: no movement, no sound | *tested*, all of them |

**Dances are not on the spot.** They turn him and move him around, so give him room and keep him away from table edges.

The official app labels the arrows from the viewer's side; the directions above are from Robert's own point of view.

### Beats (T)

- `2`, `4`, `8`, `12`: the app's choices. At speed 0 one beat is roughly **0.2 s** (see type 12), so 8 beats ≈ 1.6 s.
- **`FF` = repeat the move until the next command** (*tested* with walking and dance #10). This is how walking works: the app sends 1-4 with `FF` and sends stop on release.

## Stop and report (type 0x0C)

```
app → robot   AA AA CC 0C 00 55 55
robot → app   AA AA CC 0D 01 nn 55 55      (on 0xFFC2)
```

Stops the current move and replies with `nn` = beats danced since it started (*tested*: a 2 s loop of dance #10 reported 9, a 5 s loop reported 24, so about 5 beats per second at speed 0).

The app uses this in program mode: you start a move with `FF`, stop it with type 12, and it stores `nn` so playback runs the move for the same number of beats.

## Sensors

The robot sent nothing on `0xFFC2` when its head was tapped or its chest and ear buttons were pressed (*tested*). The only reply seen is the type 13 beats report.

## Examples

```
AA AA CC 32 01 01 01 FF 00 02 00 02 02 01 01 55 55   walk forward until stopped, cyan eyes
AA AA CC 32 01 4D 4D 08 00 05 00 02 02 01 01 55 55   stop, eyes red
AA AA CC 32 01 0A 0A 08 00 06 04 02 02 01 01 55 55   dance #10, 8 beats, purple blinking eyes
AA AA CC 03 01 0A 08 00 55 55                         dance #10, short form
AA AA CC 0C 00 55 55                                  stop + beats report
```

## Not explored

- The meaning of the two `02 02` bytes.
- Hand (100-110) and leg (200-235) moves one by one, and dances other than #10.
- The app also supports a second model, **"Our"**, on service `0x00FF` / characteristic `0xFF01`. Not tested, no such robot here.
