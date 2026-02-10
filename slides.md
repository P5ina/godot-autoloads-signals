---
theme: default
title: "Godot Deep Dive: Autoloads & Signals"
info: |
  A practical guide to two essential Godot patterns:
  Autoloads (singletons) and Signals (observer pattern).
  AUCA Gamedev Club 2026.
transition: slide-left
mdc: true
lineNumbers: true
drawings:
  persist: false
---

# Godot Deep Dive
## Autoloads & Signals

<div class="pt-12">
  <span class="text-xl text-gray-400">
    AUCA Gamedev Club — 2026
  </span>
</div>

<div class="abs-br m-6 flex gap-2">
  <a href="https://docs.godotengine.org" target="_blank" class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    📚 Docs
  </a>
</div>

<!--
Welcome everyone! Today we'll cover two fundamental patterns that will level up your Godot projects.
-->

---
transition: fade
layout: center
---

# Part 1: Autoloads

<div class="text-2xl text-gray-400">
  Global singletons that persist across scenes
</div>

---

# 🤔 The Problem

Imagine you have a **score** variable in your game.

<v-clicks>

- Player collects a coin → score increases ✅
- Player moves to the **next level** (scene change) → score resets to 0 ❌
- You need score to **survive scene transitions**

</v-clicks>

<div v-click class="mt-8 p-4 bg-red-500/10 rounded-lg border border-red-500/20">
  <strong>Scene changes destroy all nodes.</strong> Your score variable dies with them.
</div>

<!--
This is the core problem. When you change scenes, everything gets freed. Let's see how autoloads solve this.
-->

---

# What is an Autoload?

A script (or scene) that Godot **loads automatically** when the game starts.

<v-clicks>

- 🌍 **Global** — accessible from anywhere via its name
- 🔄 **Persistent** — survives scene changes
- 1️⃣ **Singleton** — only one instance exists
- 📦 **Always loaded** — sits above the scene tree

</v-clicks>

<div v-click class="mt-6">

```
Root
├── GameManager    ← autoload (always here)
├── AudioManager   ← autoload (always here)
└── CurrentScene   ← gets replaced on scene change
    ├── Player
    └── Enemy
```

</div>

<!--
Think of autoloads as the "skeleton" of your game that never changes, while scenes are the "clothes" that get swapped.
-->

---

# Setting Up an Autoload

## Step 1: Create the script

```gdscript
# game_manager.gd
extends Node

var score: int = 0
var high_score: int = 0
var current_level: int = 1
```

<v-click>

## Step 2: Register it

**Project → Project Settings → Autoload**

| Path | Name | Enabled |
|------|------|---------|
| `res://game_manager.gd` | `GameManager` | ✅ |

</v-click>

<!--
Two simple steps. Create a script, register it. That's it. Godot handles the rest.
-->

---

# Using Autoloads

Now you can access `GameManager` from **any script** in your project:

```gdscript {1-3|5-7|9-12|all}
# In coin.gd
func _on_collected():
    GameManager.score += 10

# In enemy.gd
func _on_killed():
    GameManager.score += 50

# In ui.gd
func _process(_delta):
    $ScoreLabel.text = "Score: %d" % GameManager.score
    $LevelLabel.text = "Level: %d" % GameManager.current_level
```

<div v-click class="mt-4 p-4 bg-green-500/10 rounded-lg border border-green-500/20">
  💡 No need for <code>get_node()</code> or exports — just use the autoload name directly!
</div>

<!--
Click through to see different scripts all accessing the same GameManager. Notice how clean this is — no references to manage.
-->

---

# Practical Autoload Examples

<div class="grid grid-cols-2 gap-6">
<div v-click>

### 🎵 AudioManager

```gdscript
extends Node

var music_bus := "Music"

func play_sfx(sound: AudioStream):
    var player := AudioStreamPlayer.new()
    player.stream = sound
    add_child(player)
    player.play()
    player.finished.connect(
        player.queue_free
    )
```

</div>
<div v-click>

### 💾 SaveManager

```gdscript
extends Node

const PATH = "user://save.dat"

func save_game():
    var f = FileAccess.open(
        PATH, FileAccess.WRITE)
    f.store_var({
        "score": GameManager.score,
        "level": GameManager.current_level
    })

func load_game():
    if not FileAccess.file_exists(PATH):
        return
    var f = FileAccess.open(
        PATH, FileAccess.READ)
    var d = f.get_var()
    GameManager.score = d.score
    GameManager.current_level = d.level
```

</div>
</div>

<!--
Two super common autoloads. AudioManager for playing sounds from anywhere, SaveManager for persistence.
-->

---

# ⚠️ Autoload Best Practices

<v-clicks>

- **Keep them focused** — one responsibility per autoload
- **Don't put everything there** — only truly global state
- **Use signals** for communication (we'll cover this next!)
- **Avoid circular dependencies** — autoloads shouldn't depend on scene nodes

</v-clicks>

<div v-click class="mt-8 grid grid-cols-2 gap-4">
<div class="p-4 bg-green-500/10 rounded-lg border border-green-500/20">

### ✅ Good autoloads
- GameManager (score, lives)
- AudioManager
- SaveManager
- SceneTransition

</div>
<div class="p-4 bg-red-500/10 rounded-lg border border-red-500/20">

### ❌ Bad autoloads
- Player (scene-specific)
- EnemySpawner (scene-specific)
- UIController (changes per scene)
- "GodObject" (does everything)

</div>
</div>

<!--
The key rule: if it needs to exist across scenes, it's an autoload. If it belongs to a specific scene, it's not.
-->

---
transition: fade
layout: center
---

# Part 2: Signals

<div class="text-2xl text-gray-400">
  The Observer pattern, Godot-style
</div>

---

# 🤔 The Problem (Again)

Your **Player** takes damage. Who needs to know?

<v-clicks>

- 💊 **HealthBar** — update the display
- 🎵 **AudioManager** — play hurt sound
- 📸 **Camera** — screen shake
- 🎮 **GameManager** — check if player died
- 😈 **Enemy** — celebrate?

</v-clicks>

<div v-click class="mt-6 p-4 bg-red-500/10 rounded-lg border border-red-500/20">

```gdscript
# ❌ Without signals: Player knows about EVERYONE
func take_damage(amount):
    health -= amount
    $"../HealthBar".update(health)          # tight coupling!
    $"../AudioManager".play("hurt")         # what if path changes?
    $"../Camera".shake(0.5)                 # what if Camera doesn't exist?
    GameManager.check_death(health)         # getting messy...
```

</div>

<!--
This is spaghetti code. The Player shouldn't need to know about every system that cares about damage. Enter: signals.
-->

---

# What is a Signal?

A way for nodes to **emit events** without knowing who's listening.

<div class="mt-4">

```
Player says: "Hey, I took damage!"

              ╭──→ HealthBar: "Got it, updating display"
              │
Player ──emit─┼──→ AudioManager: "Playing hurt sound"
              │
              ├──→ Camera: "Shaking screen"
              │
              ╰──→ GameManager: "Checking if dead"
```

</div>

<v-clicks>

- **Emitter** doesn't know (or care) who's listening
- **Listeners** connect to signals they're interested in
- Adding/removing listeners doesn't change the emitter
- **Decoupled** — nodes are independent

</v-clicks>

<!--
The Player just broadcasts "damage happened". Anyone interested can listen. The Player doesn't need to change when we add new listeners.
-->

---

# Defining Signals

```gdscript {1-2|4-5|7-8|all}
# Simple signal — no data
signal died

# Signal with parameters
signal health_changed(new_health: int, max_health: int)

# Signal with complex data
signal item_collected(item_name: String, item_value: int)
```

<v-click>

## Emitting Signals

```gdscript {1-8|all}
# player.gd
var health: int = 100
var max_health: int = 100

func take_damage(amount: int):
    health -= amount
    health_changed.emit(health, max_health)
    if health <= 0:
        died.emit()
```

</v-click>

<!--
Defining signals is just one line. Emitting is calling .emit() with the right arguments. Simple.
-->

---

# Connecting Signals

### Method 1: In code (recommended)

```gdscript {1-5|7-11|all}
# health_bar.gd
func _ready():
    # Find the player and connect to its signal
    var player = get_node("../Player")
    player.health_changed.connect(_on_health_changed)

func _on_health_changed(new_health: int, max_health: int):
    # Update the health bar
    value = float(new_health) / float(max_health) * 100
    if new_health < max_health * 0.25:
        modulate = Color.RED  # flash red when low
```

<v-click>

### Method 2: In the Editor

1. Select the **Player** node
2. Go to **Node** tab → **Signals**
3. Double-click `health_changed`
4. Pick the target node & method

</v-click>

<!--
Two ways to connect. Code is more flexible, editor is more visual. I recommend code for anything dynamic.
-->

---

# Built-in Signals

Godot nodes come with **tons** of useful signals:

<div class="grid grid-cols-2 gap-4 mt-4">
<div>

### Common Node Signals

```gdscript
# Node
ready
tree_entered
tree_exiting

# Timer
timeout

# Area2D
body_entered(body)
body_exited(body)
area_entered(area)
```

</div>
<div>

### UI Signals

```gdscript
# Button
pressed

# LineEdit
text_changed(new_text)
text_submitted(text)

# AnimationPlayer
animation_finished(anim_name)

# HTTPRequest
request_completed(result, code,
    headers, body)
```

</div>
</div>

<div v-click class="mt-4 p-4 bg-blue-500/10 rounded-lg border border-blue-500/20">
  💡 <strong>Tip:</strong> Always check the <strong>Node → Signals</strong> tab in the editor to see available signals!
</div>

<!--
You don't always need custom signals. Godot's built-in ones cover most use cases.
-->

---

# Signals + Autoloads = 🔥

The **real power** comes from combining both patterns:

```gdscript {1-7|9-15|17-23|all}
# game_manager.gd (autoload)
signal game_over
signal score_changed(new_score: int)
signal level_completed(level: int)

var score: int = 0:
    set(value):
        score = value
        score_changed.emit(score)

# player.gd — emit through autoload
func die():
    GameManager.game_over.emit()

# ANY script can listen — no node references needed!
# ui.gd
func _ready():
    GameManager.score_changed.connect(_on_score_changed)
    GameManager.game_over.connect(_on_game_over)

func _on_score_changed(new_score):
    $ScoreLabel.text = str(new_score)

func _on_game_over():
    $GameOverScreen.show()
```

<!--
This is the pattern you'll use 90% of the time. Global signals through autoloads. Any node can emit, any node can listen.
-->

---

# Real-World Example: Coin Pickup

Let's put it all together:

<div class="grid grid-cols-2 gap-4">
<div>

```gdscript
# game_manager.gd (autoload)
signal coin_collected(total: int)

var coins: int = 0

func add_coin():
    coins += 1
    coin_collected.emit(coins)
```

```gdscript
# coin.gd
func _on_body_entered(body):
    if body.is_in_group("player"):
        GameManager.add_coin()
        queue_free()
```

</div>
<div>

```gdscript
# coin_counter_ui.gd
func _ready():
    GameManager.coin_collected.connect(
        _update_display
    )
    _update_display(GameManager.coins)

func _update_display(total: int):
    text = "🪙 %d" % total
```

```gdscript
# achievement_manager.gd
func _ready():
    GameManager.coin_collected.connect(
        _check_achievements
    )

func _check_achievements(total: int):
    if total >= 100:
        unlock("coin_master")
```

</div>
</div>

<div v-click class="mt-4 p-3 bg-yellow-500/10 rounded-lg border border-yellow-500/20 text-center">
  🧩 Coin doesn't know about UI. UI doesn't know about achievements. Everything just works.
</div>

<!--
Look how clean this is. Each piece does one thing. Adding new listeners doesn't require changing anything else.
-->

---

# ⚠️ Signal Best Practices

<v-clicks>

- **Name signals as past tense events**: `health_changed`, `died`, `item_collected`
- **Keep parameters minimal** — pass only what listeners need
- **Disconnect when needed**: `signal.disconnect(callable)` to prevent leaks
- **Use `Callable`** for lambdas: `signal.connect(func(): print("hi"))`
- **One-shot connections**: `signal.connect(callback, CONNECT_ONE_SHOT)`
- **Don't chain signals** excessively — it becomes hard to debug

</v-clicks>

<div v-click class="mt-6 p-4 bg-blue-500/10 rounded-lg border border-blue-500/20">

### 🐛 Debugging tip

```gdscript
# Print when signal fires — great for debugging
my_signal.connect(func(args): print("Signal fired: ", args))
```

</div>

---
layout: center
---

# Quick Comparison

| Feature | Direct Call | Signal |
|---------|------------|--------|
| Coupling | Tight 🔗 | Loose 🔓 |
| Emitter knows listener? | Yes | No |
| Multiple listeners? | Manual | Automatic |
| Easy to add new listeners? | Requires changes | Just connect |
| Debugging | Easier to trace | Harder to trace |
| Performance | Slightly faster | Slightly slower |

<div v-click class="mt-6 text-center text-xl">
  <strong>Rule of thumb:</strong> Use signals when multiple systems need to react to the same event
</div>

---
layout: center
---

# Recap 🎯

<div class="grid grid-cols-2 gap-8 mt-8">
<div v-click>

## Autoloads

- Global singletons
- Persist across scenes
- Access by name: `GameManager.score`
- Good for: game state, audio, saves, transitions
- Set up: Project Settings → Autoload

</div>
<div v-click>

## Signals

- Event-driven communication
- Decoupled architecture
- Define: `signal my_event`
- Emit: `my_event.emit()`
- Connect: `node.my_event.connect(func)`

</div>
</div>

<div v-click class="mt-8 text-center text-xl">
  <strong>Combined:</strong> Autoload signals = global event bus 🚌
</div>

---
layout: center
---

# Let's Code! 🎮

<div class="text-xl text-gray-400 mt-4">
  Time for a hands-on exercise
</div>

<v-clicks class="mt-8 text-lg">

1. Create a `GameManager` autoload with score + lives
2. Add signals: `score_changed`, `life_lost`, `game_over`
3. Build a simple scene that uses both patterns
4. Bonus: Add an `AudioManager` autoload

</v-clicks>

<!--
Let's spend the rest of the time coding together. Open Godot and follow along!
-->

---
layout: center
---

# Questions? 🤔

<div class="text-xl text-gray-400 mt-4">

**Resources:**

- [Godot Docs: Singletons (Autoload)](https://docs.godotengine.org/en/stable/tutorials/scripting/singletons_autoload.html)
- [Godot Docs: Signals](https://docs.godotengine.org/en/stable/getting_started/step_by_step/signals.html)
- [GDQuest: Signals Guide](https://www.gdquest.com/tutorial/godot/gdscript/signals/)

</div>

<div class="abs-br m-6 text-gray-500">
  Made with Slidev ⚡
</div>
