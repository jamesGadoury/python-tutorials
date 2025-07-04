# Lesson 1.1 – Data Types & Basic I/O  
*Game: Guess-the-Number*

---

## 1 · Theory bites
* Integers, floats, strings, booleans  
* `input()` and `print()` with f-strings  
* `random.randint()`  
* Basic `while` loop

---

## 2 · Guided steps
1. **Prompt for player name**; greet them.  
2. **Generate secret number** between 1–100.  
3. **Loop** until the guess is correct, giving **higher/lower** hints.  
4. **Count attempts** and congratulate the player.

```python
import random

name = input("👋  What is your name? ")
secret = random.randint(1, 100)
attempts = 0

while True:
    guess = int(input(f"{name}, take a guess (1-100): "))
    attempts += 1
    if guess < secret:
        print("Higher ↑")
    elif guess > secret:
        print("Lower ↓")
    else:
        print(f"🎉  Correct in {attempts} attempt(s)!")
        break
```

---

## 3 · Checkpoint

```bash
python3 guess_number.py
git add guess_number.py
git commit -m "Lesson 1.1: Guess-the-Number"
```

# Lesson 1.2 – Loops, Conditionals & Enums  
*Game: Rock-Paper-Scissors (best-of-three)*

---

## 1 · Theory bites
* `for` vs `while` loops  
* `if / elif / else` decision chains  
* `enum.Enum` for tidy constants  
* Doctest basics (`python3 -m doctest`)

---

## 2 · Guided steps
1. **Define choices**

   ```python
   from enum import Enum

   class Choice(Enum):
       ROCK     = "rock"
       PAPER    = "paper"
       SCISSORS = "scissors"
   ```

2. **Function `beats(a, b) → bool`**

   ```python
   def beats(a: Choice, b: Choice) -> bool:
       return (
           (a is Choice.ROCK     and b is Choice.SCISSORS) or
           (a is Choice.PAPER    and b is Choice.ROCK)     or
           (a is Choice.SCISSORS and b is Choice.PAPER)
       )
   ```

3. **Main loop**  
   * Prompt user until a valid choice.  
   * Computer picks randomly (`random.choice(list(Choice))`).  
   * Announce winner; update scores.  
   * First to **two** wins takes the match.

4. **Doctests** – append a truth-table check:

   ```python
   """
   >>> from rps import Choice, beats
   >>> beats(Choice.ROCK, Choice.SCISSORS)
   True
   >>> beats(Choice.PAPER, Choice.SCISSORS)
   False
   """
   ```

---

## 3 · Checkpoint

```bash
python3 -m doctest rps.py   # no output ⇒ all tests pass
python3 rps.py              # play the game
git add rps.py
git commit -m "Lesson 1.2: Rock-Paper-Scissors with doctests"
```

# Lesson 1.3 – Functions, Modules & Unit Testing  
*Project: Mad-Libs Generator*

---

## 1 · Theory bites
* Writing and calling functions  
* Splitting code into modules (`lib_io.py`, `mad_libs.py`)  
* `pytest` for automated tests  
* Docstrings & type hints (`-> str`)

---

## 2 · Guided steps
1. **Template file** – place a story with placeholders such as  
   `A {adjective} {noun} jumped over the {object}.` in `templates/story.txt`.

2. **Function `fill_template(path) → str`**

   ```python
   import re, pathlib

   def fill_template(path: pathlib.Path) -> str:
       text = path.read_text()
       keys = re.findall(r"{(.*?)}", text)
       answers = {k: input(f"Give me a {k}: ") for k in keys}
       return text.format(**answers)
   ```

3. **Replay loop** – after printing the story ask “Play again? (y/n)”.

---

## 3 · Unit tests (`tests/test_fill_template.py`)

```python
from mad_libs import fill_template

def test_fill(monkeypatch, tmp_path):
    tpl = tmp_path / "tpl.txt"
    tpl.write_text("The {animal} sat on the {object}.")
    inputs = iter(["cat", "mat"])
    monkeypatch.setattr("builtins.input", lambda _: next(inputs))
    out = fill_template(tpl)
    assert out == "The cat sat on the mat."
```

```bash
pytest -q
```

---

## 4 · Checkpoint

```bash
python3 mad_libs.py
pytest -q
git add mad_libs.py lib_io.py tests/
git commit -m "Lesson 1.3: Mad-Libs with pytest"
```

# Lesson 1.4 – Collections & Algorithmic Thinking  
*Game: Hangman (CLI)*

---

## 1 · Theory bites
* **Lists** and slicing  
* **Sets** for O(1) membership checks  
* **Dictionaries** for config / ASCII art  
* Basic terminal graphics (ANSI)

---

## 2 · Guided steps
1. **Word source**

   ```python
   import random, pathlib
   WORDS = pathlib.Path("words.txt").read_text().split()
   word = random.choice(WORDS).lower()
   ```

2. **Game state**

   ```python
   lives = 6
   guessed: set[str] = set()
   ```

3. **Main loop**

   ```python
   while lives and not set(word) <= guessed:
       masked = " ".join(c if c in guessed else "_" for c in word)
       print(f"\nLives {lives} | {masked}")
       guess = input("Letter: ").lower()

       if len(guess) != 1 or not guess.isalpha():
           print("Enter a single letter.")
           continue
       if guess in guessed:
           print("Already guessed.")
           continue

       guessed.add(guess)
       if guess not in word:
           lives -= 1
   ```

4. **Win / lose** – reveal the word and print final gallows stage.

---

## 3 · Checkpoint

```bash
python3 hangman.py
git add hangman.py words.txt
git commit -m "Lesson 1.4: Hangman complete"
```

# Mini-Capstone A – ASCII Brick-Breaker  

Build a terminal Brick-Breaker that runs at ≈ 15 FPS using `curses` **or** pure ANSI escapes.

---

## Requirements
1. **Grid render** – at least 20 × 40 characters  
2. **Entities**  
   * `Ball` (`x, y, vx, vy`)  
   * `Paddle` (arrow-key control)  
   * `Brick` list; remove on collision  
3. **Physics** – bounce ball on walls, paddle, bricks  
4. **UI** – score, lives, game-over screen

---

## Suggested structure
```
brick_breaker/
├─ game.py          # main loop & rendering
├─ models.py        # Ball, Paddle, Brick classes
├─ utils.py         # collision helpers
└─ README.md        # run instructions + screenshot
```

---

## Deliverable
* `python3 game.py` launches and plays  
* Paddle responds to ← / → keys  
* Bricks disappear; score updates  
* Lose life when ball falls; game over when lives = 0  
* README shows controls and an ASCII frame

```bash
git add brick_breaker/
git commit -m "Mini-Capstone A: ASCII Brick-Breaker"
git tag -a "module-1-complete" -m "Finished Module 1"
```

**Next:** Module 2 – bringing these mechanics to life with `pygame`.

