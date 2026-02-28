# Facet Library Authoring Guide

## What is a Facet library?

A Facet library is a directory containing a single `.fct` file named after the directory. For example, a library at `fasteners/` must contain `fasteners/fasteners.fct`. Functions, structs, and methods defined in the file become available through the library's import variable.

The standard library (`facet/std`) is always available in root scope without an explicit import — `Cube`, `Translate`, `Sin`, etc. all work out of the box. Other built-in libraries like `facet/threads` require an explicit `lib` import.

## Directory structure

Libraries use a simple directory layout. A monorepo can contain multiple libraries as subdirectories:

```
facetlibs/
  fasteners/
    fasteners.fct
  gears/
    gears.fct
```

Each subdirectory is a separate importable library. The `.fct` file must be named the same as its containing directory.

## Creating a library

A library is plain Facet code. There is no special declaration or manifest file. Any functions, structs, and methods you define are exported automatically.

### Example: the fasteners library

The real fasteners library lives at [`github.com/firstlayer-xyz/facetlibs`](https://github.com/firstlayer-xyz/facetlibs). Here's a simplified excerpt:

**fasteners/fasteners.fct**

```
var T = lib "facet/threads";

struct Fastener {
    String size;
    Length major_d;
    Length hex_af;
    Length nut_h;
    Length bolt_head_h;
}

Fastener Fastener(String size) {
    return if size == "m3" {
        return Fastener {
            size: "m3", major_d: 3 mm,
            hex_af: 5.5 mm, nut_h: 2.4 mm,
            bolt_head_h: 2.0 mm,
        };
    } else {
        return Fastener {
            size: "m8", major_d: 8 mm,
            hex_af: 13 mm, nut_h: 6.8 mm,
            bolt_head_h: 5.3 mm,
        };
    };
}

Solid Fastener.HexNut() {
    var cr = self.hex_af / (2 * Cos(30 deg));
    var hex = Polygon(for i [0:<6] {
        yield Point2d(Cos(i * 60 deg) * cr, Sin(i * 60 deg) * cr);
    }).Extrude(self.nut_h);
    var internal = T.Thread(self.size).Inside(self.nut_h);
    return hex - internal;
}

Solid Fastener.HexBolt(Length length) {
    var cr = self.hex_af / (2 * Cos(30 deg));
    var head = Polygon(for i [0:<6] {
        yield Point2d(Cos(i * 60 deg) * cr, Sin(i * 60 deg) * cr);
    }).Extrude(self.bolt_head_h);
    var shaft = T.Thread(self.size).Outside(length)
        .Translate(0 mm, 0 mm, self.bolt_head_h);
    return head + shaft;
}
```

All library code lives in the single file.

### Methods

Methods extend a struct with behavior using the implicit `self` receiver:

```
Solid Fastener.HexNut() {
    # self refers to the Fastener instance
    return ...;
}
```

## Publishing

Push your library directory to a Git repository on GitHub, GitLab, or any public Git host. Tag releases using semantic versioning:

```bash
git tag v1.0
git push origin v1.0
```

Tags let consumers pin a stable version. You can also use branch names (`main`) for development versions.

## Importing libraries

### Built-in libraries

Facet ships with built-in libraries under the `facet/` prefix:

```
var T = lib "facet/threads";
var K = lib "facet/knurling";
var G = lib "facet/gears";
```

### Remote libraries

Import from a Git repository by specifying the host, path, and `@ref`:

```
var F = lib "github.com/firstlayer-xyz/facetlibs@main";
```

The `@ref` can be a tag or branch name. On first use, the library is auto-cloned and cached locally.

### Monorepo support

Repositories can contain multiple libraries as subdirectories. The path after `host/user/repo` selects the subdirectory:

```
var F = lib "github.com/firstlayer-xyz/facetlibs@main";
var G = lib "github.com/firstlayer-xyz/facetlibs/gears@v2.0";
```

### Transitive dependencies

If a library imports other libraries, those are resolved automatically but don't leak into your namespace:

```
# The fasteners library internally uses facet/threads,
# but as a consumer you only see what fasteners exports.
var F = lib "github.com/firstlayer-xyz/facetlibs@main";
```

## Using a library

Access everything through dot notation on the import variable:

```
var F = lib "github.com/firstlayer-xyz/facetlibs@main";

Main() {
    var f = F.Fastener("m8");
    return [f.HexBolt(30 mm), f.HexNut()];
}
```

Structs returned by library functions carry their methods with them — call methods directly on the value regardless of where the struct was defined.

## Managing libraries

Libraries can also be installed and managed through the Settings > Libraries panel in the Facet app:

- **Clone** — add a library by URL and ref
- **Update** — pull latest changes for branch-based installs
- **Tag** — create and push a new git tag (for library authors)
- **Remove** — delete a local clone
