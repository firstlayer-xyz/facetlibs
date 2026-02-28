# Gears Library — Reference

Generates involute spur gears with standard or custom pressure angles.

## Loading

```
var G = lib "facet/gears";
```

## Gear Struct

```
struct Gear {
    Number teeth;           # Number of teeth
    Length module;          # Module (pitch / π) — controls tooth size and gear spacing
    Angle  pressure_angle;  # Pressure angle (standard: 20°)
}
```

The **module** `m` determines overall gear scale. Two meshing gears must have the same module. Common values: `1 mm`, `1.5 mm`, `2 mm`, `2.5 mm`, `3 mm`.

## Constructor

| Function | Returns | Description |
|----------|---------|-------------|
| `NewGear(Number teeth, Length module)` | `Gear` | Standard 20° pressure angle |
| `NewGear(Number teeth, Length module, Angle pressure_angle)` | `Gear` | Custom pressure angle |

## Methods

### Geometry Queries

| Method | Returns | Description |
|--------|---------|-------------|
| `Gear.PitchRadius()` | `Length` | `m × N / 2` — radius of the pitch circle |
| `Gear.BaseRadius()` | `Length` | `r_p × cos(α)` — involute base circle radius |
| `Gear.AddendumRadius()` | `Length` | `r_p + m` — outer tip radius |
| `Gear.DedendumRadius()` | `Length` | `r_p − 1.25m` — root (valley) radius |

### Solid Generation

| Method | Returns | Description |
|--------|---------|-------------|
| `Gear.Profile(Number n_involute = 20)` | `Sketch` | 2D involute profile. Higher `n_involute` = smoother tooth flanks. |
| `Gear.Spur(Length width)` | `Solid` | Extruded spur gear of given face width. |

## Meshing Two Gears

Two gears mesh when they share the same **module** and **pressure angle**. The center distance between their axes is:

```
center_distance = (G1.PitchRadius() + G2.PitchRadius())
```

Speed ratio = teeth₂ / teeth₁.

## Examples

### Single spur gear

```
var G = lib "facet/gears";

Main() {
    G.NewGear(24, 2 mm).Spur(8 mm)
}
```

### Meshing gear pair

```
var G = lib "facet/gears";

Main() {
    var g1 = G.NewGear(12, 2 mm);
    var g2 = G.NewGear(36, 2 mm);

    var cd = g1.PitchRadius() + g2.PitchRadius();

    var gear1 = g1.Spur(8 mm);
    var gear2 = g2.Spur(8 mm).Translate(cd, 0 mm, 0 mm);

    gear1 + gear2
}
```

### Custom pressure angle

```
var G = lib "facet/gears";

Main() {
    G.NewGear(20, 2 mm, 25 deg).Spur(10 mm)
}
```

### High-resolution profile

```
var G = lib "facet/gears";

Main() {
    G.NewGear(16, 2 mm).Profile(40).Extrude(6 mm)
}
```

## Gear Geometry Reference

```
        ← Addendum radius (tip)
       ╱‾‾‾‾‾╲
      ╱        ╲       ← Pitch radius (reference circle)
     │   tooth   │
      ╲        ╱       ← Base radius (involute starts here)
       ╲______╱
        ← Dedendum radius (root)
```

| Variable | Formula | Description |
|----------|---------|-------------|
| `r_p` | `m × N / 2` | Pitch radius |
| `r_b` | `r_p × cos(α)` | Base circle radius (involute origin) |
| `r_a` | `r_p + m` | Addendum radius (tooth tip) |
| `r_d` | `r_p − 1.25m` | Dedendum radius (root valley) |

The **involute** tooth profile is traced from the base circle outward. A parameter `t` (radians) traces the involute:

```
x = r_b × (cos(t) + t × sin(t))
y = r_b × (sin(t) − t × cos(t))
```

## References

- [Involute Gear — Engineers Edge](https://www.engineersedge.com/gears/involute-gear-5996.htm)
- [Module (Gear) — Wikipedia](https://en.wikipedia.org/wiki/Module_(gear))
