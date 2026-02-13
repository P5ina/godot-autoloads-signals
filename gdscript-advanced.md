---
theme: default
title: "GDScript: Продвинутый синтаксис"
info: |
  Углублённое изучение возможностей GDScript:
  аннотации, сеттеры, лямбды и многое другое.
  AUCA Gamedev Club 2026.
transition: slide-left
mdc: true
lineNumbers: true
drawings:
  persist: false
---

# GDScript
## Продвинутый синтаксис

<div class="pt-12">
  <span class="text-xl text-gray-400">
    AUCA Gamedev Club — 2026
  </span>
</div>

<div class="abs-br m-6 flex gap-2">
  <a href="https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/" target="_blank" class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    📚 Docs
  </a>
</div>

<!--
Всем привет! Сегодня разберём продвинутые возможности GDScript, которые сделают ваш код чище, безопаснее и выразительнее.
-->

---

# Зачем это знать?

<v-clicks>

- 📖 **Читаемость** — код документирует сам себя
- 🔧 **Автодополнение** — редактор понимает ваш код лучше
- 🏗️ **Масштабируемость** — проект легче поддерживать
- 🧩 **Выразительность** — меньше кода для тех же задач
- 🎮 **Геймдев-паттерны** — фичи заточены под игры

</v-clicks>

<div v-click class="mt-8 p-4 bg-blue-500/10 rounded-lg border border-blue-500/20">
  💡 GDScript — это не «Python для игр». У него свои мощные фичи, заточенные под геймдев.
</div>

<!--
GDScript выглядит простым, но под капотом есть множество возможностей, которые многие разработчики не используют.
-->

---
transition: fade
layout: center
---

# Часть 1: Аннотации

<div class="text-2xl text-gray-400">
  @export, @onready, @tool и другие
</div>

---

# @export — параметры в инспекторе

```gdscript {1-3|5-8|10-13|all}
# Базовые типы
@export var speed: float = 200.0
@export var player_name: String = "Hero"

# Диапазоны и подсказки
@export_range(0, 100, 1) var health: int = 100
@export_range(0.1, 10.0, 0.1) var jump_force: float = 5.0
@export_multiline var description: String = ""

# Ресурсы и ноды
@export var texture: Texture2D
@export var explosion_scene: PackedScene
@export_file("*.json") var config_path: String
```

<div v-click class="mt-4 p-3 bg-blue-500/10 rounded-lg border border-blue-500/20">
  💡 <code>@export</code> позволяет настраивать ноды через инспектор без изменения кода
</div>

<!--
export — это мост между кодом и визуальным редактором. Дизайнеры могут настраивать параметры не трогая скрипты.
-->

---

# @export — продвинутые варианты

```gdscript {1-3|5-7|9-11|13-17|all}
# Enum в инспекторе — выпадающий список
@export var element: Element = Element.FIRE
@export var direction: Vector2 = Vector2.RIGHT

# Группы — организация в инспекторе
@export_group("Movement")
@export var max_speed: float = 300.0

@export_group("Combat")
@export var attack_power: int = 10
@export var defense: int = 5

# Подгруппы
@export_subgroup("Resistances")
@export var fire_resistance: float = 0.0
@export var ice_resistance: float = 0.0
@export_subgroup("")  # конец подгруппы
```

<div v-click class="mt-4 p-3 bg-green-500/10 rounded-lg border border-green-500/20">
  💡 <code>@export_group</code> и <code>@export_subgroup</code> делают инспектор чистым и организованным
</div>

<!--
Группировка особенно важна для нод с большим количеством параметров. Без неё инспектор превращается в хаос.
-->

---

# @onready — инициализация нод

```gdscript {1-4|6-10|12-16|all}
# ❌ Без @onready — ноды ещё не готовы!
var sprite = $Sprite2D              # null! Дерево не построено
var label = $UI/HealthLabel         # null!
var collision = $CollisionShape2D   # null!

# ✅ С @onready — ноды инициализируются в _ready()
@onready var sprite: Sprite2D = $Sprite2D
@onready var label: Label = $UI/HealthLabel
@onready var collision: CollisionShape2D = $CollisionShape2D
@onready var start_pos: Vector2 = global_position  # позиция при старте

# Эквивалент:
var sprite: Sprite2D
func _ready():
    sprite = $Sprite2D
    # ...остальная инициализация
```

<div v-click class="mt-2 p-3 bg-yellow-500/10 rounded-lg border border-yellow-500/20">
  ⚠️ <code>@onready</code> + тип = автодополнение и безопасность. <strong>Всегда</strong> указывайте тип!
</div>

<!--
onready — одна из самых частых аннотаций. Без неё — null reference ошибки гарантированы.
-->

---

# NodePath и $ — доступ к нодам

```gdscript {1-4|5-9|10-14|all}
# $ — сокращение для get_node()
var sprite = $Sprite2D                # то же что get_node("Sprite2D")
var anim = $"Animation Player"        # кавычки для пробелов в имени

# ⚠️ Можно, но лучше не надо — хрупкие пути
var label = $UI/HUD/ScoreLabel        # сломается при перемещении
var parent = $".."                     # зависит от структуры
var sibling = $"../Enemy"              # то же

# NodePath как тип — путь задаётся в инспекторе
@export var target_path: NodePath
func _ready():
    var target = get_node(target_path)
```

<div v-click class="mt-4 p-3 bg-red-500/10 rounded-lg border border-red-500/20">
  ⚠️ Используйте <code>$</code> <strong>только для прямых детей</strong>.
</div>

<!--
$ лучше всего работает с прямыми детьми: $Sprite2D, $CollisionShape2D. Для длинных путей и вложенных нод используйте % — он не сломается при реструктуризации.
-->

---

# %UniqueNode — доступ без пути

<div class="grid grid-cols-2 gap-6 mt-4">
<div>

### Проблема с $

```
Player (CharacterBody2D)
├── Body
│   └── Sprite2D
├── UI
│   └── Container
│       └── HealthBar   ← нужен этот
└── Hitbox
```

```gdscript
# Хрупкий путь — сломается
# при перемещении HealthBar
var hp = $UI/Container/HealthBar
```

</div>
<div v-click>

### Решение: %

```
Player (CharacterBody2D)
├── Body
│   └── Sprite2D
├── UI
│   └── Container
│       └── %HealthBar  ← unique!
└── Hitbox
```

```gdscript
# Работает из любого места сцены!
var hp = %HealthBar
```

</div>
</div>

<div v-click class="mt-4 p-3 bg-blue-500/10 rounded-lg border border-blue-500/20">
  💡 ПКМ по ноде → <strong>Access as Unique Name</strong> — и путь больше не сломается при перемещении
</div>

<!--
Unique name — решение главной боли $-путей. Не важно, где в дереве находится нода — % найдёт её по имени внутри сцены.
-->

---

# $ vs % — когда что использовать

<div class="grid grid-cols-2 gap-6 mt-4">
<div class="p-4 bg-blue-500/10 rounded-lg border border-blue-500/20">

### $ (NodePath)

- Прямые потомки: `$Sprite2D`
- Короткие стабильные пути
- Динамические пути через `get_node()`
- `@export var path: NodePath`

</div>
<div class="p-4 bg-green-500/10 rounded-lg border border-green-500/20">

### % (Unique Name)

- Глубоко вложенные ноды
- Ноды, которые могут перемещаться
- UI-элементы внутри контейнеров
- Ключевые ноды сцены

</div>
</div>

<v-click>

```gdscript
# Типичная комбинация в реальном проекте
@onready var sprite: Sprite2D = $Sprite2D          # прямой потомок → $
@onready var health_bar: TextureProgressBar = %HealthBar  # вложен глубоко → %
@onready var damage_label: Label = %DamageLabel     # может перемещаться → %
```

</v-click>

<div v-click class="mt-4 p-3 bg-yellow-500/10 rounded-lg border border-yellow-500/20">
  ⚠️ <code>%</code> ищет только внутри <strong>текущей сцены</strong> — не видит ноды из вложенных сцен
</div>

<!--
Простое правило: $ для прямых потомков, % для всего остального. И всегда комбинируйте с @onready и типом.
-->

---

# @tool — скрипт в редакторе

```gdscript
@tool
extends Sprite2D

@export var radius: float = 100.0:
    set(value):
        radius = value
        queue_redraw()  # перерисовать в редакторе!

func _draw():
    draw_circle(Vector2.ZERO, radius, Color(1, 0, 0, 0.3))
```

<v-click>

<div class="mt-4 grid grid-cols-2 gap-4">
<div class="p-3 bg-green-500/10 rounded-lg border border-green-500/20">

### Когда использовать
- Визуализация зон (AI, Area2D)
- Автогенерация нод в редакторе
- Превью эффектов и частиц
- Кастомные плагины

</div>
<div class="p-3 bg-red-500/10 rounded-lg border border-red-500/20">

### Осторожно!
- Код выполняется **в редакторе**
- Баг может сломать сцену
- Используйте `Engine.is_editor_hint()`
- Не вызывайте игровую логику

</div>
</div>

</v-click>

<!--
tool-скрипты выполняются прямо в редакторе. Мощная фича, но требует осторожности.
-->

---
transition: fade
layout: center
---

# Часть 2: Сеттеры и геттеры

<div class="text-2xl text-gray-400">
  Реактивные переменные и инкапсуляция
</div>

---
layout: center
---

<img src="/assets/encapsulation_meme.jpg" class="h-100 rounded-lg shadow-xl" />

<!--
Мем-пауза перед тем, как погружаемся в инкапсуляцию.
-->

---

# Что такое инкапсуляция?

**Инкапсуляция** — объединение данных и методов в одном объекте и сокрытие внутренней логики.

<div class="mt-4">

```
╔══════════════════════════════════════╗
║           Player (объект)            ║
║                                      ║
║   Приватное (скрыто внутри):         ║
║     _health, _mana, _inventory       ║
║     _update_stats(), _die()          ║
║                                      ║
║   Публичное (доступно снаружи):      ║
║     take_damage(), heal(), get_hp()  ║
║     is_alive, health_percent         ║
╚══════════════════════════════════════╝
```

</div>

<v-clicks class="mt-4">

- **Данные** защищены от прямого доступа — нельзя записать мусор
- **Методы** контролируют как данные изменяются — валидация, побочные эффекты
- **Интерфейс** простой и стабильный — внутри можно менять что угодно
- В GDScript: `_prefix` для приватного, **сеттеры/геттеры** для контроля доступа

</v-clicks>

<!--
Инкапсуляция — один из четырёх столпов ООП. Думайте о ней как о капсуле с лекарством: внутри сложный состав, снаружи — простая оболочка.
-->

---

# set / get

```gdscript {1-7|9-16|all}
# Сеттер — выполняется при записи
var health: int = 100:
    set(value):
        health = clamp(value, 0, max_health)
        health_changed.emit(health)
        if health <= 0:
            died.emit()

# Геттер — выполняется при чтении
var is_alive: bool:
    get:
        return health > 0  # вычисляемое свойство

var display_name: String:
    get:
        return "[%s] %s" % [team, player_name]
```

<div v-click class="mt-4 p-3 bg-blue-500/10 rounded-lg border border-blue-500/20">
  💡 Сеттер + сигнал = <strong>реактивность</strong>. Любое изменение автоматически уведомляет подписчиков.
</div>

<!--
Сеттеры заменяют отдельные методы типа set_health(). Геттеры создают вычисляемые свойства без хранения данных.
-->

---

# Полный пример: Health система

```gdscript
signal health_changed(new_health: int, max_health: int)
signal died

@export var max_health: int = 100

var health: int = max_health:
    set(value):
        set_health(value)

func set_health(h: float):
    health = 10.0

var health_percent: float:
    get:
        return float(health) / float(max_health)

var is_alive: bool:
    get:
        return health > 0
```

<div v-click class="mt-2 p-3 bg-green-500/10 rounded-lg border border-green-500/20">
  Просто пишем <code>health -= 25</code> — всё остальное происходит автоматически!
</div>

<!--
Красота этого подхода в том, что вызывающий код не знает о логике внутри сеттера. Просто меняете значение.
-->

---
transition: fade
layout: center
---

# Часть 3: Лямбды и Callable

<div class="text-2xl text-gray-400">
  Анонимные функции и функциональный стиль
</div>

---

# Lambda-функции

```gdscript {1-4|6-10|12-16|all}
# Простая лямбда
var greet = func(name: String) -> String:
    return "Привет, %s!" % name
print(greet.call("Godot"))  # "Привет, Godot!"

# Лямбда в connect
button.pressed.connect(func():
    print("Кнопка нажата!")
    score += 1
)

# Лямбда с одноразовым подключением
enemy.died.connect(func():
    score += 100
    check_wave_clear()
, CONNECT_ONE_SHOT)
```

<div v-click class="mt-4 p-3 bg-yellow-500/10 rounded-lg border border-yellow-500/20">
  ⚠️ Лямбды вызываются через <code>.call()</code>, не напрямую: <code>my_func.call(args)</code>
</div>

<!--
Лямбды отлично подходят для коротких callback'ов. Но не злоупотребляйте — длинные лямбды ухудшают читаемость.
-->

---

# Функциональный стиль с массивами

```gdscript {1-3|5-8|10-13|15-20|all}
var enemies = get_tree().get_nodes_in_group("enemies")
var items: Array[Dictionary] = [
    {"name": "Sword", "price": 100}, {"name": "Shield", "price": 50}]

# filter — отфильтровать
var alive = enemies.filter(func(e): return e.is_alive)
var cheap = items.filter(
    func(item): return item.price < 80)

# map — преобразовать
var names = enemies.map(func(e): return e.name)
var discounted = items.map(
    func(item): return {"name": item.name, "price": item.price * 0.8})

# reduce — свернуть
var total_hp = enemies.reduce(
    func(sum, e): return sum + e.health, 0)

# any / all
var has_alive = enemies.any(func(e): return e.is_alive)
var all_dead = enemies.all(func(e): return not e.is_alive)
```

<!--
filter, map, reduce — три кита функционального программирования. В GDScript они работают через Callable.
-->

---
transition: fade
layout: center
---

# Часть 4: Enum

<div class="text-2xl text-gray-400">
  Перечисления для читаемого и безопасного кода
</div>

---

# Enum

```gdscript {1-2|4-6|8-13|15-19|all}
# Базовый enum
enum Direction { UP, DOWN, LEFT, RIGHT }

# С явными значениями
enum Layer { PLAYER = 1, ENEMY = 2, BULLET = 4, WALL = 8 }

# Использование
var facing: Direction = Direction.RIGHT

func move(dir: Direction) -> void:
    match dir:
        Direction.UP:    velocity.y = -speed
        Direction.DOWN:  velocity.y = speed

# Битовые флаги (маски)
var collision_mask: int = Layer.ENEMY | Layer.WALL

func can_hit(target_layer: int) -> bool:
    return collision_mask & target_layer != 0
```

<div v-click class="mt-4 p-3 bg-green-500/10 rounded-lg border border-green-500/20">
  💡 Enum + match = <strong>безопасная</strong> обработка состояний. Ни одно значение не будет пропущено.
</div>

<!--
Enum заменяют магические числа и строки. Код становится самодокументируемым и защищённым от опечаток.
-->

---

# Enum для State Machine

```gdscript
enum State { IDLE, RUN, JUMP, FALL, ATTACK, HURT, DEAD }

var state: State = State.IDLE:
    set(new_state):
        if state == new_state:
            return
        _exit_state(state)
        state = new_state
        _enter_state(state)

func _enter_state(s: State) -> void:
    match s:
        State.IDLE:   $Anim.play("idle")
        State.RUN:    $Anim.play("run")
        State.JUMP:   $Anim.play("jump"); velocity.y = -jump_force
        State.ATTACK: $Anim.play("attack")
        State.HURT:   $Anim.play("hurt"); _start_invincibility()
        State.DEAD:   $Anim.play("death"); set_physics_process(false)
```

<div v-click class="mt-4 p-3 bg-blue-500/10 rounded-lg border border-blue-500/20">
  💡 Сеттер на enum = автоматические переходы между состояниями при простом присваивании
</div>


<!--
Классический паттерн: enum для состояний + сеттер для переходов. Чисто, читаемо, расширяемо.
-->

---
transition: fade
layout: center
---

# Часть 5: Классы и наследование

<div class="text-2xl text-gray-400">
  class_name, inner classes, наследование
</div>

---

# class_name и наследование

```gdscript {1-8|10-17|all}
# base_weapon.gd
class_name BaseWeapon
extends Node2D

@export var damage: int = 10
@export var cooldown: float = 0.5

func attack() -> void:
    pass  # переопределяется в наследниках

# sword.gd
class_name Sword
extends BaseWeapon

func attack() -> void:
    # вызываем родительский метод
    super.attack()
    $AnimationPlayer.play("slash")
    _deal_damage_in_area()
```

<div v-click class="mt-4 p-3 bg-blue-500/10 rounded-lg border border-blue-500/20">
  💡 <code>class_name</code> делает класс доступным <strong>глобально</strong> — можно использовать как тип и в <code>is</code> проверках
</div>

<!--
class_name — не обязательный, но очень полезный. Позволяет использовать класс как тип, проверять через is, и видеть в редакторе.
-->

---

# Inner classes

```gdscript {1-8|10-15|all}
# inventory.gd
class_name Inventory

class Item:
    var name: String
    var quantity: int
    func _init(n: String, q: int = 1) -> void:
        name = n; quantity = q

var items: Array[Item] = []

func add(item_name: String) -> void:
    var existing = items.filter(func(i): return i.name == item_name)
    if existing.size() > 0: existing[0].quantity += 1
    else: items.append(Item.new(item_name))
```

<div v-click class="mt-4 p-3 bg-green-500/10 rounded-lg border border-green-500/20">
  💡 Inner class = вспомогательная структура данных, связанная с основным классом
</div>

<!--
Inner classes хороши для вспомогательных типов данных: Item в Inventory, Entry в SaveManager и т.д.
-->

---

# is и as — проверка и приведение типов

```gdscript {1-6|8-13|15-20|all}
# Проверка типа
func _on_body_entered(body: Node2D) -> void:
    if body is Player:
        print("Это игрок!")
    elif body is Enemy:
        body.take_damage(10)

# Безопасное приведение с as
func interact(node: Node) -> void:
    var chest = node as Chest
    if chest:              # null если приведение не удалось
        chest.open()
        collect_loot(chest.loot)

# Комбинация с типизированными массивами
func get_all_enemies() -> Array[Enemy]:
    var result: Array[Enemy] = []
    for child in get_children():
        if child is Enemy:
            result.append(child)
    return result
```

<!--
is проверяет тип, as пытается привести. as возвращает null при неудаче — безопасная альтернатива прямому приведению.
-->

---
transition: fade
layout: center
---

# Часть 6: Строки и форматирование

<div class="text-2xl text-gray-400">
  Шаблоны, форматирование и полезные методы
</div>

---

# Форматирование строк

```gdscript {1-4|6-9|11-15|17-20|all}
# printf-стиль
var msg = "Игрок %s нанёс %d урона" % [player_name, damage]
var formatted = "HP: %d/%d (%.1f%%)" % [health, max_health, percent]
var padded = "Score: %05d" % score   # "Score: 00042"

# format() — именованные плейсхолдеры
var text = "Level {level}: {name}".format({
    "level": current_level,
    "name": level_name
})

# Многострочные строки
var dialog = """
Добро пожаловать, %s!
Ваш текущий уровень: %d
Удачи в приключениях!
""" % [player_name, level]

# Повтор и соединение
var separator = "-".repeat(40)
var csv = ",".join(["sword", "shield", "potion"])
```

<!--
GDScript поддерживает printf-стиль и format. Используйте то, что удобнее в контексте.
-->

---

# Полезные методы строк

```gdscript {1-5|7-11|13-17|all}
# Проверки
"Hello".begins_with("He")    # true
"image.png".get_extension()   # "png"
"  spaces  ".strip_edges()    # "spaces"
"player_name".to_pascal_case() # "PlayerName"

# Разделение и поиск
"one,two,three".split(",")        # ["one", "two", "three"]
"res://levels/level_1.tscn".get_file()  # "level_1.tscn"
"res://levels/level_1.tscn".get_base_dir() # "res://levels"
"/path/to/file.txt".get_basename() # "/path/to/file"

# Подстановка
"Hello World".replace("World", "Godot")  # "Hello Godot"
"Attack: {value}".format({"value": 42})  # "Attack: 42"
"CamelCase".to_snake_case()              # "camel_case"
"snake_case".to_camel_case()             # "snakeCase"
```

<!--
Строковые методы в GDScript покрывают большинство задач. Не нужны регулярные выражения для базовых операций.
-->

---
transition: fade
layout: center
---

# Часть 7: Полезные конструкции

<div class="text-2xl text-gray-400">
  Тернарный оператор, await, preload и другие
</div>

---

# Тернарный оператор и однострочники

```gdscript {1-3|5-7|9-12|14-17|all}
# Тернарный оператор
var label = "Alive" if health > 0 else "Dead"
var direction = -1 if is_flipped else 1

# Однострочный if
if health <= 0: die()

# Тернарный в аргументах
sprite.modulate = Color.RED if is_hurt else Color.WHITE
velocity.x = speed * (-1.0 if facing_left else 1.0)
$Label.text = "%d item%s" % [count, "" if count == 1 else "s"]

# Clamp — ограничение диапазона
health = clampi(health, 0, max_health)
position.x = clampf(position.x, 0.0, screen_width)
rotation = clampf(rotation, -PI / 4, PI / 4)
```

<!--
Тернарный оператор в GDScript — как в Python. Отлично подходит для коротких условных присваиваний.
-->

---

# preload vs load

```gdscript {1-5|7-11|13-18|all}
# preload — загрузка при компиляции (быстрее)
const BulletScene: PackedScene = preload("res://bullet.tscn")
const HitSound: AudioStream = preload("res://sfx/hit.wav")
const EnemySprite: Texture2D = preload("res://sprites/enemy.png")
# ☝️ Аргумент ОБЯЗАН быть строковым литералом

# load — загрузка в рантайме (гибче)
var scene_path = "res://levels/level_%d.tscn" % level_num
var scene: PackedScene = load(scene_path)
# ☝️ Путь может быть переменной

# Когда что использовать
# ✅ preload — для постоянных ресурсов (пули, звуки, эффекты)
const Explosion = preload("res://effects/explosion.tscn")

# ✅ load — для динамических ресурсов (уровни, контент)
func load_level(num: int) -> void:
    var level = load("res://levels/level_%d.tscn" % num)
    get_tree().change_scene_to_packed(level)
```

<!--
Правило простое: если путь известен заранее — preload. Если путь строится динамически — load.
-->

---

# static функции и переменные

```gdscript {1-7|9-15|all}
# Статические методы — вызываются без инстанса
class_name MathUtils

static func chance(percent: float) -> bool:
    return randf() * 100.0 < percent

static func random_direction() -> Vector2:
    return Vector2(randf_range(-1, 1), randf_range(-1, 1)).normalized()

# Использование — не нужно создавать объект
func _process(delta: float) -> void:
    if MathUtils.chance(1.0):  # 1% шанс каждый кадр
        spawn_particle()

    var dir = MathUtils.random_direction()
    velocity = dir * speed
```

<div v-click class="mt-4 p-3 bg-blue-500/10 rounded-lg border border-blue-500/20">
  💡 Статические функции — отлично для утилит: математика, генерация, конвертация
</div>

<!--
static — когда функция не зависит от состояния объекта. Как классический utility class.
-->

---
transition: fade
layout: center
---

# Часть 8: Нейминг и стиль кода

<div class="text-2xl text-gray-400">
  Конвенции, которые должен знать каждый
</div>

---

# Нейминг в GDScript

<div class="grid grid-cols-2 gap-6 mt-4">
<div>

### Правила

```gdscript
# snake_case — переменные и функции
var player_health: int = 100
func calculate_damage() -> int:

# PascalCase — классы и enum
class_name PlayerController
enum WeaponType { SWORD, BOW }

# SCREAMING_SNAKE — константы
const MAX_SPEED: float = 400.0
const GRAVITY: float = 980.0

# _prefix — приватные
var _internal_state: int = 0
func _update_cache() -> void:
```

</div>
<div v-click>

### Сигналы — прошедшее время

```gdscript
# ✅ Событие уже произошло
signal health_changed
signal enemy_died
signal item_collected
signal level_completed

# ❌ Не используйте
signal change_health
signal die
signal on_collect
```

</div>
</div>


<!--
Нейминг — это не про вкус, а про конвенции. Весь Godot использует эти правила, и ваш код должен тоже.
-->

---

# Нейминг нод в сцене

<div class="grid grid-cols-2 gap-6 mt-4">
<div>

### PascalCase — всегда

```
✅ Хорошо:
Player
EnemySpawner
HealthBar
MainMenu
BulletSpawnPoint
AnimationPlayer

❌ Плохо:
player
enemy_spawner
health-bar
MAIN_MENU
```

</div>
<div v-click>

### Суффикс = тип или роль

```
Player (CharacterBody2D)
├── Sprite2D
├── CollisionShape2D
├── HitboxArea        ← Area2D
├── AttackTimer       ← Timer
├── DeathSound        ← AudioStreamPlayer
└── UI
    ├── HealthBar     ← TextureProgressBar
    └── DamageLabel   ← Label
```

<div class="mt-4 p-3 bg-green-500/10 rounded-lg border border-green-500/20">
  💡 Имя ноды = <strong>что делает</strong>, не какой у неё тип
</div>

</div>
</div>

<!--
Ноды в Godot — PascalCase. Суффикс помогает понять назначение без открытия инспектора. HitboxArea понятнее, чем просто Area2D.
-->

---

# Стиль кода

<div class="grid grid-cols-2 gap-6 mt-2">
<div>

### Порядок в скрипте

```gdscript
# 1. class_name и extends
class_name Player
extends CharacterBody2D

# 2. Сигналы
signal died

# 3. Константы и enum
const MAX_HP = 100
enum State { IDLE, RUN }

# 4. @export
@export var speed: float = 200.0

# 5. @onready
@onready var sprite := $Sprite2D

# 6. Обычные переменные
var health: int = MAX_HP
```

</div>
<div v-click>

### Порядок (продолжение)

```gdscript
# 7. Встроенные методы
func _ready() -> void:
    pass

func _process(delta: float) -> void:
    pass

func _physics_process(delta: float) -> void:
    pass

# 8. Публичные методы
func take_damage(amount: int) -> void:
    pass

# 9. Приватные методы
func _update_animation() -> void:
    pass
```

</div>
</div>


<!--
Фиксированный порядок секций в скрипте — как оглавление в книге. Любой разработчик сразу найдёт нужное место.
-->

---

# Именование файлов

```
res://
├── scenes/
│   ├── player/
│   │   ├── player.tscn           ← snake_case.tscn
│   │   └── player.gd             ← snake_case.gd
│   └── enemies/
│       ├── base_enemy.tscn
│       └── base_enemy.gd
├── scripts/
│   ├── autoloads/
│   │   ├── game_manager.gd       ← autoloads в отдельной папке
│   │   └── audio_manager.gd
│   └── resources/
│       └── weapon_data.gd
└── assets/
    ├── sprites/
    └── sfx/
```

<div v-click class="mt-4 p-3 bg-blue-500/10 rounded-lg border border-blue-500/20">
  💡 Файлы — <strong>snake_case</strong>, ноды — <strong>PascalCase</strong>, константы — <strong>SCREAMING_SNAKE</strong>
</div>

<!--
Структура проекта — то, что вы увидите первым. Держите её чистой и предсказуемой.
-->

---
layout: center
---

# Итоги

<div class="grid grid-cols-2 gap-8 mt-8">
<div v-click>

## Структура кода

- Enum вместо магических чисел
- `is` / `as` для проверки типов
- Сеттеры для валидации
- class_name и inner classes

</div>
<div v-click>

## Выразительность

- match с деструктуризацией
- Лямбды и функциональный стиль
- @export для визуальной настройки
- preload / load для ресурсов

</div>
</div>

<div v-click class="mt-8 text-center text-xl">
  <strong>Используйте весь потенциал GDScript</strong> — будущий вы скажет спасибо!
</div>

---
transition: fade
layout: center
---

# Практика: Top-Down Player

<div class="text-2xl text-gray-400">
  Пишем контроллер игрока с State Machine и анимациями
</div>

---

# Структура сцены

```
Player (CharacterBody2D)
├── Sprite2D
├── CollisionShape2D
└── AnimationPlayer
```

<v-click>

### Что будем использовать

- **enum** — состояния (IDLE, RUN, DASH)
- **сеттер** на state — автоматическое переключение анимаций
- **@export** — настраиваемые параметры движения
- **@onready** — ссылки на ноды
- **_prefix** — приватные методы

</v-click>

---

# Player Controller (часть 1)

```gdscript
class_name Player
extends CharacterBody2D

enum State { IDLE, RUN, DASH }

signal state_changed(new_state: State)

@export_group("Movement")
@export var speed: float = 200.0
@export_group("Dash")
@export var dash_speed: float = 600.0
@export var dash_duration: float = 0.15
@export var dash_cooldown: float = 0.5

@onready var sprite: Sprite2D = $Sprite2D
@onready var anim: AnimationPlayer = $AnimationPlayer

var _dash_ready := true

var state: State = State.IDLE:
    set(new_state):
        if state == new_state: return
        state = new_state
        _on_state_changed(state)
        state_changed.emit(state)
```

---

# Player Controller (часть 2)

```gdscript
var _last_direction := Vector2.DOWN

func _physics_process(_delta: float) -> void:
    if state == State.DASH:
        move_and_slide()
        return

    var direction := Input.get_vector(
        "move_left", "move_right", "move_up", "move_down")

    velocity = direction * speed

    if direction != Vector2.ZERO:
        _last_direction = direction
        _update_sprite_direction(direction)

    if Input.is_action_just_pressed("dash") and _can_dash():
        _start_dash()
        return

    move_and_slide()
    _update_state()
```

---

# Player Controller (часть 3)

```gdscript
func _update_state() -> void:
    if velocity.length() > 10:
        state = State.RUN
    else:
        state = State.IDLE

func _on_state_changed(new_state: State) -> void:
    match new_state:
        State.IDLE: anim.play("idle")
        State.RUN:  anim.play("run")
        State.DASH: anim.play("dash")

func _update_sprite_direction(direction: Vector2) -> void:
    if abs(direction.x) > abs(direction.y):
        sprite.flip_h = direction.x < 0
```

---

# Player Controller (часть 4)

```gdscript
func _can_dash() -> bool:
    return state != State.DASH and _dash_ready

func _start_dash() -> void:
    state = State.DASH
    _dash_ready = false
    var dash_dir := _last_direction.normalized()
    var tween := create_tween()
    tween.tween_property(self, "velocity",
        Vector2.ZERO, dash_duration
    ).from(dash_dir * dash_speed)
    await tween.finished
    state = State.IDLE
    await get_tree().create_timer(dash_cooldown).timeout
    _dash_ready = true
```

<div v-click class="mt-4 p-3 bg-green-500/10 rounded-lg border border-green-500/20">
  Всё что мы изучили — enum, сеттер, match, @export, @onready, _prefix — в одном скрипте!
</div>

---
layout: center
---

# Ваша очередь!

<v-clicks class="mt-8 text-lg">

1. Создайте сцену **Player** с нодами из примера
2. Перепишите контроллер **самостоятельно** (не копируйте!)
3. Добавьте 4 направления анимаций (**idle_down**, **run_up**, ...)
4. Добавьте состояние **ATTACK** с блокировкой движения
5. Бонус: добавьте **knockback** при получении урона

</v-clicks>

<!--
Главное — не копипаст, а понимание. Попробуйте написать с нуля, подглядывая в слайды.
-->

---
layout: center
---

# Вопросы?

<div class="text-xl text-gray-400 mt-4">

**Ресурсы:**

- [GDScript Reference](https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_basics.html)
- [GDScript Exports](https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_exports.html)
- [GDScript Style Guide](https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_styleguide.html)

</div>

<div class="abs-br m-6 text-gray-500">
  Сделано с Slidev ⚡
</div>
