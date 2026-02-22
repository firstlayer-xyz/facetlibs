# Thread Library — Geometry Reference

## Function Signature

```chisel
var T = lib "chisel/threads";
var thread = T.New("m10");                 # Metric
var thread = T.New("1/4-20");              # SAE UNC
var thread = T.New("1/4-npt");             # NPT
```

## Thread Struct

```chisel
struct Thread {
    Length pitch;            # Axial distance between adjacent thread crests
    Length major_diameter;   # Major (outside) diameter — nominal bolt/pipe OD
    Number taper;            # Diameter growth per unit length (0 = straight, 1/16 = NPT)
}
```

## Methods

| Method              | Returns | Description |
|---------------------|---------|-------------|
| `Thread.Outside(Length length)` | `Solid` | External thread — for bolts, screws, NPT fittings |
| `Thread.Inside(Length length)`  | `Solid` | Internal thread — subtract from nut body or coupling |

## Supported Sizes

### ISO Metric (coarse pitch)

| Size | `major_diameter` | `pitch` |
|------|-----------|---------|
| `"m3"` | 3 mm | 0.5 mm |
| `"m4"` | 4 mm | 0.7 mm |
| `"m5"` | 5 mm | 0.8 mm |
| `"m6"` | 6 mm | 1.0 mm |
| `"m8"` | 8 mm | 1.25 mm |
| `"m10"` | 10 mm | 1.5 mm |
| `"m12"` | 12 mm | 1.75 mm |
| `"m16"` | 16 mm | 2.0 mm |
| `"m20"` | 20 mm | 2.5 mm |

### ISO Metric (fine pitch)

| Size | `major_diameter` | `pitch` |
|------|-----------|---------|
| `"m8x1"` | 8 mm | 1.0 mm |
| `"m10x1"` | 10 mm | 1.0 mm |
| `"m10x1.25"` | 10 mm | 1.25 mm |
| `"m12x1.25"` | 12 mm | 1.25 mm |
| `"m12x1.5"` | 12 mm | 1.5 mm |
| `"m16x1.5"` | 16 mm | 1.5 mm |
| `"m20x1.5"` | 20 mm | 1.5 mm |
| `"m20x2"` | 20 mm | 2.0 mm |

### SAE UNC (Unified National Coarse)

| Size | `major_diameter` | `pitch` | TPI |
|------|-----------|---------|-----|
| `"#4-40"` | 2.845 mm | 0.635 mm | 40 |
| `"#6-32"` | 3.505 mm | 0.794 mm | 32 |
| `"#8-32"` | 4.166 mm | 0.794 mm | 32 |
| `"#10-24"` | 4.826 mm | 1.058 mm | 24 |
| `"1/4-20"` | 6.35 mm | 1.27 mm | 20 |
| `"5/16-18"` | 7.938 mm | 1.411 mm | 18 |
| `"3/8-16"` | 9.525 mm | 1.588 mm | 16 |
| `"7/16-14"` | 11.113 mm | 1.814 mm | 14 |
| `"1/2-13"` | 12.7 mm | 1.954 mm | 13 |
| `"5/8-11"` | 15.875 mm | 2.309 mm | 11 |
| `"3/4-10"` | 19.05 mm | 2.54 mm | 10 |

### SAE UNF (Unified National Fine)

| Size | `major_diameter` | `pitch` | TPI |
|------|-----------|---------|-----|
| `"#4-48"` | 2.845 mm | 0.529 mm | 48 |
| `"#6-40"` | 3.505 mm | 0.635 mm | 40 |
| `"#8-36"` | 4.166 mm | 0.706 mm | 36 |
| `"#10-32"` | 4.826 mm | 0.794 mm | 32 |
| `"1/4-28"` | 6.35 mm | 0.907 mm | 28 |
| `"5/16-24"` | 7.938 mm | 1.058 mm | 24 |
| `"3/8-24"` | 9.525 mm | 1.058 mm | 24 |
| `"7/16-20"` | 11.113 mm | 1.27 mm | 20 |
| `"1/2-20"` | 12.7 mm | 1.27 mm | 20 |
| `"5/8-18"` | 15.875 mm | 1.411 mm | 18 |
| `"3/4-16"` | 19.05 mm | 1.588 mm | 16 |

### NPT (National Pipe Thread)

Diameter is the actual pipe OD, not the nominal pipe size.
Use `NPTOutside()` / `NPTInside()` for tapered geometry.

| Size | `diameter` (pipe OD) | `pitch` | TPI |
|------|---------------------|---------|-----|
| `"1/8-npt"` | 10.287 mm | 0.941 mm | 27 |
| `"1/4-npt"` | 13.716 mm | 1.411 mm | 18 |
| `"3/8-npt"` | 17.145 mm | 1.411 mm | 18 |
| `"1/2-npt"` | 21.336 mm | 1.814 mm | 14 |
| `"3/4-npt"` | 26.67 mm | 1.814 mm | 14 |
| `"1-npt"` | 33.401 mm | 2.209 mm | 11.5 |

## Thread Profile (60° V)

All three standards (ISO metric, UTS/SAE, NPT) use the same 60° symmetric V-profile with flat truncations at the crest and root.

```
        ← P →

        crest (flat)
       ╱‾‾‾‾‾╲            ─── r_maj  (major radius)
      ╱        ╲
     ╱   60°    ╲          ─── r_pitch (pitch radius, at H/2)
    ╱            ╲
   ╱______________╲         ─── r_min  (minor radius)
     root (flat)

   ├── H ──────────┤  (fundamental triangle height)
   ├── 5H/8 ───┤       (actual thread depth)
```

### Derived Variables

All derived from `pitch` (P) and `major_diameter`:

| Variable | Formula | Description |
|----------|---------|-------------|
| `H`      | `P × √3/2` ≈ `P × 0.866025` | Height of the fundamental 60° triangle |
| `r_maj`  | `major_diameter / 2` | Major radius — outermost thread surface |
| `r_min`  | `r_maj − 5H/8` = `r_maj − H × 0.625` | Minor radius — innermost thread surface (root) |

### Truncations

- **Crest truncation**: `H/8` removed from the outer tip → flat at `r_maj`
- **Root truncation**: `H/4` removed from the inner tip → flat at `r_min`
- **Effective thread depth**: `H − H/8 − H/4 = 5H/8 ≈ 0.541P`

## NPT Taper

NPT threads are tapered at **3/4 inch per foot** on diameter (1/16 per unit length).

For a thread of length `L`:
- Total diameter change: `delta_d = L / 16`
- Small end (z=0): `d_small = major_diameter - delta_d/2`
- Large end (z=L): `d_large = major_diameter + delta_d/2`
- Scale factor: `d_large / d_small`

When `taper > 0`, `Outside()` and `Inside()` build the tooth polygon at the small-end radius and use the Extrude `scaleX`/`scaleY` parameters to grow the cross-section linearly to the large end.

## Tooth Profile (2D Cross-Section)

The tooth is a trapezoid defined in polar coordinates — each vertex is placed at its true `(r, θ)` position using `(r·cos θ, r·sin θ)`.

### Angular Half-Widths

| Position | Axial half-width | Angular half-width | Derivation |
|----------|------------------|--------------------|------------|
| Crest    | `P/16`           | `π/8 = 22.5°`     | `(P/16) × (2π/P)` |
| Root     | `3P/8`           | `3π/4 = 135°`     | `(3P/8) × (2π/P)` |

### Why Polar Coordinates?

Manifold's `Extrude` with twist rotates the 2D cross-section around the Z-axis. A point at `(x, y)` orbits at radius `√(x² + y²)`. With polar coordinates, `(r·cos θ, r·sin θ)` always orbits at exactly radius `r`, keeping every vertex at its correct distance from the bolt axis during the helical twist.

### Polygon Construction

```
           crest arc (4 pts)
         ╭──at r_maj──╮
        ╱    45° span   ╲
leading╱                  ╲trailing
flank ╱  (10 pts each)     ╲flank
     ╱                      ╲
    ╰──────── root arc ──────╯
         at r_min (8 pts)
          90° span
```

1. **Leading flank** (10 points): `r` from `r_min→r_maj`, `θ` from `-135°→-22.5°`
2. **Crest arc** (4 points): `r = r_maj`, `θ` from `-22.5°→+22.5°`
3. **Trailing flank** (10 points): `r` from `r_maj→r_min`, `θ` from `+22.5°→+135°`
4. **Root arc** (8 points): `r = r_min`, `θ` from `+135°→+225°` (closing the polygon)

## Helical Extrusion

```chisel
tooth.Extrude(length, turns * 80, turns * 360 deg, scaleX, scaleY)
```

| Extrude Parameter | Straight | NPT | Purpose |
|-------------------|----------|-----|---------|
| height            | `length` | `length` | Total axial extent |
| slices            | `turns × 80` | `turns × 80` | 80 cross-sections per revolution |
| twist             | `turns × 360 deg` | `turns × 360 deg` | One full rotation per turn |
| scaleX, scaleY    | `1, 1` | `taper_scale` | NPT: grows cross-section for taper |

Because the polygon wraps 270° around the origin, the extruded solid already includes the bolt core (material from 0 to `r_min`). No separate core cylinder is needed.

## References

- [ISO 68-1 Metric Thread Profile — Engineers Edge](https://www.engineersedge.com/hardware/iso_681_metric_thread_14599.htm)
- [Unified Thread Standard — Wikipedia](https://en.wikipedia.org/wiki/Unified_Thread_Standard)
- [NPT Thread — Engineers Edge](https://www.engineersedge.com/hardware/npt-national-pipe-thread-taper-chart.htm)
