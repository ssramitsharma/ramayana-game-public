# RL Protocol (Godot ⇄ Python)

The headless environment uses **newline-delimited JSON** over stdin/stdout.

## Commands (Python → Godot)

### Reset

```json
{"cmd":"reset"}
```

### Step

```json
{"cmd":"step","action":{"move":0,"jump":false,"crouch":false,"throw":true},"frames":6}
```

- `action.move`: float in \([-1, 1]\)
- `action.jump`: bool (one-shot)
- `action.crouch`: bool (hold)
- `action.throw`: bool (one-shot, rate-limited in-game)
- `frames`: number of physics frames to advance (clamped, e.g. 1–30)

### Quit

```json
{"cmd":"quit"}
```

## Responses (Godot → Python)

### Reset response

```json
{"obs":{...},"reward":0.0,"done":false,"info":{"reset":true}}
```

### Step response

```json
{"obs":{...},"reward":0.5,"done":false,"info":{"ram_health":95,"rak_health":100}}
```

### Error response

```json
{"error":"bad_json"}
```

