# Lesson 2.1 – The Pygame Main Loop & Timing  
*Project slice: draw paddles + ball for “Pong​α”*

---

## 1 · Theory bites
* `pygame.init()`, `display.set_mode()`  
* The **main loop** pattern  
* `Clock.tick(fps)` for frame-rate independence  
* Drawing primitives (`pygame.draw.rect`, `draw.circle`)  
* Handling keyboard events

---

## 2 · Guided steps
1. **Skeleton**

   ```python
   import pygame, sys

   pygame.init()
   WIDTH, HEIGHT = 800, 600
   screen = pygame.display.set_mode((WIDTH, HEIGHT))
   clock = pygame.time.Clock()
   ```

2. **Create game objects**

   ```python
   PADDLE_W, PADDLE_H = 10, 100
   ball       = pygame.Rect(WIDTH//2 - 10, HEIGHT//2 - 10, 20, 20)
   left_pad   = pygame.Rect(30, HEIGHT//2 - PADDLE_H//2, PADDLE_W, PADDLE_H)
   right_pad  = pygame.Rect(WIDTH-40, HEIGHT//2 - PADDLE_H//2, PADDLE_W, PADDLE_H)
   ball_vx, ball_vy = 4, 4          # pixels per frame
   paddle_speed      = 6
   ```

3. **Main loop**

   ```python
   while True:
       for event in pygame.event.get():
           if event.type == pygame.QUIT:
               pygame.quit(); sys.exit()

       keys = pygame.key.get_pressed()
       if keys[pygame.K_w] and left_pad.top > 0:
           left_pad.y -= paddle_speed
       if keys[pygame.K_s] and left_pad.bottom < HEIGHT:
           left_pad.y += paddle_speed
       if keys[pygame.K_UP] and right_pad.top > 0:
           right_pad.y -= paddle_speed
       if keys[pygame.K_DOWN] and right_pad.bottom < HEIGHT:
           right_pad.y += paddle_speed

       ball.x += ball_vx
       ball.y += ball_vy

       screen.fill("black")
       pygame.draw.rect(screen, "white", left_pad)
       pygame.draw.rect(screen, "white", right_pad)
       pygame.draw.ellipse(screen, "white", ball)
       pygame.display.flip()
       clock.tick(60)
   ```

4. **Bounce off walls** – reverse `ball_vy` when `ball.top <= 0` or `ball.bottom >= HEIGHT`.

---

## 3 · Checkpoint

```bash
python3 pong_basic.py          # paddles move; ball bounces off top/bottom walls
git add pong_basic.py
git commit -m "Lesson 2.1: Pong alpha – main loop and timing"
```

# Lesson 2.2 – Sprites, Groups & Collisions  
*Project slice: finish core Pong gameplay*

---

## 1 · Theory bites
* `pygame.sprite.Sprite` lifecycle (`update`, `rect`)  
* `pygame.sprite.Group` for batch `update`/`draw`  
* Collision helpers (`spritecollide`, `Rect.colliderect`)  
* Simple score keeping with `pygame.font.Font`

---

## 2 · Guided steps
1. **Refactor to classes**

   ```python
   class Paddle(pygame.sprite.Sprite):
       def __init__(self, x, y):
           super().__init__()
           self.image = pygame.Surface((10, 100))
           self.image.fill("white")
           self.rect  = self.image.get_rect(midleft=(x, y))

       def update(self, up_key, down_key):
           keys = pygame.key.get_pressed()
           if keys[up_key]   and self.rect.top > 0:     self.rect.y -= 6
           if keys[down_key] and self.rect.bottom < 600: self.rect.y += 6
   ```

2. **Ball sprite with collision**

   ```python
   class Ball(pygame.sprite.Sprite):
       def __init__(self):
           super().__init__()
           self.image = pygame.Surface((20, 20), pygame.SRCALPHA)
           pygame.draw.circle(self.image, "white", (10, 10), 10)
           self.rect  = self.image.get_rect(center=(400, 300))
           self.vx, self.vy = 4, 4

       def update(self):
           self.rect.x += self.vx
           self.rect.y += self.vy
           if self.rect.top <= 0 or self.rect.bottom >= 600:
               self.vy *= -1
   ```

3. **Group setup**

   ```python
   all_sprites = pygame.sprite.Group()
   paddles     = pygame.sprite.Group()

   left = Paddle(30, 300)
   right = Paddle(770, 300)
   ball = Ball()

   all_sprites.add(left, right, ball)
   paddles.add(left, right)
   ```

4. **Bounce on paddle**

   ```python
   if pygame.sprite.spritecollide(ball, paddles, False):
       ball.vx *= -1
   ```

5. **Scoring**

   ```python
   font = pygame.font.Font(None, 60)
   score_l = score_r = 0

   # when ball.rect.left <= 0: score_r += 1; reset ball
   ```

---

## 3 · Checkpoint

```bash
python3 pong.py               # full Pong matches playable to 10 pts
git add pong.py
git commit -m "Lesson 2.2: Pong completed with sprites and scoring"
```

# Lesson 2.3 – Images & Animation Sheets  
*Project slice: polish Pong visuals*

---

## 1 · Theory bites
* Loading images (`pygame.image.load`) & converting (`convert_alpha`)  
* Sprite sheets: slicing frames, iterating for animation  
* `blit()` transparent PNGs over background  

---

## 2 · Guided steps
1. **Assets folder**

   ```
   assets/
   ├─ paddles.png          # 3-frame vertical sheet (idle, swing, glow)
   ├─ ball.png
   └─ bg.png
   ```

2. **Sprite-sheet helper**

   ```python
   def frames_from_sheet(path, w, h):
       sheet = pygame.image.load(path).convert_alpha()
       return [sheet.subsurface(pygame.Rect(i*w, 0, w, h)) for i in range(sheet.get_width()//w)]
   ```

3. **Animate paddle hit**

   ```python
   class Paddle(...):
       frames = frames_from_sheet("assets/paddles.png", 10, 100)

       def __init__(...):
           ...
           self.frame_idx = 0

       def animate(self, hitting: bool):
           self.frame_idx = 1 if hitting else 0
           self.image = self.frames[self.frame_idx]
   ```

4. **Background image**

   ```python
   bg = pygame.image.load("assets/bg.png").convert()
   ...
   screen.blit(bg, (0, 0))
   all_sprites.draw(screen)
   ```

---

## 3 · Checkpoint

```bash
python3 pong.py
# paddles use images; brief glow on collision; background rendered
git add assets/ pong.py
git commit -m "Lesson 2.3: Added graphics and simple animation"
```

# Lesson 2.4 – Sound Mixer, Music & Menus  
*Project slice: title screen, SFX, background music*

---

## 1 · Theory bites
* `pygame.mixer.init()` latency vs. quality params  
* Loading / playing sounds (`Sound`, `Channel`)  
* Looping background music (`mixer.music`)  
* Simple **state machine** for Menu → Playing → Game-Over

---

## 2 · Guided steps
1. **Sounds folder**

   ```
   sfx/
   ├─ hit.wav
   ├─ wall.wav
   └─ score.wav
   music/
   └─ loop.ogg
   ```

2. **Init audio**

   ```python
   pygame.mixer.init(frequency=44100, buffer=512)
   hit_sfx   = pygame.mixer.Sound("sfx/hit.wav")
   wall_sfx  = pygame.mixer.Sound("sfx/wall.wav")
   score_sfx = pygame.mixer.Sound("sfx/score.wav")
   pygame.mixer.music.load("music/loop.ogg")
   pygame.mixer.music.play(-1)          # loop forever
   ```

3. **Play sounds in logic**

   ```python
   if ball.rect.colliderect(pad.rect):
       hit_sfx.play()
   if ball.rect.top <= 0 or ball.rect.bottom >= HEIGHT:
       wall_sfx.play()
   if scored:
       score_sfx.play()
   ```

4. **State machine**

   ```python
   state = "menu"

   while True:
       if state == "menu":
           if any(event.type == pygame.KEYDOWN for event in events):
               state = "play"
       elif state == "play":
           ...
           if winner:
               state = "gameover"
       elif state == "gameover":
           ...
   ```

5. **Title & game-over screens** – render big fonts centred.

---

## 3 · Checkpoint

```bash
python3 pong.py
# title screen, gameplay with sounds, game-over restart
git add pong.py sfx/ music/
git commit -m "Lesson 2.4: Sounds, music, and menu state machine"
```

# Mini-Capstone B – Polished Pong & Web Build  

**Goal:** refactor your Pong into a clean package and publish it to the web via Pyodide (e.g., with *pygbag*).

---

## Requirements
1. **Package structure**

   ```
   pong/
   ├─ __init__.py
   ├─ game.py            # Game class (state machine)
   ├─ sprites.py         # Paddle, Ball, etc.
   ├─ assets/            # images & sounds
   └─ config.py          # constants
   ```

2. **Clean APIs** – `Game.run()` entry; no globals except `CONFIG`.  
3. **PEP 8** compliance (`ruff` or `flake8` clean).  
4. **Web build**  
   * Install `pip install pygbag`  
   * `pygbag --build pong/game.py` → generates `build/`  
   * Push `build/` to **gh-pages** branch → live demo on GitHub Pages.  
5. **README** with gameplay GIF, controls, and link to web demo.  
6. **CI** – GitHub Actions workflow that lints code and (optionally) uploads new web build on push to `main`.

---

## Deliverable checklist
* [ ] `python3 -m pong.game` launches desktop version at 60 FPS  
* [ ] Web build playable in browser (tested on Firefox/Chrome)  
* [ ] Repo root contains `LICENSE`, `README.md`, and `requirements.txt`  
* [ ] Passing CI badge in README  

```bash
git add pong/ .github/workflows/
git commit -m "Mini-Capstone B: Polished Pong with web build"
git tag -a "module-2-complete" -m "Finished Module 2"
```

🎉 **Module 2 complete!** You now understand the full pygame event loop, collision handling, asset pipelines, sound, and basic deployment. Next up: Module 3 – scrolling platformers and more advanced architecture.

