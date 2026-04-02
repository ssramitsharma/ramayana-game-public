# Ramayan (Godot) — RL Rakshasa (PPO) Case Study

This is a **public showcase repo** for a private Godot 4 game project where I trained the enemy (**Rakshasa**) using **reinforcement learning** (PPO) and deployed the trained policy back into the live game.

- **Game repo**: private (assets + full project)
- **This repo**: documentation + safe code snippets + reproducible commands

## What I built

- **Headless training environment in Godot** exposed via newline-delimited **JSON** (`reset` / `step`)
- **Python PPO training** (Stable-Baselines3) that learns to control Rakshasa
- **Evaluation script** to measure win-rate / reward stats headlessly
- **Live in-game deployment**: a lightweight **TCP bridge** so Python can drive Rakshasa while the game runs normally

## Architecture (high level)

```mermaid
flowchart LR
  subgraph Godot[Godot 4 Game]
    ENV[Headless RL runner\n(stdin/stdout JSON)]
    LIVE[Live TCP bridge\n(127.0.0.1:5555)]
    SIM[Game simulation\n(Ram scripted, Rakshasa controlled)]
    ENV --> SIM
    LIVE --> SIM
  end

  subgraph Python[Python]
    PPO[PPO policy\n(Stable-Baselines3)]
    EVAL[Evaluation runner]
  end

  PPO <--> ENV
  PPO <--> LIVE
  EVAL <--> ENV
```

## Key interfaces (snippets)

- **Protocol**: see `docs/protocol.md`
- **Observation & reward**: see `docs/obs-and-reward.md`

## Case study (blog)

Read the full write-up:
- `docs/rl-case-study.md`

## Commands I used (from the private repo)

> These assume you have the private repo checked out locally and Godot installed.

### Train headless

```bash
export GODOT_BIN="/Applications/Godot.app/Contents/MacOS/Godot"
TOTAL_STEPS=200000 python "ramayan-godot/training/train_rakshasa.py"
```

### Evaluate

```bash
export GODOT_BIN="/Applications/Godot.app/Contents/MacOS/Godot"
python "ramayan-godot/training/evaluate_rakshasa.py" --episodes 20 --deterministic
```

### Run live (play vs RL Rakshasa)

Terminal 1:

```bash
"/Applications/Godot.app/Contents/MacOS/Godot" --path "/path/to/ramayan-godot" -- --rl
```

Terminal 2:

```bash
python3 "/path/to/ramayan-godot/training/play_rakshasa.py"
```

## Why the full project is private

The full game repo includes assets and project contents that I’m not publishing publicly. This repo is intended to showcase the **engineering + ML integration** clearly enough for review.

