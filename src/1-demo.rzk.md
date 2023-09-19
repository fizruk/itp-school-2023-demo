# Demo

## Disclaimers

!!! info

    This demo is for the [«Interactions of Proof Assistants and Mathematics» school](https://itp-school-2023.github.io)
    in Regensburg, September 18–29, 2023 and contains introduction to the syntax of Rzk
    and basics of theorem proving in Rzk, as well as a few exercises
    and links to formalisation projects.

!!! warning

    This demo relies on [`rzk` version 0.5.4](https://rzk-lang.github.io/rzk/v0.5.4/)
    and might not be up-to-date, as the proof assistant is in active development
    and breaking changes should be expected with further releases.

## Setup

To check the formalisations in this demo you can:

1. Have `rzk` installed locally
   (the recommended way is to have [VS Code extension for Rzk](https://marketplace.visualstudio.com/items?itemName=NikolaiKudasovfizruk.rzk-1-experimental-highlighting) to handle it for you).
   and running `rzk typecheck src/*.rzk.md` from the root of this project.

2. Use an online playground at <https://rzk-lang.github.io/rzk/v0.5.4/playground/>
   (and copy-paste code blocks there one by one)

### Formalisation project structure

Usually, formalisation projects in Rzk consist of multiple Rzk (or literate Rzk) files.
For example, this demo project has the following structure:

```
itp-school-2023-demo
│
...
└── src
    ├── 1-demo.rzk.md
    └── 2-exercises.rzk.md
```

The formalisations are located in the `src/` directory and contain just the two
literate Rzk Markdown files.
For another example, [:material-github: rzk-lang/sHoTT](https://github.com/rzk-lang/sHoTT)
has its formalisations further split into subdirectories:

```
sHoTT
│
...
└── src
    ├── STYLEGUIDE.md
    ├── hott
    │   ├── 00-common.rzk.md
    │   ├── 01-paths.rzk.md
    │   ├── 02-homotopies.rzk.md
    │   ├── 03-equivalences.rzk.md
    │   ├── 04-half-adjoint-equivalences.rzk.md
    │   ├── 05-sigma.rzk.md
    │   ├── 06-contractible.rzk.md
    │   ├── 07-fibers.rzk.md
    │   ├── 08-families-of-maps.rzk.md
    │   ├── 09-propositions.rzk.md
    │   └── 10-trivial-fibrations.rzk.md
    ├── index.md
    └── simplicial-hott
        ├── 03-simplicial-type-theory.rzk.md
        ├── 04-extension-types.rzk.md
        ├── 05-segal-types.rzk.md
        ├── 06-2cat-of-segal-types.rzk.md
        ├── 07-discrete.rzk.md
        ├── 08-covariant.rzk.md
        ├── 09-yoneda.rzk.md
        ├── 10-rezk-types.rzk.md
        └── 12-cocartesian.rzk.md
```

### Running Rzk

To typecheck files, at the moment you have to run the following command in
the terminal:

```sh
rzk typecheck FILE-1 FILE-2 ... FILE-N
```

!!! warning

    It is important to pass formalisation files in the order
    you want them to be checked, as dependencies between files
    (in the form of imports) are not implemented yet.

!!! tip

    Starting filenames with numbers (as in the examples above) helps automatically
    achieve the desired order when using wildcars (e.g. `rzk typecheck src/*.rzk.md`), although in a slightly inelegant way.

### Literate Rzk

If you are familiar with Markdown, then the recommended approach is to use [literate](https://en.wikipedia.org/wiki/Literate_programming) Rzk Markdown,
so that conventional Markdown rendering tools can be used to produce
readable documentation from the formalisation files.

For instance, this file is a literate Rzk Markdown file!

## Syntax overview

Each file in Rzk (or literate Rzk) must start
with a declaration of the version/dialect used.
We will use (the only supported) `rzk-1` dialect:

```rzk
#lang rzk-1
```

The rest of the file contains primarily of `#!rzk #define`-statements,
each of which introduces a new definition into scope. For example,
consider the following definition:

```rzk
#define modus-ponens
  (A B : U)
  : (A → B) → A → B
  := \ f x → f x
```

Going line by line, we have

- `modus-ponens` as the name of a new definition
- two parameters — `A` and `B` (both of type `#!rzk U`, meaning `A` and `B` are types[^1])
- declared type of `modus-ponens` is `#!rzk (A → B) → A → B`
- declared term (value) of `modus-ponens` is `#!rzk \ f x → f x`

[^1]:
    technically, in Rzk currently `#!rzk U` contains also `#!rzk CUBE`,
    `#!rzk TOPE` universes, and itself!

Rzk typechecks the term against the declared type and,
if no type errors found, remembers this definition for future use.

### Unicode

Rzk supports both ASCII and Unicode versions of many syntactic constructions.
We will use the following Unicode symbols:

- `->` should be always replaced with `→` (`\to`)
- `|->` should be always replaced with `↦` (`\mapsto`)
- `===` should be always replaced with `≡` (`\equiv`)
- `<=` should be always replaced with `≤` (`\<=`)
- `/\` should be always replaced with `∧` (`\and`)
- `\/` should be always replaced with `∨` (`\or`)
- `0_2` should be always replaced with `0₂` (`0\2`)
- `1_2` should be always replaced with `1₂` (`1\2`)
- `I * J` should be always replaced with `I × J` (`\x` or `\times`)

We use ASCII versions for `TOP` and `BOT` since `⊤` and `⊥` do not read better
in the code.

## Variables

We will use the following identifiers without corresponding term
definitions. These will serve as assumptions to be used for
demonstration purposes:

```rzk
#variables X : U
#variables y z : X
```

## Dependent types with Rzk

We now proceed to look at the primitives in Rzk
for working with dependent types.

### Functions

The type `#!rzk (x : A) → B x` is the type of (dependent)
functions with an argument of type `A` and, for each input `x`,
the output type `B x`.

As a simple example of a dependent function,
consider the identity function:

```rzk
#define identity
  : (A : U) → (x : A) → A
  := \ A x → x
```

Since we are not using `x` in the type of `identity`,
we can simply write the type of the argument,
without providing its name:

```rzk
#define identity₁
  : (A : U) → A → A
  := \ A x → x
```

We can write this definition differently,
by putting `#!rzk (A : U)` into parameters (before `:`),
and omitting it in the lambda abstraction:

```rzk
#define identity₂
  (A : U)
  : A → A
  := \ x → x
```

We could also move `x` into parameters as well,
although this probably does not increase readability anymore:

```rzk
#define identity₃
  (A : U)
  (x : A)
  : A
  := x
```

We can compute the `identity` using `#!rzk #compute` command:

```rzk
#compute identity
```

You should see the following after running Rzk:

```
[ 8 out of 13 ] Computing WHNF for identity
  \ (A : U) → \ (x : A) → x
```

We can also apply `identity` to the variables we have introduced earlier:

```rzk
#compute identity X       -- \ (x : X) → x
#compute identity X z     -- z
```

### Identity type

For each type `#!rzk (A : U)` and elements `#!rzk (a b : A)`
we have the identity type `#!rzk a =_{A} b` of equalities (identities, paths)
between `a` and `b`.

```rzk
#define FunExt
  : U
  := (A : U) → (B : U)
    → (f : A → B) → (g : A → B)
    → ((x : A) → f x =_{B} g x)
    → (f =_{A → B} g)
```

Rzk can figure out the type indices for identity types and we can omit them:

```rzk
#define FunExt₁ : U
  := (A : U) → (B : U)
    → (f : A → B) → (g : A → B)
    → ((x : A) → f x = g x)
    → (f = g)
```

One way to prove an equality that exists for all types,
is the proof by reflexivity. For example,

```rzk
#define identity-x-eq-x
  (A : U)
  (x : A)
  : (identity A x = x)
  := refl_{x : A}
```

Indeed, `identity A x` computes to `x`,
and is therefore definitionally (computatioally) equal to it
allowing for the use of `refl_{x : A}`.
Again, Rzk can figure out indices accepting `refl_{x}` or just `refl`.

Using a value of an identity type requires the path induction,
which we can define via the built-in version of it, `#!rzk idJ`:

```rzk
#define ind-path
  (A : U)
  (a : A)
  (C : (x : A) -> (a = x) -> U)
  (d : C a refl)
  (b : A)
  : (p : a = b) → C b p
  := \ p → idJ (A, a, C, d, b, p)
```

As an example, we can show symmetry for equality types
by induction on the argument of type `a = b`:

```rzk
#define inverse
  (A : U)
  (a b : A)
  : (a = b) → (b = a)
  := ind-path A a (\ x _ → x = a) refl b
```

### Dependent sums

The type `#!rzk Σ (x : A), B x` is a type of (dependent) pairs
where the first component (named `x` here) is of type `A`,
and the second component is of type `B x`.

For example, preimages (fibers) of functions can be defined
using `#!rzk Σ`-types:

```rzk
#define preimage
  (A B : U)
  (f : A → B)
  (y : B)
  : U
  := Σ (x : A), (f x = y)
```

An element of a type `Σ (x : A), B x` is given by a pair of elements `(a, b)`
of types `A` and `B a` correspondingly. For example, we can provide
an element in the preimage of `identity X` at `z : X`:

```rzk
#define preimage-of-identity-X-at-z
  : preimage X X (identity X) z
  := (z, refl)
```

## HoTT with Rzk

In HoTT (and in Rzk), the identity type is not always a proposition
and `refl` might not be the only proof of equality!

```{ unchecked .rzk title="Type Error!" }
#define does-not-typecheck
  (A : U)
  (x : A)
  (p : x = x)
  : p = refl
  := refl -- path induction on p also cannot work!
```

!!! info

    At the moment Rzk does not have support for higher inductive types,
    but it is expected in the future.

## Simplicial types with Rzk

Following Riehl–Shulman's paper[^2], Rzk has cube and tope layers[^3].
These provide the ability to specify (higher) diagram schemas.

[^2]:
    Emily Riehl & Michael Shulman. A type theory for synthetic ∞-categories.
    Higher Structures 1(1), 147-224. 2017. <https://arxiv.org/abs/1705.07442>

[^3]:
    Instead of builtin layers, Rzk uses `#!rzk CUBE` and `#!rzk TOPE` universes
    to separate them from the types.

For the purposes of his demo, we will only care about the directed interval cube
`#!rzk 2`, 2-dimensional directed cube `#!rzk (2 × 2)`, and 3-dimensional
directed cube `#!rzk (2 × 2 × 2)`.

A cube may have points in it, and the directed interval `#!rzk 2`
has two known points `#!rzk 0₂ : 2` and `#!rzk 1₂ : 2`.

Tope layer adds logical constraints on the points in a cube,
"carving out" a shape inside a space. There is a handful of ways to specify topes:

- `#!rzk TOP` selects all points (provides no constraints)
- `#!rzk BOT` selects no points (producing an empty shape)
- `#!rzk (φ ∧ ψ)` selects all points that satisfy both `φ` and `ψ`
- `#!rzk (φ ∨ ψ)` selects all points that satisfy `φ` or `ψ` (or both)
- `#!rzk (t ≡ s)` selects all points such that `t` is equal to `s`
- `#!rzk (t ≤ s)` selects all points such that `t ≤ s` (only when both `t` and `s` are in `#!rzk 2`)

Mostly for technical reasons, we define shapes as mappings from cubes into the `#!rzk TOPE` universe.
For example, here is a shape that carves a (directed) triangle out of a (directed) square:

```rzk
#define Δ²
  : (2 × 2) → TOPE
  := \ (t, s) → (s ≤ t)
```

Another example would be the horn shape that only takes two edges of a square:

```rzk
#define Λ
  : (2 × 2) → TOPE
  := \ (t, s) → (s ≡ 0₂) ∨ (t ≡ 1₂)
```

We could refine this definition by specifying that it is a subshape of `Δ²`:

```rzk
#define Λ'
  : ( (t, s) : 2 × 2 | Δ² (t, s) ) → TOPE
  := \ (t, s) → (s ≡ 0₂) ∨ (t ≡ 1₂)
```

Since shapes have the information about the cube in their types,
Rzk provides an implicit coercion, allowing shorter definitions:

```rzk
#define Λ''
  : Δ² → TOPE
  := \ (t, s) → (s ≡ 0₂) ∨ (t ≡ 1₂)
```

!!! note

    This more precise type expresses a restriction on the definition of `Λ''`,
    but, perhaps counterintuitively, **not** on the input points!
    With this type, Rzk additionally checks that `#!rzk (s ≡ 0) ∨ (t ≡ 1)`
    implies `Δ² (t, s)` for all `t`, `s`. However, we can still apply `Λ''`
    to **all points** in the cube `#!rzk (2 × 2)`.

Mapping from a shape into a type, effectively selects a concrete diagram in it.
We can select a subdiagram, simply by restricting to a subshape:

```rzk
#define horn-restriction
  (A : U)
  : (Δ² → A) → (Λ → A)
  := \ f t → f t
```

To construct diagrams from parts, we can use `#!rzk recOR`
by specifying values for several shapes. Rzk will be
checking that provided values agree (definitionally) on the intersections
of these shapes.

For example, given a triangle, we can construct a square:

```rzk
#define unfolding-square
  (A : U)
  (triangle : Δ² → A)
  : (2 × 2) → A
  :=
    \ (t, s) →
    recOR
      ( t ≤ s ↦ triangle (s , t) ,
        s ≤ t ↦ triangle (t , s))
```

## Extension types with Rzk

Mapping from a shape into a type, we get a diagram.
To fix some parts of the diagram, we can specify refinements — for each subshape we want to fix,
we provide an explicit term of the proper type.

For example, taking a diagram in `A` correspoding to the directed interval `#!rzk 2`
and fixing the endpoints, we get the type of all arrows between two given points in `A`:

```rzk
#define hom
  (A : U)
  (x y : A)
  : U
  := (t : 2) →
    A [ t ≡ 0₂ ↦ x
      , t ≡ 1₂ ↦ y ]
```

We can show that the diagram `#!rzk 2 → A` consists exactly of two elements and an arrow
between them:

```rzk
#define Σ-arrow
  (A : U)
  : U
  := Σ (x : A), Σ (y : A), hom A x y

#define 2-to-Σ
  (A : U)
  : (2 → A) → Σ-arrow A
  := \ d → (d 0₂, (d 1₂, d))

#define Σ-to-2
  (A : U)
  : (Σ-arrow A) → (2 → A)
  := \ (x, (y, f)) → \ t →
    recOR
      ( t ≡ 0₂ ↦ x
      , t ≡ 1₂ ↦ y
      , TOP ↦ f t )
```

## Technical cleanup

The following definitions silence errors
about unused variables.

```rzk
#define unused-y : X := y
#define unused-z : X := z
```

<!-- Definitions for the SVG images below -->
<svg width="0" height="0">
  <defs>
    <style data-bx-fonts="Noto Serif">@import url(https://fonts.googleapis.com/css2?family=Noto+Serif&display=swap);</style>
    <marker id="arrow" viewBox="0 0 10 10" refX="5" refY="5"
      markerWidth="5" markerHeight="5" orient="auto-start-reverse">
      <path d="M 0 2 L 5 5 L 0 8 z" stroke="black" fill="black" />
    </marker>
  </defs>
  <style>
    text, textPath {
      font-family: Noto Serif;
      font-size: 28px;
      dominant-baseline: middle;
      text-anchor: middle;
    }
  </style>
</svg>
