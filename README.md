# Fishao Auto-Angler

A small Python script that plays the **fishing mini-game** in the browser/desktop game *Fishao* for you.
When a fish bites, the game shows a colored bar (red – yellow – green – yellow – red) with a blue
marker that the fish keeps pulling to the left or right. The player has to hold the arrow keys so the
marker stays inside the green zone. This script does exactly that automatically, and closes the
"fish caught" window afterwards.

> **Note:** Automating a game can be against its terms of service. Use it at your own risk and only
> where it is allowed. The script only reads the screen and presses keys; it does not touch the
> game's files or network traffic.

## What it does

1. **Finds the bar by itself.** It continuously scans the screen (all monitors) for the mini-game
   dialog. No calibration is needed; the position of the dialog, the background (sand, grass, water,
   stone path) and the size of the green zone do not matter.
2. **Keeps the marker in the green zone.** While the bar is visible it measures the blue marker and
   the center of the green zone many times per second. It predicts where the marker is heading and
   holds the opposite arrow key: marker moves left → hold **right**, marker moves right → hold
   **left**. The key stays pressed (like a human would do) and is only switched when the direction
   changes.
3. **Stops when the bar is gone.** As soon as the bar disappears, every key is released and nothing
   is sent until the next bite, so your own movement in the game is not affected.
4. **Closes the "caught" window.** After a catch, the game shows a result window
   ("… GEFANGEN!"). The script recognizes it (independent of the fish and its name) and presses
   **Esc** to close it.

## Requirements

- Windows 10 or 11 (keys are sent with the Windows `SendInput` API)
- Python 3.9 or newer
- Python packages: `mss`, `numpy`, `pyautogui`

```
pip install mss numpy pyautogui
```

## Usage

```
cd C:\fishao
python fishao_angler.py
```

1. Start the script and leave the console window open.
2. Click into the game window and cast your rod yourself, so the game window has the keyboard focus
   when the fish bites.
3. The script reacts on its own to every bite. Stop it with **Ctrl+C**, or move the mouse into the
   top-left corner of the screen (pyautogui fail-safe).

If `python` is not found, try `py fishao_angler.py`.

## Settings

All settings are constants at the top of `fishao_angler.py`.

| Setting | Default | Meaning |
|---|---|---|
| `MONITOR` | `0` | `0` = search all monitors, `1` = only monitor 1, `2` = only monitor 2 … |
| `AUTO_CLOSE_CATCH` | `True` | Close the "fish caught" window with Esc |
| `INVERT_KEYS` | `False` | Set to `True` if the keys act in the wrong direction in your game |
| `KEY_MODE` | `"scancode"` | How keys are sent: `"scancode"`, `"vk"` or `"pyautogui"` (try the others if keys do not arrive) |
| `LEAD_S` | `0.30` | How far ahead (seconds) the controller predicts the marker. Higher = counters earlier, lower = calmer |
| `KP` | `4.0` | Strength of the reaction to the distance from the green center |
| `STRICT` | `True` | Strict check that a found bar really is the mini-game dialog |
| `LOG` | `True` | Write `fishao_log.csv` (marker, zone center, error, pressed key) |
| `DIAG` | `True` | Print diagnostics while searching and save `debug_beste_zeile.png` |

## Files the script writes

- `debug_screen.png` – screenshot taken at start, to check which area is captured
- `debug_beste_zeile.png` – best search frame so far (only with `DIAG = True`)
- `fishao_log.csv` – per-frame log of the last mini-game (set `LOG = False` to disable)

## Troubleshooting

- **Bar is not found:** check `debug_screen.png` to see that the game is visible on the captured
  monitor.
- **Only one key is pressed / marker runs away:** the game window probably does not have the focus.
  Click into the game first. If it still happens, set `INVERT_KEYS = True` or change `KEY_MODE`.
- **Marker swings too much:** raise `LEAD_S` (e.g. `0.4`). **Too nervous:** lower it (e.g. `0.2`).
- **Catch window does not close:** the script tries at most 4 times per window. Make sure the game
  has the focus. If the window looks different from the standard one (for example without the green
  "Show Fishbook" button), it will not be recognized.

## How detection works (short)

Everything is based on colors and shapes, not on reading text, so the game's language should not
matter (only the German version was tested):

- **Bar:** a horizontal strip with red edges and a smooth green peak in the middle, inside the
  beige dialog, with two blue marker triangles.
- **Marker:** the blue pixels inside the bar; their average x position is the marker position.
- **Green zone:** the strongest "green minus red" part of the bar; its center is the target.
- **Catch window:** the green "Show Fishbook" button, the beige panel around it and the yellow
  length ruler next to it.

## Limitations

- Tested only on the German client at a normal UI scale. A very different scale or a heavily changed
  color profile can break detection.
- It does not cast the rod, buy bait or handle other pop-ups; you start each fishing round yourself.
