# Observation and Reward

## Observation (`obs`)

The environment returns a dictionary like:

```json
{
  "ram":  {"x":200.0,"y":940.0,"vx":0.0,"vy":0.0,"health":100},
  "rak":  {"x":1650.0,"y":940.0,"vx":0.0,"vy":0.0,"health":100},
  "stones": 0,
  "player_arrows": 0
}
```

In Python, this can be flattened into a fixed vector:

```
[ram_x, ram_y, ram_vx, ram_vy, ram_health,
 rak_x, rak_y, rak_vx, rak_vy, rak_health,
 stones, player_arrows]
```

## Reward

Reward encourages damaging Ram and penalizes taking damage:

- **+0.5 × (damage to Ram during the step)**
- **−1.0 × (damage to Rakshasa during the step)**

This makes the optimization objective:
- attack effectively (increase Ram damage)
- survive (avoid Rakshasa damage)

## Episode end (`done`)

An episode ends when either character reaches 0 health:

- `done = (ram_health <= 0) OR (rak_health <= 0)`

