# Training a Godot Enemy with Reinforcement Learning (PPO) — Ramayan “Rakshasa”

I built a 2D archery game in **Godot 4** where you play as **Lord Ram** against a **Rakshasa** opponent. After getting the base gameplay working, I extended the project so Rakshasa can be trained using **reinforcement learning** (PPO) from Python, entirely **headless**, and then deployed back into the live game.

This repository is a **public case study**; the full game repo is private.

## Why this project

Most game AI demos stop at scripted enemies or toy RL environments. I wanted to show an end-to-end integration:

- real Godot gameplay code
- a stable training interface
- training + evaluation in Python
- deployment back into the playable game

## Step 1 — Make Rakshasa controllable

The key design choice was adding a clean “seam” in the Rakshasa controller:

- **Scripted mode**: Rakshasa uses hand-written logic (movement, dodging, shooting).
- **Controlled mode**: external code (RL) sets actions each step.

This is implemented via:
- `set_controlled(true/false)`
- `set_control_action({"move":..,"jump":..,"crouch":..,"throw":..})`

That kept the character implementation stable while allowing a learned policy to drive it.

## Step 2 — Build a headless training API in Godot

To train from Python, I needed deterministic stepping and structured I/O. Godot was run headless with a small stdin/stdout JSON protocol:

- `reset`: spawn the fight scene and return the initial observation
- `step`: apply an action, advance N physics frames, return `(obs, reward, done, info)`

Protocol details are documented in:
- `docs/protocol.md`

### Observation design

The observation intentionally started small:
- positions/velocities/health for Ram and Rakshasa
- counts of active projectiles

This is enough to learn basic movement + attack timing without overfitting to rendering details.

See:
- `docs/obs-and-reward.md`

### Reward design

Reward is computed from health deltas:
- positive when **Ram takes damage**
- negative when **Rakshasa takes damage**

That directly encodes “be dangerous but survive.”

## Step 3 — Train PPO from Python

On the Python side:

- I wrapped the JSON protocol as a Gymnasium environment.
- I trained a PPO agent using Stable-Baselines3.

The result is a saved policy (`.zip`) that can be reloaded for evaluation or live play.

## Step 4 — Evaluate (so we don’t guess)

I added a simple evaluation runner that:
- runs N episodes headlessly
- prints win/loss/timeout counts
- prints reward and episode-length statistics

This made it easy to detect regressions and measure progress.

## Step 5 — Deploy the trained policy into the live game

Godot can’t directly load a Stable-Baselines3 model, so I used a practical deployment approach:

- Godot runs the game normally.
- A lightweight **TCP bridge** streams observations to Python.
- Python predicts actions and sends them back.

This allowed “play the real game vs the trained Rakshasa” with no special build step.

### Quality-of-life fix: don’t lose before the model connects

A real usability issue appeared immediately: by the time Python connected, the match could already be over.

So the bridge was updated to:
- pause the game while waiting for the policy connection
- unpause and restart once Python connects

## Results and next steps

From here, there are several natural improvements:

- richer observations (e.g., projectile trajectories, cooldown timers)
- curriculum learning (start easy, increase difficulty)
- faster training (parallel env instances / faster stepping)
- early stopping based on evaluation win rate

## What this demonstrates (recruiter summary)

- Godot engineering: gameplay + deterministic stepping for simulation
- RL integration: stable interfaces, reward shaping, evaluation discipline
- Deployment: bridging a Python model into a live game loop cleanly

