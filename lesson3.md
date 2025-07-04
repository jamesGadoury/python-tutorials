# Lesson 3.1 – Classes vs `@dataclass` (Player Entity)  
*Module theme: build a scrolling platformer*

---

## 1 · Theory bites
* Classic class syntax vs `dataclass` auto-generated dunder methods  
* Why `pygame.sprite.Sprite` is **not** a dataclass (mutable, complex init)  
* Hybrid pattern: **plain dataclass** for data-only components, classic class for behaviour

---

## 2 · Guided steps
1. **Component dataclass** – pure data, no Pygame inheritance.

   ```python
   from dataclasses import dataclass
   import pygame

   @dataclass
   class Kinematics:
       pos: pygame.Vector2
       vel: pygame.Vector2
       acc: pygame.Vector2 = pygame.Vector2()
   ```

2. **Player sprite** – owns a `Kinematics` component.

   ```python
   class Player(pygame.sprite.Sprite):
       def __init__(self, x: int, y: int):
           super().__init__()
           self.image = pygame.image.load("assets/player_idle.png").convert_alpha()
           self.rect  = self.image.get_rect(topleft=(x, y))
           self.kin   = Kinematics(pygame.Vector2(x, y), pygame.Vector2())

       def update(self, dt: float, tiles):
           self.kin.acc = pygame.Vector2(0, 0.5)        # gravity
           keys = pygame.key.get_pressed()
           if keys[pygame.K_a]:
               self.kin.acc.x = -0.4
           if keys[pygame.K_d]:
               self.kin.acc.x = 0.4
           if keys[pygame.K_SPACE] and self.on_ground(tiles):
               self.kin.vel.y = -10

           # Euler integration
           self.kin.vel += self.kin.acc * dt
           self.kin.pos += self.kin.vel * dt
           self.rect.topleft = self.kin.pos
   ```

3. **Delta-time** (`dt`) – pass `clock.tick(60) / 16` for smooth motion regardless of FPS.

---

## 3 · Checkpoint

```bash
python3 player_test.py      # walk & jump on static ground
git add player.py kinematics.py
git commit -m "Lesson 3.1: Player entity with dataclass kinematics"
```

# Lesson 3.2 – Inheritance & Mix-ins (Enemy NPC Super-class)

---

## 1 · Theory bites
* **Inheritance tree** – `NPCBase → (Patroller, Chaser, Shooter)`  
* **Mix-ins** for optional abilities (`JumpMixin`, `HealthMixin`)  
* Composition vs inheritance (when to favour each)

---

## 2 · Guided steps
1. **NPCBase**

   ```python
   class NPCBase(pygame.sprite.Sprite):
       speed = 2
       def __init__(self, x, y):
           super().__init__()
           self.image = pygame.Surface((32, 32))
           self.image.fill("red")
           self.rect = self.image.get_rect(topleft=(x, y))

       def move(self, dt):
           raise NotImplementedError
   ```

2. **Patroller**

   ```python
   class Patroller(NPCBase):
       def __init__(self, x, y, left_bound, right_bound):
           super().__init__(x, y)
           self.left, self.right = left_bound, right_bound
           self.dir = 1

       def move(self, dt):
           self.rect.x += self.dir * self.speed * dt
           if self.rect.left < self.left or self.rect.right > self.right:
               self.dir *= -1
   ```

3. **Health mix-in**

   ```python
   class HealthMixin:
       hp_max = 3
       def __init__(self):
           self.hp = self.hp_max
       def take_hit(self, dmg=1):
           self.hp -= dmg
           return self.hp <= 0
   ```

4. **Combine**

   ```python
   class Shooter(HealthMixin, NPCBase):
       ...
   ```

---

## 3 · Checkpoint

```bash
python3 npc_test.py         # patroller moves between bounds
git add npc.py
git commit -m "Lesson 3.2: NPC hierarchy with mix-ins"
```

# Lesson 3.3 – Scene / State Manager

---

## 1 · Theory bites
* Finite State Machine (FSM) vs stack-based **Scene** system  
* Decoupling **game states**: `Title`, `Play`, `Pause`, `GameOver`  
* Push / pop scenes for menus over gameplay

---

## 2 · Guided steps
1. **Base Scene**

   ```python
   class Scene:
       def __init__(self, game):
           self.game = game
       def handle_event(self, e): pass
       def update(self, dt):     pass
       def draw(self, surf):     pass
   ```

2. **SceneManager**

   ```python
   class SceneManager:
       def __init__(self, start_scene: Scene):
           self.stack = [start_scene]
       def push(self, scene): self.stack.append(scene)
       def pop(self):        self.stack.pop()
       def current(self):    return self.stack[-1]
   ```

3. **Implement Title → Play transition**

   ```python
   if e.type == pygame.KEYDOWN:
       self.game.scenes.push(PlayScene(self.game))
   ```

4. **Pause overlay** – push `PauseScene`, draw underlying play scene first.

---

## 3 · Checkpoint

```bash
python3 game.py             # title, play, pause working
git add scene.py game.py
git commit -m "Lesson 3.3: Scene manager implemented"
```

# Lesson 3.4 – Tiled Maps, CSV/JSON & Camera

---

## 1 · Theory bites
* Using **Tiled Map Editor** – export CSV layer or JSON map  
* `pytmx` vs rolling your own loader  
* Camera / viewport: follow target but clamp to level bounds

---

## 2 · Guided steps
1. **Export from Tiled**

   * Tileset `tiles.png` (32 × 32).  
   * Layers: `ground`, `spikes`, `decor`.  
   * Export as `level1.json`.

2. **Load tiles**

   ```python
   import json, pygame

   def load_map(path):
       data = json.load(open(path))
       for layer in data["layers"]:
           if layer["name"] == "ground":
               for idx, gid in enumerate(layer["data"]):
                   if gid == 0: continue
                   x = (idx % data["width"])  * 32
                   y = (idx // data["width"]) * 32
                   rect = pygame.Rect(x, y, 32, 32)
                   ground_tiles.append(rect)
   ```

3. **Camera class**

   ```python
   class Camera:
       def __init__(self, width, height):
           self.rect = pygame.Rect(0, 0, width, height)

       def apply(self, target_rect):
           return target_rect.move(-self.rect.x, -self.rect.y)

       def update(self, target_rect, level_w, level_h):
           self.rect.center = target_rect.center
           self.rect.clamp_ip(pygame.Rect(0, 0, level_w, level_h))
   ```

4. **Render**

   ```python
   for tile in ground_tiles:
       screen.blit(tileset, camera.apply(tile), area=tile_index_rect)
   player.draw(screen, camera)
   ```

---

## 3 · Checkpoint

```bash
python3 play_level.py       # scrolls as player moves
git add camera.py map_loader.py
git commit -m "Lesson 3.4: Tiled map loading and camera scroll"
```

# Lesson 3.5 – Saving & Loading (JSON, `pathlib`)

---

## 1 · Theory bites
* Game state serialisation: what to save? (`level`, `player pos`, `hp`, `inventory`)  
* Using `json` + `dataclasses.asdict`  
* File I/O with `pathlib.Path.home()` for cross-platform save dir

---

## 2 · Guided steps
1. **Serialisation helpers**

   ```python
   import json, pathlib, dataclasses

   SAVE_PATH = pathlib.Path.home() / ".platformer" / "save.json"
   SAVE_PATH.parent.mkdir(exist_ok=True)

   def save_game(state):
       SAVE_PATH.write_text(json.dumps(state, indent=2))

   def load_game():
       if SAVE_PATH.exists():
           return json.loads(SAVE_PATH.read_text())
       return None
   ```

2. **Gather state**

   ```python
   state = {
       "level": current_level,
       "player": dataclasses.asdict(player.kin),
       "hp": player.hp
   }
   save_game(state)
   ```

3. **Resume**

   ```python
   s = load_game()
   if s:
       player.kin.pos = pygame.Vector2(s["player"]["pos"])
       current_level  = s["level"]
   ```

---

## 3 · Checkpoint

```bash
python3 save_test.py        # F5 to save, F9 to load
git add savegame.py
git commit -m "Lesson 3.5: JSON save/load system"
```

# Lesson 3.6 – Particle FX & Basic Shaders (Optional Polish)

---

## 1 · Theory bites
* Procedural particle systems – pool vs spawn-and-forget  
* Additive blending (`pygame.BLEND_ADD`) for glows  
* GLSL shaders via `pygame._sdl2.video` (SDL 2.0.18+) – preview only

---

## 2 · Guided steps
1. **Particle sprite**

   ```python
   class Particle(pygame.sprite.Sprite):
       def __init__(self, pos):
           super().__init__()
           self.image = pygame.Surface((4, 4), pygame.SRCALPHA)
           pygame.draw.circle(self.image, (255, 200, 0, 180), (2, 2), 2)
           self.rect = self.image.get_rect(center=pos)
           self.vel  = pygame.Vector2(random.uniform(-2, 2), random.uniform(-3, 0))
           self.life = 30

       def update(self):
           self.rect.center += self.vel
           self.life -= 1
           self.image.set_alpha(max(0, self.life * 8))
           if self.life <= 0:
               self.kill()
   ```

2. **Spawn on jump / hit**

   ```python
   for _ in range(10):
       particles.add(Particle(player.rect.midbottom))
   ```

3. **Additive blit**

   ```python
   particles.draw(screen, special_flags=pygame.BLEND_ADD)
   ```

4. **Optional shader** – full-screen colour grade using `GLSL` if supported.

---

## 3 · Checkpoint

```bash
python3 play_level.py      # see jump dust or hit sparks
git add particles.py
git commit -m "Lesson 3.6: Particle effects added"
```

# Mini-Capstone C – Scrolling Platformer Release

---

## Goals
* Ship a **1-level platformer** with at least one enemy type, collectibles, checkpoints, and a polished menu.  
* Demonstrate principles from Lessons 3.1–3.6.

---

## Requirements
1. **Core loop** – run, jump, enemy avoidance or attack.  
2. **Camera & map** – level size ≥ 4× viewport; uses Tiled JSON.  
3. **Enemies** – at least **Patroller** and one other subclass.  
4. **Collectibles** (coins/keys) – update score HUD.  
5. **Save point** – respawn at last checkpoint on death; persist via save file.  
6. **Particles** – landing dust OR coin sparkle.  
7. **Menu & pause** – stack-based scene manager.  
8. **Audio** – background track + 3 SFX.  
9. **Documentation** – README with controls, screenshots, short GIF.  
10. **Quality** – `ruff`/`pytest` CI green; < 5 % code duplication (see `flake8-eradicate`).

---

## Suggested repo tree
```
platformer/
├─ core/
│  ├─ __init__.py
│  ├─ scene.py
│  ├─ camera.py
│  ├─ savegame.py
│  └─ particles.py
├─ entities/
│  ├─ player.py
│  ├─ npc.py
│  └─ collectibles.py
├─ levels/
│  └─ level1.json
├─ assets/
│  ├─ tiles.png
│  ├─ music.ogg
│  └─ sfx/
└─ main.py
```

---

## Deliverable checklist
* [ ] `python3 -m platformer.main` plays start-to-finish, ~60 FPS  
* [ ] Death → respawn at checkpoint  
* [ ] JSON save reloads last checkpoint  
* [ ] README shows badges (`CI`, `CodeQL`, `License`)  
* [ ] GitHub Release v0.1 with **Linux AppImage** via PyInstaller

```bash
git add platformer/ .github/workflows/
git commit -m "Mini-Capstone C: Platformer MVP"
git tag -a "module-3-complete" -m "Finished Module 3"
```

🚀  Module 3 complete! You now handle larger-scale game architecture, level streaming, save systems, and visual polish. Onward to Module 4 for networking, optimisation, and distribution.

