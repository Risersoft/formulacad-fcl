# 📘 NOTE FOR AI — HOW TO GENERATE VALID FCL (FORMULACAD LANGUAGE)

Use this note as the **complete specification** when generating FCL.

Assume **no prior context**.

In this session the user **may provide**:

* `catalog.json` — list of FCL rules and parameters
* `functions.json` — list of allowed `@functions`
* `components.json` — allowed external component keys

Your job is to produce **syntactically valid**, **catalog-correct**, and **engine-compatible** FCL.

---

# 1. What is FCL?

FCL (FormulaCAD Language) is a **declarative DSL for parametric CAD**.

It describes:

* 3D **parts**
* 3D **assemblies**
* reusable **curve** definitions
* **2D geometry**
* **drawings**

Properties:

* deterministic — same FCL → same CAD
* rule-based
* no loops / conditionals
* Co-ordinate system is Z-UP (X=Longitudinal, Y=Transverse, Z=Height)
* geometry produced through **rules with parameters and explicitly evaluated expressions**

---

# 2. External Data You May Be Given

## 2.1 `catalog.json`

Defines all **available rules**.

Example rules:

```text
circle
line
rect
extrude
revolve
child
```

Each rule contains:

* `ShortKey`
* `Variants`
* `Params`

Each parameter defines:

* `Key`
* `Type`
* `Required`
* optional `Aliases`
* optional `Enums`

### Important rule

You must only emit **rules and parameters defined here**.

The only universal parameter typically available across rules is:

```text
qty
```

---

## 2.2 `functions.json`

Defines allowed functions.

Examples:

```text
@rotZDeg(angle)
@rotXDeg(angle)
@mulMat(A,B)
```

Rules:

* use only listed functions
* respect argument types
* respect argument counts

Example:

```text
rotation=@rotZDeg(90)
```

---

## 2.3 `components.json`

Defines **external components**.

Used with the `child` rule.

Example:

```text
child name=Beam1 key=BeamProfile
```

The `key` must match a component defined in `components.json`.

---

# 3. Component Parameters (`params`)

FCL supports **first-class parameter declarations**.

Syntax:

```text
params name=value
params name:type=value
params a=10, b=20, c=30
```

Rules:

* multiple parameters per line allowed
* multiple `params` statements allowed
* may appear at file top
* may appear inside inline parts/assemblies
* parameters are evaluated **in source order** within the same scope
* later parameters may reference earlier parameters in the same `params` block or previous `params` lines in that scope

---

## 3.1 Allowed Types

```text
number
bool
string
ref
point
list
axis
plane
matrix
expr
```

Type may be omitted when obvious.

---

## 3.2 Type inference rules

Type inference is **structure-based**.

### number

```text
800
20.5
800mm
8deg
```

### bool

```text
true
false
```

### string

```text
"ABC"
'IPE200'
```

### ref

```text
#Something
```

### point

Both forms are valid:

```text
(x,y)
(x,y,z)
@point(x,y)
@point(x,y,z)
```

### axis / vector

Both forms are valid:

```text
((x,y,z))
@vector(x,y,z)
```

### list

Both forms are valid:

```text
[a,b,c]
@list(a,b,c)
```

### matrix

Both forms are valid:

```text
[[a,b,c],[d,e,f],[g,h,i]]
@matrixl(@list(a,b,c),@list(d,e,f),@list(g,h,i))
```

### expr

```text
@( ... )
```

### Parenthesis semantics

* `( ... )` → point
* `(( ... ))` → vector / axis
* `( ... )` is **not** top-level expression grouping
* top-level expressions must use `@(...)`

### Structured literal forms

The following shorthand forms are valid:

```text
(x,y)
((x,y,z))
[a,b,c]
[[...],[...]]
```

The following explicit function forms are also valid:

```text
@point(x,y)
@vector(x,y,z)
@list(a,b,c)
@matrixl(@list(...),@list(...))
```

### Notes

* No heuristic inference such as list length is used
* Structure determines type
* Both shorthand and explicit function syntax are valid where the target type permits them

---

## Evaluation Model

FCL does not evaluate expressions implicitly.

All **top-level** evaluation must be explicitly marked using `@`.

Examples:

```text
param=10
param=$x
param=@func(...)
param=@(...)
```

This ensures:

* no ambiguity about evaluation
* clear separation between FCL and formula engine

Rules:

* Bare expressions like `$a+$b` are **not allowed as top-level values**
* Inside structured values and function arguments, bare expressions are allowed
* `$param` alone is allowed
* `@function(...)` is allowed

Examples:

```text
param=@($x+$y)
c=($edge + $index*$pitch, $width/2)
.translate=@vector($dx,$dy,$dz)
@foo($a+$b, 10)
```

---

## 3.3 Example parameter declarations

```text
params plate_len=800, plate_wid=400, plate_thk=20
params hole_r=12, hole_edge=50
params beamProfile="IPE200"
params basePlane:plane=#XY_Plane
params origin:point=(0,0,0)
params dir:axis=((0,0,1))
params vals:list=[10,20,30]
params rot:matrix=[[1,0,0],[0,1,0],[0,0,1]]
params gap:expr=@($a+$b)
```

---

# 4. Feature Syntax

General form:

```text
featureKey key=value key=value ...
```

Example:

```text
circle name=hole1 c=(0,0) r=10
rect p1=(0,0) p2=(100,50) mode=2p
extrude profile=#base distance=20 mode=add
```

Explicit-function forms are also valid:

```text
rect p1=@point(0,0) p2=@point(100,50) mode=2p
```

---

## 4.1 Multiline features

Features may span multiple lines.

Continuation lines must contain **only key=value pairs**.

Example:

```text
circle
    c=(10,0)
    r=5
```

Do **not repeat the feature key** on continuation lines.

---

## 4.2 Optional grouping braces

```text
{
    circle
        c=(10,0)
        r=5
}
```

---

# 5. Environments

FCL has four environments.

---

## 5.1 Part Environment (`EnvPart`)

Contains:

* `sketch ... endsketch`
* `curve ... endcurve`
* 2D geometry
* reusable section/path definitions
* 3D operations such as `extrude`, `revolve`, `sweep`, and `loft`

Example:

```text
sketch name=base
  rect p1=(0,0) p2=(100,50) mode=2p
endsketch

extrude profile=#base distance=10 mode=add
```

Explicit-function form is also valid:

```text
sketch name=base
  rect p1=@point(0,0) p2=@point(100,50) mode=2p
endsketch
```

### Sketch plane placement

`sketch` may also be created on an explicit plane using `on=...` when the catalog supports it.

Example:

```text
sketch name=sec on=@plane((0,0,0),((0,0,1)))
  circle c=(0,0) r=8 mode=cr
endsketch
```

Use explicit sketch planes when the section orientation is an important part of the geometry, especially for 3D sweep rails.

Supports **topology tags**.

### `curve ... endcurve`

Use `curve` blocks for reusable path/section geometry that is referenced later by other features.

Typical use cases:

* `sweep` path curves
* `loft` section curves
* standalone 3D circles, splines, and helices

Example:

```text
curve name=rail
  spline
    pointlist=[(0,0,0),(40,0,20),(80,20,40),(120,20,80)]
endcurve
```

### Sweep and loft guidance

* `sweep` typically uses a **sketch** as the profile/section and a **curve** as the path.
* `loft` typically uses a list of **curve** sections.
* Keep section definitions clean and explicit.
* Prefer named references such as `profile=#sec`, `path=#rail`, and `sections=[#c0,#c1,#c2]`.

### Important modeling guidance for users

* For `sweep`, the profile should normally be a **closed sketch** when a solid result is desired. Nested closed loops may produce hollow sections.
* For `loft`, all sections should be mutually compatible. In practice, keep the same section family across the loft when possible.
* For `loft`, avoid mixing unrelated section structures unless you know the engine supports that combination.
* For 3D curves (circles, arcs), `normal=((x,y,z))` defines the plane orientation. Without it, a default plane is assumed.
* For vectors/axes, use **double parentheses**: `((x,y,z))`. Single parentheses denote points.
* `helix` is a path-style curve rule. It is useful as sweep rail geometry and should normally be defined inside a `curve` block.
* For helix-based sweeps, the profile sketch should be defined on a plane derived from the helix start frame (e.g., via @CurvePlane(...)), unless there is a deliberate and justified alternative.
* The helix start frame refers to a coordinate frame aligned with the curve at its start (typically tangent direction with a stable normal/binormal basis).
* Do not assume a profile drawn on the default XY plane will sweep correctly along a helix.
* In general, use placement/transform operations for positioning geometry. However, when the sweep rail is strongly 3D (such as a helix), orientation must be encoded at the sketch-plane level rather than via post-placement transforms.
* When generating FCL, do not rely on implicit sweep behavior to reorient profiles, as this can introduce unintended twist or misalignment.
* If you specify a sketch plane using `on=...`, do not apply additional transforms or rely on sweep behavior to change its orientation. The specified plane is the sole source of orientation for that sketch.
* Implicit Booleans: Closed profiles inside another closed profile on the same sketch are automatically treated as holes upon extrusion.
* Native Z-Axis: 3D primitives like cylinder, cone, and torus build natively along the Z-axis. Stack them vertically by applying Z-translations (.translate = @vector(0, 0, Z)) to their parent component.

---

## 5.2 Assembly Environment (`EnvAsm`)

Contains:

* `child`
* inline `part`
* inline `assembly`
* placement operations

### Placement rule

External components → use `child`.

Inline components → use placement operations directly on the inline container.

Never instantiate an inline component using `child`.

> Inline `assembly` blocks support the same placement operations as `part`, including `.translate`, `.rx`, `.ry`, `.rz`, `.rotation`, and constraint calls.

---

## 📦 Inline Assembly Example (showing placement)

```text
assembly name=Frame

    params span=400, height=300

    // Place entire sub-assembly
    .translate = ((100,0,0))
    .rz = 15

    // Inline parts
    part name=Leg qty=2
        .translate = (( $index * $span , 0 , 0 ))

        sketch name=base
            rect p1=(0,0) p2=(40,40) mode=2p
        endsketch

        extrude profile=#base distance=$height mode=add
    endpart

    // External component
    child name=Beam key=BeamProfile
        .translate = ((0,0,$height))
        .rx = 90

endassembly
```

---

### Inline container semantics (`qty`, `$index`, `newdef`)

Inline `part` and `assembly` blocks may specify `qty` to create multiple instances.

```text
part name=Panel qty=10
```

`$index` is **1-based** and is available inside the container body.

Two evaluation modes exist:

#### Shared definition (default)

```text
part name=Panel qty=10
```

* geometry and params are evaluated **once** using `$index = 1`
* the same definition is reused
* placement operations are evaluated per instance

#### Per-instance definition

```text
part name=Panel qty=10 newdef=true
```

* the container body is evaluated per instance
* `$index` may affect:

  * params
  * geometry
  * placement
* each instance becomes a separate definition

Use `newdef=true` when geometry varies across instances.

---

### Placement operations

Placement operations:

```text
.translate
.rx
.ry
.rz
.rotation
.constraint calls
```

Rules:

* may appear anywhere inside the container body
* are executed **in the order written**
* evaluated per instance
* All transformations are applied to the primitive at the origin in the order they are written.

Placement may:

* use `$index` directly
* use parameters

If placement depends on parameters that themselves depend on `$index`, use `newdef=true`.

---

## 5.3 2D Environment (`Env2d`)

Optional blocks:

```text
block ... endblock
```

Geometry may also appear directly.

---

## 5.4 Drawing Environment (`EnvDraw`)

Contains:

```text
sheet ... endsheet
```

Drawing rules may include:

* views
* sections
* detail views
* balloons
* leaders
* dimensions
* tables
* BOM tables

---

# 6. Expressions

Expressions must be explicitly evaluated using `@(...)` when they appear as top-level values.

Examples:

```text
@($a + $b)
@($edge + $index*$pitch)
```

Rules:

* Bare expressions like `$a+$b` are **not allowed** as top-level values
* All top-level expressions must be wrapped in `@(...)`
* `$param` alone is allowed
* `@function(...)` is allowed
* Inside structures and function arguments, bare expressions are allowed

Valid examples:

```text
param=@($x+$y)
circle c=($edge + $index*$pitch, $width/2) r=10
@foo($a+$b, $c/2)
(( $dx , $dy , $dz ))
[[1,0,0],[0,$c/2,0],[0,0,1]]
```

Invalid example:

```text
param=$x+$y
```

---

# 7. `$index` Pattern

Used with `qty`.

Example:

```text
circle
    c=($edge + $index*$pitch, $width/2)
    r=10
    qty=4
```

`$index` values are **1-based**:

```text
1,2,3
```

---

# 8. The `child` Rule (Assembly Placement)

Placement uses **ordered inline placement operations**.

Two syntaxes exist.

### Assignment form

```text
.rx = 90
.translate = @vector(10,0,0)
.rotation = @rotZDeg(90)
```

Shorthand form may also be used where appropriate:

```text
.translate = ((10,0,0))
```

### Call form

```text
.xy : mate(yz,2)
.z  : offset(z,100)
```

These operations are executed **in the order written**.

They are **not normal feature parameters**.

---

## 8.1 Canonical child form

```text
child name=label key=<componentKey>
      qty=<number>?
      .translate = @vector(x,y,z)?
      .rx = <number>?
      .ry = <number>?
      .rz = <number>?
      .rotation = [...]?
      .dof : constraintCall(...)?
```

Equivalent shorthand may also be used for typed values:

```text
child name=label key=<componentKey>
      .translate = ((x,y,z))
```

---

## 8.2 Non-placement parameters

`key`

* required
* string or `$param`

`qty`

* optional
* default = 1
* defines `$index`

---

## 8.3 Inline placement operations

Examples:

```text
.translate = @vector(100,0,0)
.translate = ((100,0,0))
.rx = 90
.rotation = @rotZDeg(30)

.xy : mate(yz,2)
.z  : offset(#Edge7,100)
```

Rules:

* start with `.`
* valid only in placement contexts
* executed in file order
* stored separately from normal parameters

---

# 9. Inline Part / Assembly Placement

Inline `part` or `assembly` definitions **create instances automatically**.

If placement operations are not provided, instances appear at **default placement** (typically origin) and may overlap.

Therefore inline components should normally include placement operations.

Example:

```text
part name=Bracket1 qty=4
    .translate = ((0,0,$index*100))
    .rz = 90

    sketch name=base
      rect p1=(0,0) p2=(100,50) mode=2p
    endsketch

    extrude profile=#base distance=10 mode=add
endpart
```

Equivalent explicit-function form is also valid:

```text
part name=Bracket1 qty=4
    .translate = @vector(0,0,$index*100)
```

Inline components **must not** be instantiated using `child`.

Invalid example:

```text
part name=Bracket1
 ...
endpart

child name=Bracket1 key=Bracket1
```

---

# 10. Topology Tags

Tags label faces or edges inside parts.

Example:

```text
tag name=MountFace sel=faces.top color="orange"
tag name=MountEdge sel=edges[7]
```

Parameters:

* `name`
* `sel`
* optional `color`

Selectors include:

```text
faces.top
faces.bottom
faces.left
faces.right
faces.front
faces.back
edges[n]
```

References:

```text
#Bracket1.MountFace
#Bracket1.faces.top
#Bracket1.edges[7]
```

---

# 11. BOM Blocks

Assemblies and drawings may contain BOM blocks.

Example:

```text
bom
    params part_no="A1"
    params qty=4
endbom
```

Rules:

* may contain `params`
* represent metadata
* do not create geometry

---

# 12. Rule & Param Correctness (using `catalog.json`)

For every feature (rule) you emit:

1. The feature key (e.g. `circle`, `rect`, `extrude`, `child`) must exist in `catalog.json` as `ShortKey`.

2. If the rule has multiple `Variants`, choose parameters that match exactly one variant.

   Use `mode=...` if required to disambiguate.

3. Every parameter must:

   * exist in the variant’s `Params` list, or be `qty`
   * have a compatible type

4. Missing required parameters → **do not emit the feature**.

Do **not invent** new rule names or parameters.

Inline placement operations (`.rx=`, `.translate=`, `.xy:mate(...)`) are **not part of catalog parameters** and are validated separately.

---

# 13. Common Patterns

### Params

```text
params plate_len=800, plate_wid=400, plate_thk=20
```

### Sketch + extrude

```text
sketch name=base
  rect p1=(0,0) p2=($plate_len,$plate_wid) mode=2p
endsketch

extrude profile=#base distance=$plate_thk mode=add
```

Equivalent explicit-function form:

```text
sketch name=base
  rect p1=@point(0,0) p2=@point($plate_len,$plate_wid) mode=2p
endsketch
```

### Pattern via `$index`

```text
circle
    c=($edge + $index*$pitch, $width/2)
    r=10
    qty=$count
```

### Curve + sweep

```text
sketch name=sec
  circle c=(0,0,0) r=8 mode=cr
endsketch

curve name=rail
  spline
    pointlist=[
      (0,0,0),
      (40,0,20),
      (80,20,40),
      (120,20,80)
    ]
endcurve

sweep name=Body profile=#sec path=#rail solid=true
```

### Helix + sweep

```text
curve name=SpringPath
  helix
    radius=30
    height=200
    turns=6
endcurve

sketch name=WireProfile on=@CurvePlane(#SpringPath)
  circle mode=cr centre=(0,0) radius=5
endsketch

sweep name=Coil profile=#WireProfile path=#SpringPath solid=true
```

### Curve + loft

```text
curve name=c0
  circle c=(0,0,0) r=20 mode=cr
endcurve

curve name=c1
  circle c=(20,10,40) r=16 normal=((0.2,0,1)) mode=cr
endcurve

curve name=c2
  circle c=(-12,22,80) r=10 normal=((0.4,0.2,1)) mode=cr
endcurve

curve name=c3
  circle c=(8,-6,120) r=6 normal=((0.1,0.5,1)) mode=cr
endcurve

loft name=Body sections=[#c0,#c1,#c2,#c3] solid=true
```

### Helix as a path curve

```text
curve name=rail
  helix
    radius=20
    height=100
    turns=5
endcurve
```

### Child placement

```text
child name=BrktRow key=BracketDef
      qty=$count
      .translate = (($edge + $index*$pitch,0,0))
      .rz = 90
      .xy : mate(yz,2)
```

Equivalent explicit-function form:

```text
child name=BrktRow key=BracketDef
      qty=$count
      .translate = @vector($edge + $index*$pitch,0,0)
      .rz = 90
      .xy : mate(yz,2)
```

### Inline part placement

```text
part name=Bracket1 qty=4
    .translate = ((0,0,$index*100))
    .rz = 90

    sketch name=base
      rect p1=(0,0) p2=(100,50) mode=2p
    endsketch

    extrude profile=#base distance=10 mode=add
endpart
```

Equivalent explicit-function form:

```text
part name=Bracket1 qty=4
    .translate = @vector(0,0,$index*100)
```

### Spring array assembly

```text
params spacing=120, height=200

assembly name=LinearSuspension

    part name=Spring qty=3 newdef=true
        .translate = ((($index-1) * $spacing, 0, 0))

        curve name=SpringPath
            helix radius=30 height=$height turns=6
        endcurve

        sketch name=WireProfile on=@CurvePlane(#SpringPath)
            circle mode=cr centre=(0,0) radius=5
        endsketch

        sweep name=Coil profile=#WireProfile path=#SpringPath solid=true
    endpart

    part name=TopMount
        .translate = ((0, 0, $height))

        sketch name=PlateBase
            rect mode=plw point1=(-40, -40) length=320 width=80
        endsketch

        extrude profile=#PlateBase distance=10 mode=add
    endpart

endassembly
```

---

# Final Generation Rules

When generating FCL:

1. Use only rules in `catalog.json`.
2. Use only functions in `functions.json`.
3. Use only components in `components.json`.
4. Respect environment rules.
5. Use inline placement operations for ordered placement.
6. Never instantiate inline components using `child`.
7. Prefer explicit geometric intent over relying on hidden or implicit engine behavior.

Following these rules ensures generated FCL is **valid, deterministic, and engine-compatible**.
