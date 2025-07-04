# Lesson 0.1 – Install **pygame** and open your first window  
*Goal: use `pygame` from an activated virtual environment and display a 640 × 480 game window.*

---

## 1 · Install pygame (Ubuntu 24.04)

~~~bash
sudo apt update
sudo apt install python3-pygame
~~~

> **Verify:**
> ~~~bash
> python3 - <<'PY'
> import pygame, sys
> print("pygame version:", pygame.version.ver)
> PY
> ~~~

You should see something like `pygame version: 2.5.3`.

---

## 2 · “Hello, Pygame” – minimal program

Create **hello_pygame.py** in your project folder:

~~~python
import sys
import pygame

pygame.init()
screen = pygame.display.set_mode((640, 480))
pygame.display.set_caption("Hello, Pygame!")

clock = pygame.time.Clock()

running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    screen.fill("darkslategray")  # RGB tuple also works
    pygame.display.flip()
    clock.tick(60)                # limit to 60 FPS

pygame.quit()
sys.exit()
~~~

Run it:

~~~bash
python3 hello_pygame.py
~~~

You should get a blank window titled **Hello, Pygame!** that closes cleanly.

---

## 3 · Checkpoint

- [ ] `python -m pip list | grep pygame` shows `pygame` installed in **.venv**  
- [ ] The test script prints a pygame version without errors  
- [ ] You can open and close the demo window at 60 FPS

Commit **hello_pygame.py** to your Git repository—we’ll expand it later.

---

# Lesson 0.2 – Tooling Tour (IDE, Debugger, Git)  
*Goal: be comfortable using VS Code (or PyCharm), Git, and the debugger for Python game work.*

---

## 1 · Install VS Code + Python extension

~~~bash
sudo snap install --classic code          # or use .deb from microsoft.com
code --install-extension ms-python.python
code --install-extension ms-toolsai.jupyter
~~~

> **Alternative:** PyCharm Community Edition (`sudo snap install pycharm-community --classic`).

---

## 2 · Open the project

1. **File → Open Folder…** → select your `hello-pygame` folder.  

---

## 3 · First debug session

1. Open **hello_pygame.py**.  
2. Set a breakpoint on `screen.fill(...)`.  
3. Press **F5** → select **Python File**.  
4. Inspect variables in the *Run & Debug* panel, then **Continue** to exit.

---

## 4 · Git basics

~~~bash
# If you haven’t yet:
git init
echo ".venv/" >> .gitignore
git add hello_pygame.py .gitignore
git commit -m "Add hello pygame window (Lesson 0.1)"
~~~

Optional: create a private repo on GitHub and push:

~~~bash
git remote add origin https://github.com/<user>/hello-pygame.git
git branch -M main
git push -u origin main
~~~

---

## 5 · Checkpoint

- [ ] You can launch **hello_pygame.py** under the debugger and hit breakpoints  
- [ ] Git commits show up with your name & email (`git config --global user.name "…"`)  
- [ ] `.venv/` is ignored by Git (`git status` should be clean after commit)

---

## 6 · Looking ahead

With Python 3.12, pygame, and an IDE-debugger pipeline, you’re set for **Module 1**—where we’ll sharpen core Python skills by building a handful of text-based mini-games before returning to graphical work.

> Save your progress: tag the repo  
> ~~~bash
> git tag -a "lesson-0-complete" -m "Environment & tooling ready"
> ~~~

