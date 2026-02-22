# Facet Community Libraries

Shared libraries for [Facet](https://github.com/firstlayer-xyz/facet), a code-based CAD application.

## Libraries

### threads

ISO metric, UTS/SAE, and NPT pipe thread generation.

```
var T = lib "github.com/firstlayer-xyz/facetlibs/threads@main";
var t = T.New("m8");
return t.Outside(20 mm);
```

### knurling

Diamond crosshatch grip texture for cylindrical solids.

```
var K = lib "github.com/firstlayer-xyz/facetlibs/knurling@main";
var knurl = K.NewKnurl(40, 0.5 mm, 30 deg);
return knurl.Apply(cylinder);
```

### fasteners

ISO metric fasteners: hex nuts, hex bolts, socket head cap screws, button head cap screws, countersunk screws, set screws, thumbscrews, flat head (slotted) screws, Phillips screws, and standoffs.

```
var F = lib "github.com/firstlayer-xyz/facetlibs/fasteners@main";
var f = F.New("m8");
return f.HexBolt(30 mm);
```

## Usage

Import any library in your Facet source using `lib` with a GitHub URL:

```
var L = lib "github.com/firstlayer-xyz/facetlibs/<library>@main";
```
