<!--
```agda
open import 1Lab.Path.Reasoning
open import 1Lab.Prelude

open import Algebra.Group.Cat.FinitelyComplete
open import Algebra.Group.Instances.Integers
open import Algebra.Group.Cat.Base
open import Algebra.Group.Homotopy
open import Algebra.Group

open import Data.Set.Truncation
open import Data.Int.Universal
open import Data.Bool
open import Data.Int

open import Homotopy.Space.Circle.Base
open import Homotopy.Loopspace
```
-->

```agda
module Homotopy.Space.Circle.Properties where
```

<!--
```agda
private variable
  ℓ ℓ' : Level
  X : Type ℓ
```
-->

# Fundamental group of the circle {defines="loop-space-of-the-circle"}

We will now calculate that the first [[loop space]] of the circle at the
basepoint is *a* type of integers, i.e. it satisfies the [[universal
property of the integers]]. First, we generalise the construction of
`möbius`{.Agda} to turn an equivalence on an arbitrary type into a type
family over `S¹`{.Agda}. Transport over this family will give the
universal map $\Omega S^1 \to X$ associated with an equivalence $e : X
\simeq X$ and basepoint $x_0 : X$.

```agda
equiv→family : X ≃ X → S¹ → Type _
equiv→family {X = X} eqv base = X
equiv→family eqv (loop i)     = ua eqv i
```

We will later need the "action" associated with an equivalence valued at
a path with free endpoint $y : S^1$. Taking $y = \rm{base}$ recovers a
more vanilla notion of "action on $X$".

```agda
equiv→action : ∀ {y} (e : X ≃ X) → base ≡ y → X → equiv→family e y
equiv→action e p x = subst (equiv→family e) p x

_ : X ≃ X → base ≡ base → X → X
_ = equiv→action {y = base}
```

<!--
```agda
open Integers
interleaved mutual
```
-->

The first thing we will do is assume an elimination principle for
$\Omega S^1$, which will be used in showing uniqueness of the universal
map $\Omega S^1 \to X$ associated to an equivalence $e : X \simeq X$.
We must also equip $\Omega S^1$ with an auto-equivalence, which
corresponds in some way to taking successors: since `loop`{.Agda}
corresponds to "the number 1", the equivalence we go with is thus
"adding 1": postcomposition with the `loop`{.Agda}.

```agda
  ΩS¹-elim
    : ∀ {ℓ} (P : Path S¹ base base → Type ℓ)
    → (pr : P refl)
    → (pl : P ≃[ ∙-post-equiv loop ] P)
    → ∀ x → P x

  private
    rotΩS¹ : (base ≡ base) ≃ (base ≡ base)
    rotΩS¹ = ∙-post-equiv {x = base} loop

  ΩS¹-integers : Integers (Path S¹ base base)
  ΩS¹-integers .point   = refl
  ΩS¹-integers .rotate  = rotΩS¹
```

It is easy to see that transporting the basepoint along the family
associated to an automorphism of $X$ commutes with our chosen
automorphism of $\Omega S^1$: modulo a tactic application, it is
`refl`{.Agda}.

```agda
  ΩS¹-integers .map-out        x e l = equiv→action e l x
  ΩS¹-integers .map-out-point  x e   = Regularity.precise! refl
  ΩS¹-integers .map-out-rotate x e l = Regularity.precise! refl
```

The difficult part of the proof is showing that `equiv→action`{.Agda} is
the unique map $\Omega S^1 \to X$ with these properties. We will show
this is the case assuming first that we have an elimination principle
for $\Omega S^1$.

```agda
  ΩS¹-integers .map-out-unique f {p} {r} frefl floop = ΩS¹-elim _
    (Regularity.precise! frefl) $ over-left→over rotΩS¹ λ a →
      (f a ≡ go a)                    ≃⟨ ap-equiv r ⟩
      (r .fst (f a) ≡ r .fst (go a))  ≃⟨ ∙-pre-equiv (floop a) ⟩
      (f (a ∙ loop) ≡ r .fst (go a))  ≃⟨ ∙-post-equiv (Regularity.precise! refl) ⟩
      (f (a ∙ loop) ≡ go (a ∙ loop))  ≃∎
    where
      go : _ → _
      go l = equiv→action r l p
```

# Loop induction

We must now show the elimination principle for $\Omega S^1$ that was
promised above. Note that, while this is a path type, both of the
endpoints are fixed (here, to be constructors), so we can not directly
use path induction. Instead, we will mimic the construction of
[[induction from initiality]], turning our induction methods into a
*total algebra* $\sum_{x : \Omega S^1} P(x)$, which can be mapped into
universally.

Applying the universal map at $l : \Omega S^1$ then gives us a pair of
an index $l' : \Omega S^1$ and a proof in $P(l')$. If we have a proper
initial object, we could then show that the composite
$$\Omega S^1 \to \left(\sum_{x : \Omega S^1} P(x)\right) \xto{\pi_1} \Omega S^1$$
which defines $l'$ is an algebra map $\Omega S^1 \to \Omega S^1$, so it
must be the identity; thus $l' = l$ and we have the desired $P(l)$.
Here, however, we're trying to *show* initiality, so we'll need a
hand-crafted coherence.

We note that the induction methods for `ΩS¹-elim`{.Agda} fit together
into a basepoint and auto-equivalence of the type $\sum_{l : \Omega S^1}
P(l)$. The family associated to this action will be called
`totl`{.Agda}.

```agda
  ΩS¹-elim P pr pl l = subst P (pathβ base l) attempt where
    totl : S¹ → Type _
    totl = equiv→family (over→total rotΩS¹ pl)
```

By rotating the basepoint (given by the method $\rm{pr} : P\
\rm{refl}$), we get a value in $P$, but its type appears to be way off.
Essentially, to show that our `attempt`{.Agda} landed in the right
fibre, we would like to reduce to the case where $l = \refl$, since
there the index is essentially trivially correct.

```agda
    attempt : P (subst totl l (refl , pr) .fst)
    attempt = subst totl l (refl , pr) .snd
```

However, this statement depends critically on $l$ being a loop,
preventing us from using path induction: if $l$ is instead a path
$\rm{base} \is y$, then transport takes us to a fibre of `totl`{.Agda}
which is not a sigma type, hence not something we can project from.
To generalise this, we must define a fibrewise transformation from
`totl`{.Agda} to the based path space of $S^1$, which, at the basepoint,
is the first projection function.

This turns out to be pretty easy: using the helper function `ua→`{.Agda}
to simplify the coherence condition, we are left with filling a square
with the boundary below, which we have by the definition of path
composition: it is `∙-filler`{.Agda}.

~~~{.quiver}
\[\begin{tikzcd}[ampersand replacement=\&]
  {\rm{base}} \&\& {\rm{base}} \\
  \\
  {\rm{base}} \&\& {\rm{base}}
  \arrow["a", from=1-1, to=1-3]
  \arrow["{\rm{refl}}"', from=1-1, to=3-1]
  \arrow["{\rm{loop}}", from=1-3, to=3-3]
  \arrow["{a\bullet\rm{loop}}"', from=3-1, to=3-3]
\end{tikzcd}\]
~~~

```agda
    path : ∀ y → totl y → base ≡ y
    path = S¹-elim fst $ ua→ λ _ → ∙-filler _ loop
```

Now we have a statement which is sufficiently general to prove by path
induction: projecting the index using `path`{.Agda} from the result of
applying our universal map, even at an arbitrary based path $l :
\rm{base} \is y$, is the identity function; And, by construction, when
$y = \rm{base}$, this statement reduces to precisely the identification
between indices we were looking for.

```agda
    pathβ : ∀ y l → path y (subst totl l (refl , pr)) ≡ l
    pathβ y = J (λ y l → path y (subst totl l (refl , pr)) ≡ l)
      (transport-refl refl)
```

```agda
ΩS¹≃Int : (base ≡ base) ≃ Int
ΩS¹≃Int = Integers-unique ΩS¹-integers Int-integers

open Equiv ΩS¹≃Int renaming (to to winding ; from to loopⁿ) using () public
```

It immediately follows from this that the circle is a [[groupoid]],
since it is connected and its loop space is a set.

```agda
opaque
  S¹-is-groupoid : is-groupoid S¹
  S¹-is-groupoid = S¹-elim (S¹-elim
    (Equiv→is-hlevel 2 ΩS¹≃Int (hlevel 2)) prop!) prop!
```

<!--
```agda
instance
  H-Level-S¹ : ∀ {k} → H-Level S¹ (3 + k)
  H-Level-S¹ = basic-instance 3 S¹-is-groupoid

abstract
  loopⁿ⁺¹ : (n : Int) → loopⁿ (sucℤ n) ≡ loopⁿ n ∙ loop
  loopⁿ⁺¹ n = Int-integers .map-out-rotate refl rotΩS¹ n
```
-->

By induction, we can show that this equivalence respects group composition
(that is, $\rm{loop}^{a + b} \equiv \rm{loop}^a \bullet \rm{loop}^b$), so that we have a
proper isomorphism of groups.

```agda
  loopⁿ-+ : (a b : Int) → loopⁿ (a +ℤ b) ≡ loopⁿ a ∙ loopⁿ b
  loopⁿ-+ a = Integers.induction Int-integers
    (ap loopⁿ (+ℤ-zeror a) ∙ sym (∙-idr _))
    λ b →
      loopⁿ (a +ℤ b) ≡ loopⁿ a ∙ loopⁿ b                 ≃⟨ ap (_∙ loop) , equiv→cancellable (∙-post-equiv loop .snd) ⟩
      loopⁿ (a +ℤ b) ∙ loop ≡ (loopⁿ a ∙ loopⁿ b) ∙ loop ≃⟨ ∙-post-equiv (sym (∙-assoc _ _ _)) ⟩
      loopⁿ (a +ℤ b) ∙ loop ≡ loopⁿ a ∙ loopⁿ b ∙ loop   ≃⟨ ∙-post-equiv (ap (loopⁿ a ∙_) (sym (loopⁿ⁺¹ b))) ⟩
      loopⁿ (a +ℤ b) ∙ loop ≡ loopⁿ a ∙ loopⁿ (sucℤ b)   ≃⟨ ∙-pre-equiv (loopⁿ⁺¹ (a +ℤ b)) ⟩
      loopⁿ (sucℤ (a +ℤ b)) ≡ loopⁿ a ∙ loopⁿ (sucℤ b)   ≃⟨ ∙-pre-equiv (ap loopⁿ (+ℤ-sucr a b)) ⟩
      loopⁿ (a +ℤ sucℤ b) ≡ loopⁿ a ∙ loopⁿ (sucℤ b)     ≃∎

π₁S¹≅ℤ : π₁Groupoid.π₁ S¹∙ S¹-is-groupoid Groups.≅ ℤ
π₁S¹≅ℤ = total-iso ΩS¹≃Int $
  equiv-hom→inverse-hom {a = ℤ} {b = π₁Groupoid.π₁ S¹∙ S¹-is-groupoid}
    (Equiv.inverse ΩS¹≃Int)
    (record { pres-⋆ = loopⁿ-+ })
```

<!--
```agda
abstract
  winding-∙ : (a b : base ≡ base) → winding (a ∙ b) ≡ winding a +ℤ winding b
  winding-∙ a b = Groups.to π₁S¹≅ℤ .snd .is-group-hom.pres-⋆ a b
```
-->

Furthermore, since the loop space of the circle is a set, we automatically
get that all of its higher homotopy groups are trivial.

```agda
Ωⁿ⁺²S¹-is-contr : ∀ n → is-contr ⌞ Ωⁿ (2 + n) S¹∙ ⌟
Ωⁿ⁺²S¹-is-contr zero = is-prop∙→is-contr (hlevel 1) refl
Ωⁿ⁺²S¹-is-contr (suc n) = Path-is-hlevel 0 (Ωⁿ⁺²S¹-is-contr n)

πₙ₊₂S¹≡0 : ∀ n → πₙ₊₁ (suc n) S¹∙ ≡ Zero-group {lzero}
πₙ₊₂S¹≡0 n = ∫-Path
  (Zero-group-is-terminal _ .centre)
  (is-contr→≃ (is-contr→∥-∥₀-is-contr (Ωⁿ⁺²S¹-is-contr n)) (hlevel 0) .snd)
```
