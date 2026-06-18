# Facet Library Authoring Guide

## What is a Facet library?

A Facet library is a directory containing a single `.fct` file named after the directory. For example, a library at `fasteners/` must contain `fasteners/fasteners.fct`. Functions, structs, and methods defined in the file become available through the library's import variable.

The standard library is always available in root scope without an explicit import — `Cube`, `Move`, `Sin`, etc. all work out of the box. Every other library, including the community libraries (`threads`, `knurling`, `fasteners`, …), must be brought in with an explicit `lib` import.

## Directory structure

Libraries use a simple directory layout. A monorepo can contain multiple libraries as subdirectories:

```
facetlibs/
  fasteners/
    fasteners.fct
  threads/
    threads.fct
```

Each subdirectory is a separate importable library. The `.fct` file must be named the same as its containing directory.

## Creating a library

A library is plain Facet code. There is no special declaration or manifest file. Any functions, types, and methods you define are exported automatically.

### Example: a simplified fasteners library

Here is a small, self-contained library that defines a `Fastener` type with two methods. (The real fasteners library at [`github.com/firstlayer-xyz/facetlibs`](https://github.com/firstlayer-xyz/facetlibs) is larger and structured differently — this is a teaching example.)

**fasteners/fasteners.fct**

```
var T = lib "github.com/firstlayer-xyz/facetlibs/threads@main"

type Fastener {
    size   String
    hex_af Length
    nut_h  Length
    bolt_h Length
}

fn Fastener(size String) Fastener {
    if size == "m3" {
        return Fastener{size: "m3", hex_af: 5.5 mm, nut_h: 2.4 mm, bolt_h: 2.0 mm}
    }
    return Fastener{size: "m8", hex_af: 13.0 mm, nut_h: 6.8 mm, bolt_h: 5.3 mm}
}

fn Fastener.HexNut() Solid {
    var r = self.hex_af / (2 * Cos(a: 30 deg))
    var body = Circle(r: r, segments: 6).Extrude(z: self.nut_h)
    # The hex body is corner-anchored (axis at (r, r)); re-center the thread
    # onto it so the bore lands on the centerline, not the corner.
    var hole = T.Thread(size: self.size).Inside(length: self.nut_h)
        .AlignBottom(with: body, centerX: true, centerY: true)
    return body - hole
}

fn Fastener.HexBolt(length Length) Solid {
    var r = self.hex_af / (2 * Cos(a: 30 deg))
    var head = Circle(r: r, segments: 6).Extrude(z: self.bolt_h)
    var shaft = T.Thread(size: self.size).Outside(length: length).StackOnTop(of: head)
    return head + shaft
}
```

All library code lives in the single file.

### Methods

Methods extend a type with behavior using the implicit `self` receiver:

```
fn Fastener.NutHeight() Length {
    # `self` refers to the Fastener instance; access fields with self.<name>
    return self.nut_h
}
```

## Publishing

Push your library directory to a Git repository on GitHub, GitLab, or any public Git host. Tag releases using semantic versioning:

```bash
git tag v1.0
git push origin v1.0
```

Tags let consumers pin a stable version. You can also pin a branch name (`main`) or a commit SHA for development versions.

## Importing libraries

### The standard library

The standard library is built in and always available — you never import it. `Cube`, `Sphere`, `Move`, `Cos`, `Polygon`, and the rest of the stdlib work in root scope out of the box.

### Remote libraries

Every other library is imported from a Git repository by specifying the host, path, and `@ref`:

```
var T = lib "github.com/firstlayer-xyz/facetlibs/threads@main"
```

The `@ref` can be a tag, branch name, or commit SHA. On first use, the library is auto-cloned and cached locally. Pinning a tag or SHA gives reproducible builds; pinning a branch like `@main` tracks the latest.

### Monorepo support

A repository can contain multiple libraries as subdirectories. The path after `host/user/repo` selects the subdirectory:

```
var F = lib "github.com/firstlayer-xyz/facetlibs/fasteners@main"
var T = lib "github.com/firstlayer-xyz/facetlibs/threads@v1.0"
```

### Relative imports within a repo

A library that lives alongside others in the same repository imports its siblings with a relative path — no host or `@ref`:

```
# inside fasteners/fasteners.fct
var T = lib "../threads"
```

Relative imports resolve within the same repository at the same pinned ref, so a SHA pin on the top-level import stays self-contained.

### Transitive dependencies

If a library imports other libraries, those are resolved automatically but don't leak into your namespace:

```
# The fasteners library internally imports threads,
# but as a consumer you only see what fasteners exports.
var F = lib "github.com/firstlayer-xyz/facetlibs/fasteners@main"
```

## Using a library

Access everything through dot notation on the import variable. Suppose you published the simplified `fasteners` example above to your own repository:

```
var F = lib "github.com/yourname/fasteners@v1.0"

fn Main() {
    var f = F.Fastener(size: "m8")
    return []Solid[f.HexBolt(length: 30 mm), f.HexNut().Move(x: 30 mm)]
}
```

Values returned by library functions carry their methods with them — call methods directly on the value regardless of where the type was defined. Note that Facet requires **named arguments** for every call, including calls into a library (`F.Fastener(size: "m8")`, not `F.Fastener("m8")`).

## Managing libraries

Libraries can also be installed and managed through the Settings > Libraries panel in the Facet app:

- **Clone** — add a library by URL and ref
- **Update** — pull latest changes for branch-based installs
- **Tag** — create and push a new git tag (for library authors)
- **Remove** — delete a local clone
