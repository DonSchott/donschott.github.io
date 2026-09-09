---
layout: post
title: Let pi, opencode and goose write the same Snake game, fueled by qwen3-coder on a DGX Spark
date: 2026-09-08 10:00:00
description: "An experiment: can opencode, pi and goose program the game Snake powered only by qwen3-coder:30b running locally on a DGX Spark?"
tags: llm local-llm coding-agents
categories: ai
giscus_comments: false
related_posts: false
toc:
  sidebar: left
---

I installed opencode, pi and goose on a DGX Spark machine to run an experiment: can they program the game Snake just being powered by the local model qwen3-coder:30b running on the machine? I know Claude Code or OpenAI Codex using API tokens could. Can the local setup compete?

**Model:** qwen3-coder:30b, 64k context, served with ollama
**Machine:** DGX Spark (GB10), 128 GB unified memory
**Harnesses:** pi v0.85.1, opencode v1.18.25, goose v1.48.0

{% include figure.liquid path="assets/img/2026-09-08-snake-harnesses/hero-result.png" class="img-fluid rounded z-depth-1" width="280" alt="Terminal snake game, dashed border, snake seven segments long, score 10" %}
<div class="caption">
    One of the twelve results: snake.py written by qwen3-coder:30b in 29 seconds, then played by me in the terminal.
</div>

## Intro and background

The year 2026 is the year of the harness. The focus shifts from what model to use or how to write smart prompts. The layer around the model gets more and more relevant. Models that have sucked up all knowledge of the internet but do not know how to apply it in the real world fade. Smaller models that are talented in tool use are useful in solving problems. Why memorise a fact when you can look it up?

This usefulness was the magic I felt when using Anthropic Claude AI Sonnet and Opus from December 2025. The tool use was the secret ingredient. Models I tried before just guessed code that may or may not run. This autocomplete was still interesting but not very useful as a programmer. From the end of 2025 on, Claude was able to write, run and debug its own code. In early versions you could see exactly what it did, what unix commands it chained, how it looped thinking, writing and fixing bugs to finally get a working program on the first shot without me babysitting and handholding.

Since then open and local models made big steps forward. The models you can run on your machine still lag behind the frontier models, but the gap is surprisingly slim. The tooling also improved massively in the meantime. So there are now many options for running local models on your machine that give a similar user experience to what Claude Code does when burning tokens via API calls.

So this post will be about trying out three different harnesses that I found suitable for programming: opencode, pi and goose. I tried a few others as well and also wanted to include Claude Code in this benchmark. Yet I struggled to get Claude Code running with local models served by ollama. In theory it works, but in practice I could not stop Claude Code burning a massive number of tokens before starting to code at all. The examples I ran meant opencode was done writing the code after about a minute; the same prompt given to Claude Code left me waiting for 10 minutes already at 100% GPU use with no work done and no end in sight. After an hour or more of tinkering I dropped Claude Code. Setting up opencode, pi and goose was easy. Setting up Claude Code to run with local models is surprisingly hard. After all, just use Claude Code with their Claude Pro, Claude Max or API call pricing. For local models there are better options.

{% include figure.liquid path="assets/img/2026-09-08-snake-harnesses/tui-pi.png" class="img-fluid rounded z-depth-1" alt="pi terminal user interface at startup" %}
{% include figure.liquid path="assets/img/2026-09-08-snake-harnesses/tui-opencode.png" class="img-fluid rounded z-depth-1" alt="opencode terminal user interface at startup" %}
{% include figure.liquid path="assets/img/2026-09-08-snake-harnesses/tui-goose.png" class="img-fluid rounded z-depth-1" alt="goose terminal user interface starting a new session" %}
<div class="caption">
    The three contenders at startup: pi, opencode, goose. All three took minutes to install and point at a local ollama model.
</div>

## Idea and experiment setting

How to compare the harnesses? Simon Willison famously loves to let models draw pelicans on bicycles. These harnesses are not about drawing slop but about (agentic) programming. I came up with the game that was included in the first mobile phone I owned: snake. I loved that game as a kid and it is a sweet spot between being trivially easy or taking forever. To be a fair match, I wrote a tight spec in the prompt: same board size, same tech, just one Python file, use of the `curses` package, etc. Otherwise, different results would have been hard to compare quantitatively when there would be too much freedom.

I ran the experiments on a DGX Spark from Nvidia/Dell that I access remotely with ssh and Tailscale. It is a lovely machine for using local models given its 128GB unified RAM, 20 cores and a Blackwell GPU. I used the qwen3-coder:30b model with 64k context served with ollama for the benchmark to keep things simple. The harnesses came in versions: pi v0.85.1, opencode v1.18.25 and goose v1.48.0. For the experiment I created a new non-admin user, so the agents could run wild if they wanted to without doing much harm. Not perfect, but a simpler solution than docker containers or sandboxing.

The easiest ways to compare performance that came to mind: how long did it take? Does the code run (flawlessly)? It is a simple experiment with a simple metric. More sophisticated metrics can be done, but not for now.

Just one run would be arbitrary and we know that LLMs are governed by stochastic processes. So I stopped at four runs per harness setting, as a compromise between noise and patience. I measured time as wall-clock time around the process. After each run I ran `python3 snake.py` in the terminal and judged by eye whether the game worked. Additionally, for some runs with pi and opencode I kept the JSON event stream and extracted some insights such as token counts, tool calls and tool errors afterwards. E.g. when a run needed double the time and still failed, this is what Sherlock Holmes would do to find out what happened.

In the first few tries I opened the harness with e.g. `pi` manually, pasted the prompt and viewed the inner workings with my own eyes. But then I automated: each run just at stock defaults, non-interactive, one line of bash command. Keep in mind the model qwen3-coder:30b was set as default before.

```bash
pi --mode json -a @PROMPT.md > events.jsonl
opencode run --format json --model ollama/qwen3-coder:30b "$(cat PROMPT.md)" > events.jsonl
goose run -i PROMPT.md
```

## Results

| Harness | Run times (s) | Median | Runs correctly |
|---|---|---|---|
| pi | 28, 29, 38, 164 | **33.5** | 4/4 |
| opencode | 24, 32, 37, 73 | **34.5** | 4/4 |
| goose | 56, 70, 110, 178 | 90 | 2/4 |
{: .table .table-sm .table-bordered .table-hover}

<div class="caption">
    Four runs per harness, stock defaults, same prompt and model.
</div>

All harnesses were fun to work with. Each is mature software. Each has its own flavour. In terms of performance, pi and opencode were the winners. But to be fair, I did not change the defaults. For instance, opencode is known for having great defaults and runs “out of the box”; the philosophy of pi is tinkering. So there is lots of potential for improvement. I did not take the time to alter the defaults. I would not rule out goose — changing some defaults might change the speed and effectiveness.

For reasons not completely clear to my old self, I first ran pi with a set of flags that disabled its extensions, skills and context files. The core tools were still there (it could write, read, edit and run bash), but everything pi ships with on top of that was gone. With this, pi had a much harder time producing a correct snake program in short time. Those extras are a large part of what a harness is, so I corrected the command and set it back to defaults. Still an interesting result: stripping the defaults cost a 7x slowdown and made the results worse as well:

| pi configuration | Run times (s) | Median | Runs correctly |
|---|---|---|---|
| defaults | 28, 29, 38, 164 | **33.5** | 4/4 |
| `--no-session -nc --no-extensions --no-skills` | 108, 187, 298, >300 | 243 | 2/4 |
{: .table .table-sm .table-bordered .table-hover}

<div class="caption">
    Same harness, same model, same prompt. Only the flags differ.
</div>

At first those flags looked like benchmark hygiene, and in the first round they made opencode look 6x faster than pi. It looked like a finding: opencode > pi. Hurrah! Yet, this was measuring me — not the harnesses.

Another insight was the variance of the runs, with heavy outliers in terms of runtime and code quality. In opencode run #4 it spent 48s on an edit where `oldString == newString`, thus wasting 69% of the output tokens produced for nothing. In pi run #4, instead of following the spec to write just one file `snake.py`, it wrote to a relative path and created a stray folder tree. Then it had two edit calls rejected as malformed, hit a stream error with auto-retry, and ran a `wc` that failed because the file was not where it thought it was — before finally noticing, fixing the path and cleaning up. In total, an odyssey of 164 seconds instead of 30 seconds.

I also kept an eye on the GPU use and the tokens per second. Decode held steady at around 60–65 tok/s across every run. So whatever the time differences per run, they resulted from the harness, not the model or the DGX Spark machine.

So the time reflected the decisions the agent made along the way. The optimal path was around 24 seconds, but doing something wrong that had to be fixed meant extra time, and explains the variance in runtime and result quality.

<div class="row">
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/2026-09-08-snake-harnesses/board-1.png" class="img-fluid rounded z-depth-1" alt="Snake at start, dashed border, three segments" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/2026-09-08-snake-harnesses/board-2.png" class="img-fluid rounded z-depth-1" alt="Snake with a hash-character border" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/2026-09-08-snake-harnesses/board-3.png" class="img-fluid rounded z-depth-1" alt="Snake drawn with a blue head glyph" %}
    </div>
</div>
<div class="caption">
    Same spec, three readings of it. The border is dashes here and hashes there, the head is sometimes a capital O and sometimes coloured, and the playfield is not always the same size. All three of these count as “runs correctly”.
</div>

Speaking of result quality: several times the model claimed the spec was met without checking. And this happened under all three harnesses, so it is the model's behaviour, not the tooling's. Either without measuring at all it stated “under 140 lines (actually around 80 lines)”. Or it ran `wc -l`, the result was higher than 140, yet it still marked the spec fulfilled: “under 140 lines (actually 146 lines, which is acceptable)”. In real software engineering projects this behaviour must be controlled by review agents or other means.

{% include figure.liquid path="assets/img/2026-09-08-snake-harnesses/gameover.png" class="img-fluid rounded z-depth-1" width="280" alt="Game over screen showing final score 3" %}

## Advice

Can I recommend the harnesses I tested for running with a local LLM on hardware you own? Definitely. For small and medium programming tasks this is already helpful and will only get better. The local LLMs improve at a rapid pace, and the development of pi, opencode and goose is impressive too. The set up was straightforward and only took a few minutes — I expected much more tinkering. We have two winners with similar runtime and result correctness: opencode and pi. Opencode is a good general choice, pi for its minimalism and extensibility, and goose as well (especially when scaling up to bigger projects with several agents running in parallel).

<details markdown="1">
<summary>The full prompt (PROMPT.md)</summary>

```markdown
# Snake

Write a Snake game as a single file `snake.py`.

## Environment

- `python3`, standard library only. You cannot install packages.
- This shell has no TTY. `python3 snake.py` will fail with a curses error, `pty.openpty()`
  raises `PermissionError`, and `script` cannot allocate a pseudo-terminal. That is the
  sandbox, not your code. Do not run the game and do not try to work around this.
- There is no test to run. Get it right by reading the spec carefully and desk-checking
  your own code before you finish.

## Geometry

Playfield 30 wide, 20 tall. Interior cells are `x` in `range(30)`, `y` in `range(20)`,
`(0, 0)` top left. Directions are `(dx, dy)`: right `(1, 0)`, left `(-1, 0)`, up
`(0, -1)`, down `(0, 1)`. A snake is a list of `(x, y)` tuples, head at index 0.

On screen, interior cell `(x, y)` is drawn at row `y + 1`, column `x + 1`. The border
occupies row 0, row 21, column 0 and column 31. The score line is row 22. So the game
needs at least 32 columns and 23 rows.

## Required module-level functions

Pure, no curses, importable and callable without a terminal. `main()` must use them
rather than reimplementing their logic inline.

### `apply_turn(current, requested)`

Return `current` if `requested` is the exact opposite of `current`, otherwise
`requested`.

### `step(snake, direction, food, width, height)`

Return `(new_snake, ate, alive)`.

- `new_head` is the head plus `direction`.
- If `new_head` is outside the playfield, return `(snake, False, False)`.
- `ate` is `True` when `new_head == food`.
- Self-collision depends on whether the snake is eating. When it is not eating the tail
  vacates its cell on this tick, so moving the head into the current tail cell is legal.
  When it is eating the tail stays put, so the same move is fatal. Test `new_head`
  against `snake` when eating and against `snake[:-1]` when not.
- On any collision return `(snake, False, False)`.
- Otherwise `new_snake` is `new_head` prepended to `snake` when eating, or to
  `snake[:-1]` when not. Return `(new_snake, ate, True)`.
- Do not mutate the list you were given. Do not assume 30x20; use `width` and `height`.

### `spawn_food(snake, width, height, rng)`

`rng` is a `random.Random` instance. Use it for all randomness and do not touch the
`random` module's global stream. Return an `(x, y)` cell inside the playfield not
occupied by the snake, or `None` if no free cell exists.

## Structure

- All curses calls live inside `main()` and anything `main()` calls.
- `main()` is invoked only from an `if __name__ == "__main__":` guard at the bottom.
- Importing `snake` must not call any curses function, draw anything, or read input.

## Game

- Border-drawn playfield using `curses`, runnable as `python3 snake.py`.
- If the terminal is smaller than 32x23, print a message and exit cleanly.
- Snake starts length 3, horizontal, centred, moving right.
- Arrow keys steer. A 180 degree reversal is ignored, not fatal.
- Exactly one food on screen. Eating grows the snake by 1 and adds 1 to the score.
- Wall or self collision ends the game: show the final score, wait for a keypress, exit.
- Fixed tick of 8 moves per second. Input polling must not stall the tick.
- Score visible during play. `q` quits at any time.
- The terminal is restored on every exit path, including an unhandled exception.

## Scope

Implement exactly the above. No menu, pause, difficulty, speed ramp, high scores, save
file, colours, or CLI arguments. No type hints, no docstrings, at most a couple of
comments and only where the code is genuinely non-obvious. No tests of your own, no
README, no packaging or dependency files. Under 140 lines.

Do not create, modify, or delete any file other than `snake.py`.

## When you are done

Say so plainly. Do not claim you tested or ran anything, because you cannot. If you are
unsure about part of the spec, say which part rather than guessing silently.
```

</details>

<details markdown="1">
<summary>One result: snake.py from a pi run (146 lines)</summary>

This is the file behind the “under 140 lines (actually 146 lines, which is acceptable)” claim. It plays fine. It is also 146 lines long, it carries docstrings the spec asked me not to have, and lines 56 and 57 hold the tick that is not fixed at 8 moves per second.

```python
import random
import curses

def apply_turn(current, requested):
    """Return current if requested is the exact opposite of current, otherwise requested."""
    if (current[0] == -requested[0] and current[1] == -requested[1]):
        return current
    return requested

def step(snake, direction, food, width, height):
    """Return (new_snake, ate, alive)."""
    head = snake[0]
    new_head = (head[0] + direction[0], head[1] + direction[1])

    # Check if out of bounds
    if (new_head[0] < 0 or new_head[0] >= width or
        new_head[1] < 0 or new_head[1] >= height):
        return (snake, False, False)

    # Check for self collision
    if new_head == food:
        ate = True
        # Snake eats, so tail doesn't move
        check_snake = snake
    else:
        ate = False
        # Tail moves, check against snake[:-1]
        check_snake = snake[:-1]

    # Check for collision with self
    if new_head in check_snake:
        return (snake, False, False)

    # Create new snake
    if ate:
        new_snake = [new_head] + snake
    else:
        new_snake = [new_head] + snake[:-1]

    return (new_snake, ate, True)

def spawn_food(snake, width, height, rng):
    """Return an (x, y) cell inside the playfield not occupied by the snake, or None if no free cell exists."""
    # Create list of all possible positions
    all_positions = [(x, y) for x in range(width) for y in range(height)]

    # Remove snake positions
    free_positions = [pos for pos in all_positions if pos not in snake]

    # Return random free position or None if no free position exists
    return rng.choice(free_positions) if free_positions else None

def main(stdscr):
    # Initialize curses
    curses.curs_set(0)
    stdscr.nodelay(True)
    stdscr.timeout(125)  # 125ms for 8 updates per second

    # Check terminal size
    height, width = stdscr.getmaxyx()
    if width < 32 or height < 23:
        stdscr.clear()
        stdscr.addstr(0, 0, "Terminal too small. Need at least 32x23.")
        stdscr.refresh()
        stdscr.getch()
        return

    # Initialize game state
    game_width = 30
    game_height = 20

    # Snake starts length 3, horizontal, centered, moving right
    snake = [(14, 9), (13, 9), (12, 9)]
    direction = (1, 0)  # Moving right
    score = 0

    # Seed random number generator
    rng = random.Random()

    # Initialize food
    food = spawn_food(snake, game_width, game_height, rng)
    if food is None:
        return  # No space for food, game over

    # Main game loop
    while True:
        # Handle input
        key = stdscr.getch()
        if key == ord('q'):
            break
        elif key == curses.KEY_UP:
            direction = apply_turn(direction, (0, -1))
        elif key == curses.KEY_DOWN:
            direction = apply_turn(direction, (0, 1))
        elif key == curses.KEY_LEFT:
            direction = apply_turn(direction, (-1, 0))
        elif key == curses.KEY_RIGHT:
            direction = apply_turn(direction, (1, 0))

        # Update game state
        snake, ate, alive = step(snake, direction, food, game_width, game_height)

        if not alive:
            break

        if ate:
            score += 1
            # Spawn new food
            food = spawn_food(snake, game_width, game_height, rng)
            if food is None:
                break  # No space for food, game over

        # Draw everything
        stdscr.clear()

        # Draw border
        for x in range(game_width + 2):
            stdscr.addch(0, x, '-')
            stdscr.addch(game_height + 1, x, '-')
        for y in range(game_height + 2):
            stdscr.addch(y, 0, '|')
            stdscr.addch(y, game_width + 1, '|')

        # Draw snake
        for i, (x, y) in enumerate(snake):
            if i == 0:  # Head
                stdscr.addch(y + 1, x + 1, 'O')
            else:  # Body
                stdscr.addch(y + 1, x + 1, 'o')

        # Draw food
        stdscr.addch(food[1] + 1, food[0] + 1, '*')

        # Draw score
        stdscr.addstr(game_height + 2, 0, f"Score: {score}")

        stdscr.refresh()

    # Show final score and wait for keypress
    stdscr.clear()
    stdscr.addstr(0, 0, f"Game Over! Final Score: {score}")
    stdscr.addstr(1, 0, "Press any key to exit...")
    stdscr.refresh()
    stdscr.getch()

if __name__ == "__main__":
    curses.wrapper(main)
```

</details>
