# Bot Battles

**Toasters with nail guns.** Program your bot in Python or blocks, spend your points wisely, and trick your opponent into the crusher. Runs entirely in the browser — no install, no server.

![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)

---

## What is this?

Bot Battles is a programming game. You write a program for a battle robot, equip it
with weapons and armor from a points budget, and then watch it fight on its own — you
don't drive it, you *teach* it. If it loses, you change the code and try again.

It was built to teach kids to program. You can build a bot by dragging blocks, by
typing real Python, or by mixing both, and the game shows you the Python as you drag so
the two stay connected in your head.

Everything runs in your browser. There is no server, no account, and nothing to install.

## Play it

**[▶ Play Bot Battles](https://iamalanturing.github.io/botbattles/)**

Or download `index.html` and open it — it's a single self-contained file.

**Tip for Chromebooks:** open the page in Chrome, then use the three-dot menu →
*Cast, save and share* → **Create shortcut**. You get a Bot Battles icon in your
launcher that opens without an address bar.

You need an internet connection the first time, because Python and the block editor
load from a CDN. After that your browser caches them.

---

## How a battle works

Two bots start at opposite ends of an arena. Every bot runs its own program, 30 times
a second, until one is destroyed or the 90-second timer runs out.

- **Your program can't see everything.** Radar is attached to your gun and only covers
  a cone in front of it. If you never swing your gun around, you're blind.
- **Speed hurts.** Slamming into a wall does damage based on how fast you were going.
  So does ramming. That's the whole point — a clever bot can bait a fast one into a
  wall and never fire a shot.
- **The crusher** sits in the middle of the arena on a timer. It flashes a warning,
  then slams. In the last third of the match it speeds up, so nobody can stall.
- **If time runs out,** whoever dealt the most damage wins. A bot that never lands a
  hit can't win — so an all-armor turtle bot is annoying, but it can't take the match.

Speed controls run from 0.15× slow motion up to 20× blitz. **Pause** then **Step**
advances exactly one tick at a time, which is the best way to work out why your bot did
something strange.

---

## Writing a bot

### With blocks

Drag blocks from the categories on the left. The Python appears on the right as you
build. Click **⛶ Expand** for a full-screen editor — much easier for dragging.

### With Python

Switch the toggle to **Python** and type. Here's a complete bot:

```python
BOT_NAME = "Sir Toasts-a-Lot"
LOADOUT = {
    "chassis": "toaster",
    "slots": ["bolt_blaster", "armor_plate", "oil_slick"],
}

def on_start(bot):
    bot.say("Toast time!")

def on_tick(bot):
    targets = bot.scan()
    if targets:
        enemy = targets[0]
        bot.turn_gun_toward(enemy)
        bot.fire()
        if enemy["distance"] < 110:
            bot.drop_oil_slick()
            bot.drive(-3)
        else:
            bot.drive(2)
    else:
        bot.aim(9)      # sweep the gun to find someone
        bot.drive(2)
        bot.turn(2)

def on_hit(bot, damage, source):
    bot.say("Ow!")
    bot.turn(90)

def on_wall(bot):
    bot.turn(150)
```

### Switching between them

You can switch back and forth freely. Going from Python to blocks, anything the blocks
can express becomes real blocks, and anything they can't — a `while` loop, a helper
function, some maths the blocks don't have — becomes a purple **Python Block** holding
your code exactly as you wrote it. Nothing gets lost in either direction.

A Python Block with a mistake in it turns red straight away and tells you what's wrong,
and the game won't start a battle until you fix it.

---

## Bot API

### Handlers — you write these

| Handler | When it runs |
| --- | --- |
| `on_start(bot)` | Once, at the start of the match |
| `on_tick(bot)` | 30 times a second |
| `on_hit(bot, damage, source)` | When your bot takes damage |
| `on_wall(bot)` | When your bot hits a wall |

`on_hit` fires **once per tick with the total damage**, not once per bullet, and
`bot.health` is already updated when it runs. `source` is the attacker's name, or
`None` for walls and the crusher.

### Actions

| Call | What it does |
| --- | --- |
| `bot.drive(speed)` | Set speed. Negative reverses. Capped by your chassis. |
| `bot.turn(degrees)` | Turn your body. Capped by your chassis turn rate. |
| `bot.turn_toward(bearing)` | Turn toward a compass direction. |
| `bot.stop()` | Speed to zero. |
| `bot.aim(degrees)` | Swing the gun (and your radar with it). |
| `bot.turn_gun_toward(contact)` | Point the gun at something `scan()` found. |
| `bot.fire()` | Fire every weapon that's off cooldown. |
| `bot.fire_weapon("nail_gun")` | Fire one specific weapon. |
| `bot.drop_oil_slick()` | Drop a slick behind you. |
| `bot.say(text)` | Speech bubble over your bot. |

### Sensors

| Property | Gives you |
| --- | --- |
| `bot.scan()` | List of contacts in your radar cone, nearest first |
| `bot.nearest_hazard()` | Dict about the crusher, or `None` |
| `bot.health` | Your current health |
| `bot.speed` | Your current speed |
| `bot.heading` / `bot.gun_heading` | Which way you and your gun face |
| `bot.position` | Your `(x, y)` |
| `bot.wall_ahead` | Distance to the wall in front, or `None` |
| `bot.slicks_left` | Oil slicks remaining |
| `bot.on_slick` | `True` if you're sliding |
| `bot.time_left` | Seconds left in the match |
| `bot.weapons` | List of what you're carrying |
| `bot.weapon_ready(name)` | `True` if that weapon is off cooldown |

A **contact** from `scan()` is a dict with `distance`, `bearing`, `chassis` and `team`.
You cannot see another bot's health, ammo or code — only what you could observe from
across the arena.

A **hazard** is a dict with `distance`, `bearing`, `dangerous` (about to slam) and
`inside` (you're standing on it).

---

## Parts

You get a points budget — 50, 100 or 200 — and spend it however you like. All weapons
and no armor is legal. All armor and no weapons is legal. The only hard limit is how
many slots your chassis has.

### Chassis

| Chassis | Cost | Health | Speed | Slots |
| --- | --- | --- | --- | --- |
| Roomba | 15 | 75 | Very fast | 2 |
| Toaster | 25 | 100 | Medium | 3 |
| Fridge | 45 | 165 | Slow | 5 |

### Weapons

| Weapon | Cost | Range | Damage | Rate |
| --- | --- | --- | --- | --- |
| Nail Gun | 25 | Long | 14 | Slow — you must lead the target |
| Bolt Blaster | 20 | Medium | 9 | Medium |
| Crumb Cannon | 15 | Short | 5 × 3 pellets | Fast, wide spray |
| Whisk Blades | 30 | Touch | 1.5 per tick | Continuous |

### Defenses and gadgets

| Part | Cost | Effect |
| --- | --- | --- |
| Armor Plate | 15 | −2 damage from every hit (costs more each time you stack it) |
| Bumper Guard | 15 | −45% ramming and wall damage (does nothing against bullets) |
| Oil Slick | 10 | Three slicks; whoever drives over one loses their steering |

A big gun isn't automatically the right answer. A Roomba with Whisk Blades that simply
charges is cheap, easy to program, and beats plenty of cleverer bots.

---

## Saving and sharing bots

**↓ Save** writes a `.bot.py` file named after your bot. It's ordinary Python — you can
open it in any text editor, email it, or read it. If you built with blocks, the block
layout rides along in a comment at the bottom, so it reopens exactly as you left it.

**↑ Load** reads one back. A file with no block layout gets parsed into blocks anyway.

A loaded bot never runs until you press Launch. That's deliberate: a `.bot.py` from
someone else is code, and you should get to look at it first.

---

## Safety

Bot programs run in a sandboxed Web Worker, and they can't reach the page, your files
or the network. Python never receives a JavaScript object — commands cross the boundary
as plain validated data — which closes the usual escape route out of an in-browser
Python sandbox.

A program stuck in an infinite loop hits an instruction budget, skips that tick with a
visible warning, and the page keeps running. You can try it: there's a **⚠ runaway**
button that does exactly this on purpose, and a **🔓 escape** button that attempts a
real sandbox escape so you can watch it fail.

No accounts, no tracking, no data leaves your browser.

---

## Built with

- [Pyodide](https://pyodide.org/) — CPython compiled to WebAssembly (MPL-2.0)
- [Blockly](https://developers.google.com/blockly) — the block editor (Apache-2.0)

Both load from a CDN; neither is redistributed in this repository.

## License

Bot Battles is free software, licensed under the **GNU General Public License v3.0**.

You may use, study, share and modify it. If you distribute a modified version, it must
also be released under the GPL-3.0 with its source available. See [LICENSE](LICENSE).
