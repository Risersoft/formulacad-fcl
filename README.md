# FormulaCAD FCL — Language, Samples, and Library

Open reference materials for **FCL — the FormulaCAD Language**, a text-first approach to parametric CAD.

This repository contains the FCL specification, sample models, and the standard component library used by FormulaCAD.

Models defined here are executed and rendered by the FormulaCAD platform. The platform itself is not part of this repository.

---

## What's in here

```
/context.md
/rules.catalog.json
/functions.json
/Samples/
    /FCL/             # Complete sample FCL models
/Library/
    /IS/              # Indian Standard steel sections (1,053 components)
        /index.json
        /FCL/
    /EN/              # European Standard steel sections (796 components)
        /index.json
        /FCL/
    /AISC/            # US AISC steel sections (2,299 components)
        /index.json
        /FCL/

````

**The specification and catalog** describe the FCL language itself. An AI model reading these can generate valid FCL.

**The function library specification** lists every callable function FCL supports — `@SinDeg`, `@Max`, `@Choose`, `@If`, and so on — with signatures and behavior.

**The samples** are complete FCL models that can be built directly in FormulaCAD. This repository provides the source; visual previews are available in the online sample gallery.

**The library** contains the FCL source for the standard components published in FormulaCAD's cloud library. Each regional standard (IS, EN, US) has an `index.json` catalog of section dimensions and one or more `.fcl` templates that turn those dimensions into parametric geometry.

---

## Samples

The `Samples/FCL` directory contains complete, working FCL models — parts, assemblies, and parametric systems.

Examples include structural frames, bracket assemblies, parametric layouts, and AI-generated component systems.

These are **source definitions**, not visual previews.

To explore these models with rendered geometry, open them in the FormulaCAD sample gallery:

👉 https://formulacad.com/sample/list

Each sample can be viewed, edited, and built directly in the browser.

These models can be used directly for learning, testing, or AI grounding.

---

## What FCL is

FCL is a declarative, text-first language for describing parametric CAD — parts, assemblies, 2D layouts, and drawings.

Instead of building models through a GUI workflow, you describe them directly in text using FCL. FormulaCAD executes the FCL and builds the model.

A short example:

```fcl
params plate_len=600, plate_wid=400, plate_thk=20

part name=BasePlate
    sketch name=base
        rect p1=(0,0) p2=($plate_len,$plate_wid) mode=2p
    endsketch
    extrude profile=#base distance=$plate_thk mode=add
endpart

child name=Column key="EN.IPE200"
    span=3000
    .translate = ($plate_len / 2, $plate_wid / 2, $plate_thk)
```

FCL is designed to be simple, predictable, and reusable:

- You define the model explicitly in text, with all parameters and relationships visible in one place
- Patterns and variation are expressed using `qty`, `$index`, and functions like `@If`, `@Choose`
- The same FCL always produces the same geometry
- Components are reused by reference using `child` rules

---

## Using this corpus to ground AI

The corpus is structured so modern reasoning models can read it and reliably generate valid FCL.

Upload `context.md`, `rules.catalog.json`, `functions.json`, and relevant files from the `Samples/` directory, along with any library index files you want the model to reference.

### What to expect

Grounded on this corpus, a capable model can generate:

* Parametric parts with sketches and features
* Assemblies composed of components via `child` rules
* Patterns using `qty` and `$index`
* Mathematical and geometric expressions

Lower-tier models may hallucinate syntax. If a generated FCL doesn't parse, refine the prompt or use a stronger model.

FormulaCAD itself uses this same grounding corpus to author components in the production library.

---

## The standard library

FormulaCAD ships with a structural steel component library covering the three major regional standards.

| Standard    | Families                                                                    | Components | Source                                                     |
| ----------- | --------------------------------------------------------------------------- | ---------- | ---------------------------------------------------------- |
| IS (India)  | ISLB, ISMB, ISLC, ISMC, MSTEE, MSEA, MSUA, MSRB, MSSB, MSSHS, MSRHS, MSPIPE | 1,053      | IS 808, IS 2062, IS 1239, IS 3589, IS 4923                 |
| EN (Europe) | IPE, IPN, HEA, HEB, HEM, UPN, UPE, EA, UA, CHS, SHS, RHS                    | 796        | EN 10365, EN 10056, EN 10210, EN 10219, DIN 1025, DIN 1026 |
| US (AISC)   | W, S, M, HP, C, MC, WT, ST, MT, L, 2L, HSS, PIPE                            | 2,299      | AISC Shapes Database v16                                   |

Every component in the library was authored in FCL against this grounding corpus and is auditable against published standards. Every component's source is in this repository.

To use a library component in FCL, reference it by key:

```fcl
child name=Column key="EN.IPE200" span=3000
child name=Beam   key="IS.ISMB200" span=6000
child name=Post   key="AISC.W12X26" span=3000
```

---

## License and platform boundary

This repository is open source under the MIT License.

The FormulaCAD platform is not part of this repository.

---

## Contributing

Corrections to the specification, samples, or library are welcome. See [CONTRIBUTING.md](./CONTRIBUTING.md) for the process.

---

## Links

* [https://formulacad.com](https://formulacad.com)
* [https://formulacad.com/fcl-language](https://formulacad.com/fcl-language)
* [https://formulacad.com/generative-ai](https://formulacad.com/generative-ai)
* [https://formulacad.com/sample/list](https://formulacad.com/sample/list)
* [https://formulacad.com/home/waitlist](https://formulacad.com/home/waitlist)

Questions or feedback: open an issue in this repository, or contact Risersoft through the FormulaCAD website.


```
