Nice — this is a very good final project! 🎉
Below I’ll explain **every part and step** in plain, simple terms so you can reuse what you learn and grow with confidence.

# High level (one-sentence)

This program is a simple **Snake** game using Tkinter: it draws a snake and food on a grid, moves the snake each “tick”, 
lets the player change direction with arrow keys, grows the snake and increases score when it eats food, and ends the game when the snake hits a wall or itself.

---

# 1) Imports & constants — what they do

```python
from tkinter import *
import random as R
```

* `tkinter` gives you windows, labels and a drawing canvas.
* `random` (aliased `R`) is used to place food randomly.

Constants (e.g. `GAME_WIDTH`, `SPACE_SIZE`, `SNAKE_COLOR`) define the game settings (screen size, grid cell size, speed, starting body length, and colors).
You use these throughout the program so changing a single constant changes behavior everywhere.

---

# 2) Coordinate system and grid idea

* The canvas origin `(0,0)` is the top-left corner. `x` grows to the right, `y` grows downward.
* `SPACE_SIZE` (50) makes a visible grid: every object position is a multiple of `SPACE_SIZE`. This simplifies movement and collision (snake moves one grid cell at a time).

---

# 3) `Snake` class — what it stores and draws

```python
class Snake:
    def __init__(self):
        self.body_size = BODY_PART
        self.coordinates = []
        self.squares =[]
        ...
```

* `self.coordinates` — list of positions (head first). Example: `[[0,0], [0,0], [0,0]]` initially.
* `self.squares` — list of canvas object IDs returned by `canvas.create_rectangle(...)`. Each ID lets you delete/redraw that square on the canvas.
* The constructor fills `coordinates` with BODY_PART entries and draws rectangles on the canvas, building the initial snake.

Key idea: **the coordinate list and the canvas-ID list are parallel** — index 0 in both refers to the head, last index refers to the tail.

---

# 4) `Food` class — random food placement

```python
class Food:
    def __init__(self):
        x = R.randint(0, int(GAME_WIDTH/SPACE_SIZE)-1) * SPACE_SIZE
        y = R.randint(0, int(GAME_HEIGHT/SPACE_SIZE)-1) * SPACE_SIZE
        self.coordinates = [x, y]
        canvas.create_oval(..., tag="food")
```

* Picks a random grid cell (`x`, `y`) aligned to `SPACE_SIZE`.
* Draws a circular “food” on the canvas and stores its coordinates.
* Using the `"food"` tag makes it easy to delete the food later (`canvas.delete("food")`).

---

# 5) `next_turn(snake, food)` — the game loop tick

This function is called repeatedly (every `SPEED` ms) and does the main work each frame:

1. Read current head coords: `x, y = snake.coordinates[0]`.
2. Move the head according to `direction`:

   * `up` → `y -= SPACE_SIZE`
   * `down` → `y += SPACE_SIZE`
   * `left` → `x -= SPACE_SIZE`
   * `right` → `x += SPACE_SIZE`
3. Insert the new head position at index 0: `snake.coordinates.insert(0, (x, y))`.
4. Draw a new rectangle on canvas for the new head and insert its canvas ID into `snake.squares`.
5. **If the head landed on the food position**:

   * Increase `score`, update the label `label.config(...)`.
   * Remove the food graphics with `canvas.delete("food")`.
   * Create a new `Food()` instance (so the snake grows because we **do not** remove the tail this turn).
6. **Else** (no food eaten):

   * Remove the last coordinate (tail) from `snake.coordinates`.
   * Delete the last rectangle on the canvas with `canvas.delete(snake.squares[-1])` and remove its ID from `snake.squares`. This creates the visual effect of moving without growing.
7. Check for collisions: `if check_collision(snake): game_over()` — stop the loop & show game over.
8. If no collision: schedule the next tick using Tkinter timer:
   `window.after(SPEED, next_turn, snake, food)`
   → this calls `next_turn` again later, creating the repeating game loop.

**Core movement trick:** insert new head + remove tail = movement; insert head but skip removing tail when eating = growth.

---

# 6) `change_direction(new_direction)` — keyboard control

* This function changes the global `direction` variable when arrow keys are pressed.
* It checks the current `direction` to **prevent reversing directly** (e.g., from left to right immediately), because that would make the head collide with the neck.

Bindings in the GUI:

```python
window.bind('<Left>', lambda event: change_direction('left'))
...
```

When the user presses an arrow key, Tkinter calls `change_direction(...)`.

---

# 7) `check_collision(snake)` — when the game should end

Checks three collision types:

1. **Wall collision:** head `x` or `y` is outside `[0, GAME_WIDTH)` or `[0, GAME_HEIGHT)` → return `True`.
2. **Self collision:** iterate `snake.coordinates[1:]` (the body without head) and compare positions; if any equal head → return `True`.
3. Otherwise no collision → function returns `None` (which is falsy). Ideally it should explicitly `return False`, but using `if check_collision(snake):` still works.

If collision detected, `next_turn` calls `game_over()` and does not schedule another tick.

---

# 8) `game_over()` — show end screen

```python
canvas.delete(ALL)
canvas.create_text(..., text="GAME OVER", ...)
```

* Clears the canvas and draws a centered “GAME OVER” message.
* Because `next_turn` did not schedule another `window.after`, the game stops there.

---

# 9) GUI setup and starting the game

* Create Tk window: `window = Tk()`, set title and make it non-resizable.
* Create a `Label` to display the score; use `label.config(...)` to update score later.
* Create a `Canvas` sized by `GAME_WIDTH` x `GAME_HEIGHT` to draw the snake and food.
* Center the window on the screen with `window.geometry(...)`.
* Bind arrow keys to `change_direction`.
* Create initial `snake = Snake()` and `food = Food()`.
* Start the loop with `next_turn(snake, food)`.
* Call `window.mainloop()` which runs Tkinter’s event loop (processing key events and scheduled `after` calls).

---

# 10) Important programming patterns you used — why they matter

* **Separation of data and view:** `Snake` stores coordinates and canvas IDs; Canvas draws visuals. Keeping them synchronized is crucial.
* **Grid alignment:** using `SPACE_SIZE` ensures movement, collision, and food placement align simply.
* **Insert/delete for movement:** the insert-head + delete-tail pattern is the canonical way to make the snake move and grow.
* **Event-driven loop:** `window.after()` + `mainloop()` = non-blocking repeated ticks like a simple game engine.
* **Tags on canvas** - e.g., `"food"` tag makes removing the food easy.

---

# 11) Small issues & tips to improve (next steps)

* `check_collision` should `return False` explicitly at the end for clarity.
* Encapsulate into classes (e.g., a `Game` class) so globals (`score`, `direction`, `canvas`) are not worldwide. This makes the code reusable and testable.
* Save high score to a file (JSON) when game over.
* Add sound effects, level/speed increase, pause/restart, UI buttons.
* Avoid `from tkinter import *` in larger projects — prefer `import tkinter as tk` to prevent namespace pollution.
* Add input validation and friendly messages (e.g., prevent starting direction that immediately collides).

---

# 12) Quick cheat-sheet you can reuse in other projects

* Draw a rectangle: `canvas.create_rectangle(x, y, x+w, y+h, fill=color)`
* Draw an oval: `canvas.create_oval(...)`
* Delete canvas items by id or tag: `canvas.delete(item_or_tag)`
* Schedule repeated call: `window.after(ms, function, *args)`
* Bind keys: `window.bind('<Left>', lambda e: my_func())`
* Random grid cell:
  `x = random.randint(0, (WIDTH/SPACE_SIZE)-1) * SPACE_SIZE`

---

# 13) Mini exercises to grow from here (pick 2–3)

1. Make the snake speed up slightly every 5 points.
2. Prevent food from spawning on the snake body (re-roll location).
3. Add a Restart button that recreates snake & food without closing the window.
4. Save high score to `scores.json`.
5. Refactor into classes: `Game`, `Snake`, `Food` with methods — no global variables.

---
