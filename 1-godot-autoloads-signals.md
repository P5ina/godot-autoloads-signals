---
theme: default
title: "Лекция 1: Погружение в Godot — Autoloads и Signals"
info: |
  Практическое руководство по двум ключевым паттернам Godot:
  Autoloads (синглтоны) и Signals (паттерн наблюдатель).
  AUCA Gamedev Club 2026.
transition: slide-left
mdc: true
lineNumbers: true
drawings:
  persist: false
---

# Лекция 1: Погружение в Godot
## Autoloads и Signals

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
Всем привет! Сегодня разберём два фундаментальных паттерна, которые прокачают ваши проекты на Godot.
-->

---

# Почему Godot?

<div class="grid grid-cols-2 gap-6 mt-2">
<div>

### Unity

- 🏢 Корпоративный, закрытый код
- 💰 Платный на определённом уровне дохода
- ⚙️ Компонентная архитектура (GameObject + Component)
- 🔤 C# / (бывший) UnityScript
- 📦 Тяжёлый — гигабайты на установку
- 🎮 Сильный в 3D, хорош в 2D

</div>
<div>

### Godot

- 🌍 Open source, MIT лицензия
- 🆓 Полностью бесплатный навсегда
- 🌳 Нодовая архитектура (Node + Scene)
- 🔤 GDScript / C# / GDExtension (C++)
- 🪶 Лёгкий — ~50 МБ, один бинарник
- 🎮 Отличный в 2D, растущий в 3D

</div>
</div>

<!--
Краткое сравнение для контекста. У обоих движков свои сильные стороны, но Godot выделяется доступностью и простотой.
-->

---

# Архитектура: Unity vs Godot

<div class="grid grid-cols-2 gap-6 mt-4">
<div>

### Unity: GameObject + Component

```
Player (GameObject)
├── Transform
├── SpriteRenderer
├── Rigidbody2D
├── PlayerController.cs
└── HealthSystem.cs
```

Нода сама по себе **пустая** — поведение через компоненты

</div>
<div>

### Godot: Node + Scene

```
Player (CharacterBody2D)
├── Sprite2D
├── CollisionShape2D
├── AnimationPlayer
└── HealthBar (сцена-в-сцене)
```

Каждая нода уже **имеет назначение** — композиция через вложенность

</div>
</div>

<div v-click class="mt-4 p-3 bg-blue-500/10 rounded-lg border border-blue-500/20 text-center">
  💡 В Godot <strong>всё является нодой</strong> — и это ключ к пониманию Autoloads и Signals
</div>

<!--
Главное отличие: Unity строит поведение из компонентов, Godot — из дерева нод. Оба подхода рабочие, но нодовая система Godot делает autoloads и signals очень естественными.
-->

---
transition: fade
layout: center
---

# Часть 1: Autoloads

<div class="text-2xl text-gray-400">
  Глобальные синглтоны, которые живут между сценами
</div>

---

# 🤔 Проблема

Представьте: у вас есть переменная **score** в игре.

<v-clicks>

- Игрок собирает монетку → очки растут ✅
- Игрок переходит на **следующий уровень** (смена сцены) → очки сбрасываются в 0 ❌
- Нужно чтобы очки **пережили смену сцены**

</v-clicks>

<div v-click class="mt-8 p-4 bg-red-500/10 rounded-lg border border-red-500/20">
  <strong>Смена сцены уничтожает все ноды.</strong> Ваша переменная умирает вместе с ними.
</div>

<!--
Это главная проблема. При смене сцены всё освобождается. Давайте посмотрим, как autoloads это решают.
-->

---

# Что такое Autoload?

Скрипт (или сцена), который Godot **загружает автоматически** при старте игры.

<v-clicks>

- 🌍 **Глобальный** — доступен из любого места по имени
- 🔄 **Персистентный** — переживает смену сцен
- 1️⃣ **Синглтон** — существует только один экземпляр
- 📦 **Всегда загружен** — находится над деревом сцен

</v-clicks>

<div v-click class="mt-6">

```
Root
├── GameManager    ← autoload (всегда здесь)
├── AudioManager   ← autoload (всегда здесь)
└── CurrentScene   ← заменяется при смене сцены
    ├── Player
    └── Enemy
```

</div>

<!--
Думайте об autoloads как о «скелете» игры, который никогда не меняется, а сцены — это «одежда», которую можно менять.
-->

---

# Скрипт vs Сцена Autoload

<div class="grid grid-cols-2 gap-6 mt-4">
<div v-click class="p-4 bg-blue-500/10 rounded-lg border border-blue-500/20">

### 📝 Script (.gd)

- Только код, без нод
- Легче, проще
- Для: состояния, данных, утилит
- `GameManager`, `SaveManager`

</div>
<div v-click class="p-4 bg-purple-500/10 rounded-lg border border-purple-500/20">

### 🎬 Scene (.tscn)

- Полное дерево нод + скрипт
- Может содержать UI, Timer, AudioStreamPlayer...
- Для: всего, что требует дочерние ноды
- `SceneTransition` (с ColorRect), `AudioManager` (с AudioStreamPlayer)

</div>
</div>

<div v-click class="mt-6 p-3 bg-yellow-500/10 rounded-lg border border-yellow-500/20 text-center">
  💡 Начинай со скрипта — переделать в сцену можно в любой момент
</div>

<!--
Правило простое: если автолоаду не нужны дочерние ноды — делай скриптом. Понадобились ноды — оборачивай в сцену.
-->

---

# Настройка Autoload

## Шаг 1: Создаём скрипт

```gdscript
# game_manager.gd
extends Node

var score: int = 0
var high_score: int = 0
var current_level: int = 1
```

<v-click>

## Шаг 2: Регистрируем

**Project → Project Settings → Autoload**

| Path | Name | Enabled |
|------|------|---------|
| `res://game_manager.gd` | `GameManager` | ✅ |

</v-click>

<!--
Два простых шага. Создаём скрипт, регистрируем. Всё. Godot сделает остальное.
-->

---

# Использование Autoloads

Теперь `GameManager` доступен из **любого скрипта** в проекте:

```gdscript {1-5|5-7|9-12|all}
# В coin.gd
extends Area2D

func _on_collected():
    GameManager.score += 10

# В enemy.gd
func _on_killed():
    GameManager.score += 50

# В ui.gd
func _process(_delta):
    $ScoreLabel.text = "Score: %d" % GameManager.score
    $LevelLabel.text = "Level: %d" % GameManager.current_level
```

<div v-click class="mt-4 p-4 bg-green-500/10 rounded-lg border border-green-500/20">
  💡 Не нужен <code>get_node()</code> или export — просто используйте имя autoload напрямую!
</div>

<!--
Пролистайте, чтобы увидеть, как разные скрипты обращаются к одному GameManager. Обратите внимание, как чисто это выглядит — никаких ссылок.
-->

---

# Примеры Autoloads на практике

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
    var f = FileAccess.open(PATH, FileAccess.WRITE)
    f.store_var({"score": GameManager.score,
        "level": GameManager.current_level})

func load_game():
    if not FileAccess.file_exists(PATH):
        return
    var f = FileAccess.open(PATH, FileAccess.READ)
    var d = f.get_var()
    GameManager.score = d.score
    GameManager.current_level = d.level
```

</div>
</div>

<style>
.slidev-code { font-size: 0.82em !important; line-height: 1.35 !important; }
</style>

<!--
Два самых частых autoload. AudioManager для воспроизведения звуков откуда угодно, SaveManager для сохранений.
-->

---

# ⚠️ Лучшие практики Autoloads

<v-clicks>

- **Один autoload — одна задача** — не мешайте ответственности
- **Не тащите всё в autoload** — только действительно глобальное состояние
- **Используйте сигналы** для общения (об этом дальше!)
- **Избегайте циклических зависимостей** — autoloads не должны зависеть от нод сцены

</v-clicks>

<div v-click class="mt-8 grid grid-cols-2 gap-4">
<div class="p-4 bg-green-500/10 rounded-lg border border-green-500/20">

### ✅ Хорошие autoloads
- GameManager (очки, жизни)
- AudioManager
- SaveManager
- SceneTransition

</div>
<div class="p-4 bg-red-500/10 rounded-lg border border-red-500/20">

### ❌ Плохие autoloads
- Player (привязан к сцене)
- EnemySpawner (привязан к сцене)
- UIController (меняется от сцены к сцене)
- "GodObject" (делает всё)

</div>
</div>

<!--
Ключевое правило: если что-то должно существовать между сценами — это autoload. Если принадлежит конкретной сцене — нет.
-->

---
transition: fade
layout: center
---

# Часть 2: Signals

<div class="text-2xl text-gray-400">
  Паттерн наблюдатель в стиле Godot
</div>

---

# 🤔 Проблема (снова)

Ваш **Player** получает урон. Кому нужно знать?

<v-clicks>

- 💊 **HealthBar** — обновить отображение
- 🎵 **AudioManager** — проиграть звук удара
- 📸 **Camera** — тряска экрана
- 🎮 **GameManager** — проверить, жив ли игрок
- 😈 **Enemy** — порадоваться?

</v-clicks>

<div v-click class="mt-4 p-3 bg-red-500/10 rounded-lg border border-red-500/20">

```gdscript
# ❌ Без сигналов: Player знает обо ВСЕХ
func take_damage(amount):
    health -= amount
    $"../HealthBar".update(health)       # жёсткая связь!
    $"../AudioManager".play("hurt")      # а если путь изменится?
    $"../Camera".shake(0.5)              # а если Camera не существует?
    GameManager.check_death(health)      # каша...
```

</div>

<!--
Это спагетти-код. Player не должен знать о каждой системе, которой важен урон. На помощь: сигналы.
-->

---

# Что такое Signal?

Способ для нод **отправлять события**, не зная, кто слушает.

<div class="mt-4">

```
Player говорит: "Эй, мне нанесли урон!"

              ╭──→ HealthBar: "Понял, обновляю отображение"
              │
Player ──emit─┼──→ AudioManager: "Проигрываю звук удара"
              │
              ├──→ Camera: "Трясу экран"
              │
              ╰──→ GameManager: "Проверяю, жив ли"
```

</div>

<v-clicks>

- **Отправитель** не знает (и ему всё равно), кто слушает
- **Слушатели** подключаются к интересующим сигналам
- Добавление/удаление слушателей не меняет отправителя
- **Развязка** — ноды независимы друг от друга

</v-clicks>

<!--
Player просто транслирует "получен урон". Любой заинтересованный может слушать. Player не нужно менять при добавлении новых слушателей.
-->

---

# Объявление сигналов

```gdscript {1-2|4-5|7-8|all}
signal died                        # без данных

signal health_changed(new_health: int, max_health: int)

signal item_collected(item_name: String, item_value: int)
```

<v-click>

## Отправка сигналов

```gdscript
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
Объявление сигнала — одна строка. Отправка — вызов .emit() с нужными аргументами. Просто.
-->

---

# Подключение сигналов

### Способ 1: В коде

```gdscript {1-4|6-9|all}
# health_bar.gd
func _ready():
    var player = get_node("../Player")
    player.health_changed.connect(_on_health_changed)

func _on_health_changed(new_health: int, max_health: int):
    value = float(new_health) / float(max_health) * 100
    if new_health < max_health * 0.25:
        modulate = Color.RED  # мигаем красным при низком HP
```

<v-click>

### Способ 2: В редакторе (я рекомендую)

1. Выбираем ноду **Player** → вкладка **Node** → **Signals**
2. Двойной клик по `health_changed` → выбираем целевую ноду и метод

</v-click>

<!--
Два способа подключения. Код более гибкий, редактор более наглядный. Рекомендую код для всего динамического.
-->

---

# `await` + Signals

Godot 4 позволяет **приостановить функцию** до срабатывания сигнала:

```gdscript
# Ждём таймер
await get_tree().create_timer(2.0).timeout
print("Прошло 2 секунды!")

# Ждём завершения анимации
$AnimationPlayer.play("attack")
await $AnimationPlayer.animation_finished

# Ждём кастомный сигнал
var result = await GameManager.level_completed
print("Уровень пройден: ", result)
```

<v-click>

<div class="mt-4 p-3 bg-green-500/10 rounded-lg border border-green-500/20">
  💡 <code>await</code> превращает асинхронный код в читаемые последовательные шаги — без колбэков!
</div>

</v-click>

<!--
await — одна из лучших фич GDScript. Делает асинхронный код похожим на синхронный. Супер полезно для катсцен, диалогов и последовательной игровой логики.
-->

---

# Встроенные сигналы

В нодах Godot **куча** полезных сигналов из коробки:

<div class="grid grid-cols-2 gap-4 mt-4">
<div>

### Сигналы нод

```gdscript
# Node
ready / tree_entered / tree_exiting
# Timer
timeout
# Area2D
body_entered(body)
body_exited(body)
area_entered(area)
```

</div>
<div>

### Сигналы UI

```gdscript
# Button
pressed
# LineEdit
text_changed(new_text)
text_submitted(text)
# AnimationPlayer
animation_finished(anim_name)
# HTTPRequest
request_completed(result, code, headers, body)
```

</div>
</div>

<div v-click class="mt-4 p-4 bg-blue-500/10 rounded-lg border border-blue-500/20">
  💡 <strong>Совет:</strong> Всегда проверяйте вкладку <strong>Node → Signals</strong> в редакторе, чтобы увидеть доступные сигналы!
</div>

<!--
Не всегда нужны кастомные сигналы. Встроенные покрывают большинство случаев.
-->

---

# Signals + Autoloads = 🔥

**Настоящая сила**: autoload с сигналами = глобальная шина событий

```gdscript
# game_manager.gd (autoload)
signal game_over
signal score_changed(new_score: int)
signal level_completed(level: int)

var score: int = 0:
    set(value):
        score = value
        score_changed.emit(score)
```

<v-click>

Теперь **любой скрипт** может отправлять и слушать — без ссылок на ноды!

```gdscript
# player.gd — отправляем через autoload
func die():
    GameManager.game_over.emit()
```

</v-click>

<!--
Autoload объявляет сигналы и состояние. Сеттер автоматически отправляет сигнал при изменении. Любой скрипт может взаимодействовать через autoload.
-->

---

# Подписка на сигналы Autoload

```gdscript
# ui.gd
func _ready():
    GameManager.score_changed.connect(_on_score_changed)
    GameManager.game_over.connect(_on_game_over)

func _on_score_changed(new_score):
    $ScoreLabel.text = str(new_score)

func _on_game_over():
    $GameOverScreen.show()
```

<div v-click class="mt-6 p-4 bg-green-500/10 rounded-lg border border-green-500/20">
  💡 Этот паттерн вы будете использовать в 90% случаев. Любая нода может отправлять, любая может слушать — всё через autoload.
</div>

<!--
UI просто подключается к сигналам autoload в _ready. Чисто, без жёсткой связи.
-->

---

# Пример: SceneManager Autoload

Плавные переходы между сценами с использованием обоих паттернов:

```gdscript
# scene_manager.gd (autoload)
signal scene_changed(scene_name: String)

func change_scene(path: String):
    var tween = create_tween()
    tween.tween_method(_fade, 0.0, 1.0, 0.3)   # затемнение
    await tween.finished
    get_tree().change_scene_to_file(path)
    tween = create_tween()
    tween.tween_method(_fade, 1.0, 0.0, 0.3)   # проявление
    scene_changed.emit(path)
```

<v-click>

```gdscript
# В любом месте игры:
SceneManager.change_scene("res://levels/level_2.tscn")
```

</v-click>

<style>
.slidev-code { font-size: 0.85em !important; line-height: 1.35 !important; }
</style>

<!--
SceneManager — вероятно, самый частый autoload после GameManager. Инкапсулирует логику переходов, чтобы каждая смена сцены выглядела красиво.
-->

---

# Полный пример: сбор монет

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
    GameManager.coin_collected.connect(_update_display)
    _update_display(GameManager.coins)

func _update_display(total: int):
    text = "🪙 %d" % total
```

```gdscript
# achievement_manager.gd
func _ready():
    GameManager.coin_collected.connect(_check_achievements)

func _check_achievements(total: int):
    if total >= 100:
        unlock("coin_master")
```

</div>
</div>

<div v-click class="mt-2 p-3 bg-yellow-500/10 rounded-lg border border-yellow-500/20 text-center">
  🧩 Монетка не знает про UI. UI не знает про достижения. Всё просто работает.
</div>

<style>
.slidev-code { font-size: 0.8em !important; line-height: 1.3 !important; }
</style>

<!--
Посмотрите, как чисто. Каждая часть делает одно дело. Добавление новых слушателей не требует изменений в остальном коде.
-->

---

# ⚠️ Лучшие практики Signals

<v-clicks>

- **Называйте сигналы как прошедшие события**: `health_changed`, `died`, `item_collected`
- **Минимум параметров** — передавайте только то, что нужно слушателям
- **Отключайте при необходимости**: `signal.disconnect(callable)` чтобы избежать утечек
- **Используйте `Callable`** для лямбд: `signal.connect(func(): print("hi"))`
- **Одноразовые подключения**: `signal.connect(callback, CONNECT_ONE_SHOT)`
- **Не стройте цепочки сигналов** — это сложно отлаживать

</v-clicks>

<div v-click class="mt-6 p-4 bg-blue-500/10 rounded-lg border border-blue-500/20">

### 🐛 Совет по отладке

```gdscript
# Печатаем при срабатывании сигнала — отлично для дебага
my_signal.connect(func(args): print("Сигнал сработал: ", args))
```

</div>

---

# Частые ошибки

<div class="grid grid-cols-2 gap-4">
<div v-click class="p-4 bg-red-500/10 rounded-lg border border-red-500/20">

### Спагетти из сигналов

A → emit → B → emit → C → emit → A

Циклические цепочки сигналов **невозможно отладить**. Держите поток однонаправленным.

</div>
<div v-click class="p-4 bg-red-500/10 rounded-lg border border-red-500/20">

### Забыли отключиться

```gdscript
# Нода удалена, а сигнал
# всё ещё ссылается → ошибка!
func _exit_tree():
    GameManager.score_changed.disconnect(
        _on_score_changed)
```

</div>
<div v-click class="p-4 bg-red-500/10 rounded-lg border border-red-500/20">

### God Autoload

Один autoload на 2000 строк, который делает всё. **Разделяйте** по ответственности!

</div>
<div v-click class="p-4 bg-red-500/10 rounded-lg border border-red-500/20">

### Сигнал для связи 1-к-1

Если слушает только **одна** нода — просто вызовите метод напрямую. Сигналы хороши при **нескольких** слушателях.

</div>
</div>

<style>
.slidev-code { font-size: 0.8em !important; line-height: 1.3 !important; }
</style>

---
layout: center
---

# Сравнение

| Свойство | Прямой вызов | Signal |
|----------|-------------|--------|
| Связанность | Жёсткая 🔗 | Слабая 🔓 |
| Отправитель знает слушателя? | Да | Нет |
| Несколько слушателей? | Вручную | Автоматически |
| Легко добавить слушателя? | Нужны изменения | Просто connect |
| Отладка | Проще отследить | Сложнее отследить |
| Производительность | Чуть быстрее | Чуть медленнее |

<div v-click class="mt-6 text-center text-xl">
  <strong>Правило:</strong> используйте сигналы, когда несколько систем должны реагировать на одно событие
</div>

---
layout: center
---

# Итоги 🎯

<div class="grid grid-cols-2 gap-8 mt-8">
<div v-click>

## Autoloads

- Глобальные синглтоны
- Переживают смену сцен
- Доступ по имени: `GameManager.score`
- Для: состояния игры, аудио, сохранений, переходов
- Настройка: Project Settings → Autoload

</div>
<div v-click>

## Signals

- Событийное общение
- Развязанная архитектура
- Объявление: `signal my_event`
- Отправка: `my_event.emit()`
- Подключение: `node.my_event.connect(func)`

</div>
</div>

<div v-click class="mt-8 text-center text-xl">
  <strong>Вместе:</strong> сигналы в autoload = глобальная шина событий 🚌
</div>

---
layout: center
---

# Давайте кодить! 🎮

<div class="text-xl text-gray-400 mt-4">
  Время для практики
</div>

<v-clicks class="mt-8 text-lg">

1. Создайте `GameManager` autoload с очками и жизнями
2. Добавьте сигналы: `score_changed`, `life_lost`, `game_over`
3. Соберите простую сцену, использующую оба паттерна
4. Бонус: добавьте `AudioManager` autoload

</v-clicks>

<!--
Оставшееся время кодим вместе. Открывайте Godot и повторяйте!
-->

---
layout: center
---

# Вопросы? 🤔

<div class="text-xl text-gray-400 mt-4">

**Ресурсы:**

- [Godot Docs: Singletons (Autoload)](https://docs.godotengine.org/en/stable/tutorials/scripting/singletons_autoload.html)
- [Godot Docs: Signals](https://docs.godotengine.org/en/stable/getting_started/step_by_step/signals.html)
- [GDQuest: Signals Guide](https://www.gdquest.com/tutorial/godot/gdscript/signals/)

</div>

<div class="abs-br m-6 text-gray-500">
  Сделано с Slidev ⚡
</div>
