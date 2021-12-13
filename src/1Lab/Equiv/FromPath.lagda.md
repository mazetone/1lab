```
open import 1Lab.HLevel
open import 1Lab.Equiv
open import 1Lab.Path
open import 1Lab.Type

module 1Lab.Equiv.FromPath {ℓ} (P : (i : I) → Type ℓ) where
```

# Equivs from Paths

In [Cubical Type Theory: a constructive interpretation of the univalence
axiom][1], Cohen et. al. give a direct _cubical_ construction of an
equivalence `A ≃ B` from a path `A ≡ B`. This is in contrast with the
_indirect_ definition, transporting the identity equivalence along the
path:

```
private
  badPathToEquiv : P i0 ≃ P i1
  badPathToEquiv = transport (λ i → P i0 ≃ P i) (id , idEquiv)
```

While `isEquiv`{.Agda} is a proposition -- and thus the particular proof
does not matter propositionally -- Agda is still a programming language,
so we still need to _evaluate_ the proof. Cohen et. al.'s construction
gives a much shorter normal form for `line→equiv`{.Agda}.

[1]: https://arxiv.org/abs/1611.02108

```
private
  ~P : (i : I) → Type ℓ
  ~P = λ i → P (~ i)

  A B : Type ℓ
  A = P i0
  B = P i1
```

The construction begins by giving the endpoints of `P`{.Agda} -- and the
inverse of `P`{.Agda} -- better names. We do the same for transports
along `P`{.Agda} and `~P`{.Agda}:

```
  f : A → B
  f x = transp P i0 x

  g : B → A
  g y = transp ~P i0 y
```

Since `f`{.Agda} and `g`{.Agda} are defined by `transporting`{.Agda}
along a path, we can define _fillers_ `u`{.Agda} and `v`{.Agda}
connecting `f`{.Agda} (resp `g`{.Agda}) to the identity function, over
`P`{.Agda}:

```
  u : PathP (λ i → A → P i) id f
  u i x = transp (λ j → P (i ∧ j)) (~ i) x

  v : PathP (λ i → B → P i) g id
  v i y = transp (λ j → ~P ( ~ i ∧ j)) i y
```

To prove that `f`{.Agda} is an equivalence, by definition, it must have
_contractible `fibres`{.Agda ident=fibre}_. It suffices to show that the
`fibre`{.Agda} over y is inhabited, and that the fibre over y `is a
proposition`{.Agda ident=isProp}.

To prove that the `fibre`{.Agda} over `y` is inhabited, we take `g y` to
be the preimage, and prove that there is a path `f (g y) ≡ y`, as we are
required to do. For this, we use the “lid” (the dotted face) of the
square below (this is the `comp`{.Agda} term):

~~~{.quiver}
\[\begin{tikzcd}
  y && {g(y)} \\
  \\
  {f(g(y))} && y
  \arrow["{v(j,y)}", from=1-3, to=3-3]
  \arrow["{u(j,g\ y)}"', from=1-1, to=3-1]
  \arrow["{g(y)}", from=1-1, to=1-3]
  \arrow[dashed, from=3-1, to=3-3]
\end{tikzcd}\]
~~~

```
  hasFib : (y : B) → fibre f y
  hasFib y .fst = g y
  hasFib y .snd i = comp P (λ j → λ { (i = i1) → v j y
                                    ; (i = i0) → u j (g y) })
                           (g y)
```

To prove that the fibre over y is propositional, there is significantly
more work involved. Especially since all of the paths involved are
dependent, and thus none of the path operations (especially
`sym`{.Agda}!) apply. We begin by stating the types of what we're going
to construct:

```
  fibProp : (y : B) → isProp (fibre f y)
  fibProp y (x₀ , β₀) (x₁ , β₁) k = ω k , λ j → δ k (~ j) where
    ω : x₀ ≡ x₁
    δ : PathP (λ j → y ≡ f (ω j))
              (λ i → β₀ (~ i))
              (λ i → β₁ (~ i))
```

While `ω`{.Agda} is a line, `δ`{.Agda} is a _square_. Namely, by looking
at its type, we see that its boundary is the square below. Observe that
it is essentially a path `β₀ ≡ β₁`, lying over `ω`, which is exactly
what we need to equate the fibres:

~~~{.quiver}
\[\begin{tikzcd}
  y && y \\
  \\
  {f\ x_0} && {f\ x_1}
  \arrow["y", from=1-1, to=1-3]
  \arrow["{f(\omega\ i)}"', from=3-1, to=3-3]
  \arrow["{\beta_1(\neg j)}"{description}, from=1-3, to=3-3]
  \arrow["{\beta_0 (\neg j)}"{description}, from=1-1, to=3-1]
\end{tikzcd}\]
~~~

As an intermediate step in the construction of `δ`{.Agda}, we construct
`θ`{.Agda}. However, that is also hard to do directly, so we have four
(really, two) more intermediate steps: `ω₀`{.Agda}/`θ₀`{.Agda} and
`ω₁`/`θ₁`{.Agda}.

The line `ω₀`{.Agda} is the dashed line in the composition below, and
`θ₀`{.Agda} is the square itself. The type of `θ₀`{.Agda} is hard to
look at, so focus on the diagram: It connects `β₀`{.Agda} and
`ω₀`{.Agda}, in the vertical direction.

~~~{.quiver}
\[\begin{tikzcd}
  y && {f\ x_0} \\
  \\
  {g\ y} && {x_0}
  \arrow["{\omega_0}"', dashed, from=3-1, to=3-3]
  \arrow["{v(\neg i,y)}"', from=1-1, to=3-1]
  \arrow["{u(\neg i, x)}", from=1-3, to=3-3]
  \arrow["{\beta_0(\neg ~j)}", from=1-1, to=1-3]
\end{tikzcd}\]
~~~

```
    square : A → ∀ j i → PartialP (~ j ∨ j) (λ _ → P (~ i))
    square x j i (j = i0) = v (~ i) y
    square x j i (j = i1) = u (~ i) x

    ω₀ : g y ≡ x₀
    ω₀ j = comp ~P (square x₀ j) (β₀ (~ j))

    θ₀ : PathP (λ j → PathP (λ z → P (~ z)) (β₀ (~ j)) (ω₀ j))
               (λ j → v (~ j) y)
               (λ j → u (~ j) x₀)
    θ₀ j i = fill ~P (square x₀ j) (inS (β₀ (~ j))) i
```

Analogously, we have `ω₁`{.Agda} and `θ₁`{.Agda} connecting `β₁`{.Agda}
and that, as the dashed line and filler of the square below:

~~~{.quiver}
\[\begin{tikzcd}
  y && {f\ x_1} \\
  \\
  {g\ y} && {x_1}
  \arrow["{\omega_0}"', dashed, from=3-1, to=3-3]
  \arrow["{v(\neg i,y)}"', from=1-1, to=3-1]
  \arrow["{u(\neg i, x)}", from=1-3, to=3-3]
  \arrow["{\beta_1(\neg ~j)}", from=1-1, to=1-3]
\end{tikzcd}\]
~~~

```
    ω₁ : g y ≡ x₁
    ω₁ j = comp ~P (square x₁ j) (β₁ (~ j))

    θ₁ : PathP (λ j → PathP (λ z → P (~ z)) (β₁ (~ j)) (ω₁ j))
               (λ j → v (~ j) y)
               (λ j → u (~ j) x₁)
    θ₁ j i = fill ~P (square x₁ j) (inS (β₁ (~ j))) i
```

Now, we are almost done. Like a magic trick, the paths `ω₀` and
`ω₁`{.Agda} we constructed above to aid in proving `δ`{.Agda} assemble
to give a complete proof of `ω`{.Agda}, as the dashed line in the square
below:

~~~{.quiver}
\[\begin{tikzcd}
  {g\ y} && {g\ y} \\
  \\
  {x_0} && {x_1}
  \arrow["\omega"', dashed, from=3-1, to=3-3]
  \arrow["{\omega_0\ j}"', from=1-1, to=3-1]
  \arrow["{\omega _1 j}", from=1-3, to=3-3]
  \arrow["{g\ y}", from=1-1, to=1-3]
\end{tikzcd}\]
~~~

```
    sys : (k j : I) → _
    sys k j (k = i0) = ω₀ j
    sys k j (k = i1) = ω₁ j

    ω k = hcomp (sys k) (g y)
    θ : PathP (λ i → g y ≡ ω i) ω₀ ω₁
    θ k = hfill (sys k) (inS (g y))
```

We also have `θ`{.Agda}, which is the filler of the above square - i.e.,
it is a path connecting `ω₀` and `ω₁`{.Agda}. Now we can finally
assemble `δ`{.Agda}. Since we are constructing a square, we are filling
a _cube_, i.e. a path of paths of paths! The "full" face of this cube is
given by `θ`{.Agda}, which indicates the boundaries of the other faces.
The full cube is right after the definition:

```
    δ k j =
      comp P
        (λ i → λ { (j = i0) → v i y
                 ; (k = i0) → θ₀ j (~ i)
                 ; (j = i1) → u i (ω k)
                 ; (k = i1) → θ₁ j (~ i) })
        (θ k j)
```

~~~{.quiver .tall-2}
\[\begin{tikzcd}
  \textcolor{rgb,255:red,171;green,43;blue,43}{y} &&&& \textcolor{rgb,255:red,171;green,43;blue,43}{y} \\
  & \textcolor{rgb,255:red,92;green,92;blue,214}{g\ y} && \textcolor{rgb,255:red,92;green,92;blue,214}{g\ y} \\
  \\
  & \textcolor{rgb,255:red,92;green,92;blue,214}{x_0} && \textcolor{rgb,255:red,92;green,92;blue,214}{x_1} \\
  \textcolor{rgb,255:red,171;green,43;blue,43}{f\ x0} &&&& \textcolor{rgb,255:red,171;green,43;blue,43}{f\ x1}
  \arrow[""{name=0, anchor=center, inner sep=0}, "\omega", color={rgb,255:red,92;green,92;blue,214}, from=4-2, to=4-4]
  \arrow[""{name=1, anchor=center, inner sep=0}, "{\omega_0\ j}", color={rgb,255:red,92;green,92;blue,214}, from=2-2, to=4-2]
  \arrow[""{name=2, anchor=center, inner sep=0}, "{\omega _1 j}"', color={rgb,255:red,92;green,92;blue,214}, from=2-4, to=4-4]
  \arrow[""{name=3, anchor=center, inner sep=0}, "{g\ y}"', color={rgb,255:red,92;green,92;blue,214}, from=2-2, to=2-4]
  \arrow[""{name=4, anchor=center, inner sep=0}, "{\beta_0\ (\neg i)}"', color={rgb,255:red,171;green,43;blue,43}, from=1-1, to=5-1]
  \arrow[""{name=5, anchor=center, inner sep=0}, "{\beta_1\ (\neg i)}", color={rgb,255:red,171;green,43;blue,43}, from=1-5, to=5-5]
  \arrow[""{name=6, anchor=center, inner sep=0}, color={rgb,255:red,171;green,43;blue,43}, from=5-1, to=5-5]
  \arrow[""{name=7, anchor=center, inner sep=0}, color={rgb,255:red,171;green,43;blue,43}, from=1-1, to=1-5]
  \arrow[from=2-2, to=1-1]
  \arrow[from=2-4, to=1-5]
  \arrow[from=4-4, to=5-5]
  \arrow[from=4-2, to=5-1]
  \arrow["{\theta_1\ j\ \neg i}"{marking}, Rightarrow, draw=none, from=2, to=5]
  \arrow["{\theta_0\ j\ \neg i}", Rightarrow, draw=none, from=1, to=4]
  \arrow["{v\ i\ y}"{description}, Rightarrow, draw=none, from=3, to=7]
  \arrow["{u\ i\ (\omega\ i)}"{description}, Rightarrow, draw=none, from=0, to=6]
  \arrow["\theta"{description}, color={rgb,255:red,92;green,92;blue,214}, Rightarrow, draw=none, from=3, to=0]
\end{tikzcd}\]
~~~

The idea behind the diagram is to piece together the three squares we
have constructed, `θ`{.Agda}, `θ₀`{.Agda} and `θ₁`{.Agda}, with the
intent of getting a composite `β₀ ≡ β₁`. The purpleish square behind is
`θ`{.Agda}; The brownish square in front is `δ`{.Agda}. Finally, putting
together the `proof of inhabitation`{.Agda ident=hasFib} and the `proof
of propositionality`{.Agda ident=fibProp}, we get the desired:
`f`{.Agda} is an equivalence.

```
lineToisEquiv : isEquiv f
lineToisEquiv .isEqv y .centre = hasFib y
lineToisEquiv .isEqv y .paths = fibProp y _

line→equiv : A ≃ B
line→equiv .fst = f
line→equiv .snd = lineToisEquiv
```