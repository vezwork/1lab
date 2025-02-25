<!--
```agda
open import 1Lab.Path.IdentitySystem
open import 1Lab.HLevel.Retracts
open import 1Lab.Type.Sigma
open import 1Lab.HLevel
open import 1Lab.Equiv
open import 1Lab.Path
open import 1Lab.Type

open import Data.Dec.Base
open import Data.Sum.Base
open import Data.Id.Base

import Data.Nat.Base as Nat
```
-->

```agda
module Data.Fin.Base where
```

# Finite Sets

The type `Fin`{.Agda} is the type of size `n`.
These are defined as an inductive family over `Nat`{.Agda},
such that `Fin 0` has 0 elements, `Fin 1` has 1 element, and so on.

Another way to view `Fin`{.Agda} is that it's the type of numbers
less than some upper bound. For instance, `fsuc fzero` is
of type `Fin 3`, but will _not_ typecheck as a `Fin 1`!

```agda
data Fin : Nat → Type where
  fzero : ∀ {n} → Fin (suc n)
  fsuc : ∀ {n} → Fin n → Fin (suc n)
```

Keeping with the perspective of `Fin`{.Agda} as a type of bounded
natural numbers, we provide conversion functions going back and forth.

```agda
from-nat : ∀ (n : Nat) → Fin (suc n)
from-nat zero = fzero
from-nat (suc n) = fsuc (from-nat n)

to-nat : ∀ {n : Nat} → Fin n → Nat
to-nat fzero = zero
to-nat (fsuc i) = suc (to-nat i)
```

A note of caution: because of some ✨technical reasons✨ cubical
agda cannot handle transports over indexed inductive types very well.
Instead, we define a function `cast`{.Agda} that computes on
the indices of `Fin`{.Agda}, rather than on the path.

```agda
cast : ∀ {m n} → m ≡ n → Fin m → Fin n
cast {suc m} {zero} p fzero = absurd (Nat.zero≠suc (sym p))
cast {suc m} {suc n} p fzero = fzero
cast {suc m} {zero} p (fsuc i) = absurd (Nat.zero≠suc (sym p))
cast {suc m} {suc n} p (fsuc i) = fsuc (cast (Nat.suc-inj p) i)
```

<!--
```agda
cast-is-equiv : ∀ {m n} (p : m ≡ n) → is-equiv (cast p)
cast-is-equiv =
  J (λ _ p → is-equiv (cast p)) cast-refl-is-equiv
  where
    id≡cast-refl : ∀ {n} → id ≡ cast (λ _ → n)
    id≡cast-refl {zero} i ()
    id≡cast-refl {suc n} i fzero = fzero
    id≡cast-refl {suc n} i (fsuc x) = fsuc (id≡cast-refl {n} i x)

    cast-refl-is-equiv : ∀ {n} → is-equiv (cast (λ i → n))
    cast-refl-is-equiv = subst is-equiv id≡cast-refl id-equiv
```
-->

Next, we move on to one of the most useful functions for `Fin`{.Agda}:
`strength`. This allows us to (possibly) strengthen the upper bound
on some `Fin n`.

```agda
strengthen : ∀ {n} → Fin (suc n) → Fin (suc n) ⊎ Fin n
strengthen {n = zero} fzero = inl fzero
strengthen {n = suc n} fzero = inr fzero
strengthen {n = suc n} (fsuc i) = ⊎-map fsuc fsuc (strengthen i)
```

On the other hand, `weaken`{.Agda} does the opposite: it relaxes
the upper bound on some `Fin n`, allowing us to regard it as a
`Fin (suc n)`.

```agda
weaken : ∀ {n} → Fin n → Fin (suc n)
weaken fzero = fzero
weaken (fsuc i) = fsuc (weaken i)
```

We can also relax the upper bounds if `m ≤ n`.

```agda
inject : ∀ {m n} → m Nat.≤ n → Fin m → Fin n
inject {_} {suc n} le fzero = fzero
inject {_} {suc n} (Nat.s≤s le) (fsuc i) = fsuc (inject le i)
```

## Discreteness

The proof here mirrors the one found in [`Data.Nat.Base`],
just with some slight tweaks required to handle the indexing.

[`Data.Nat.Base`]: Data.Nat.Base.html

We begin by showing that one can `distinguish`{.Agda} zero
from successor:

```agda
fzero≠fsuc : ∀ {n} {i : Fin n} → ¬ fzero ≡ fsuc i
fzero≠fsuc {n = n} path = subst distinguish path tt where
  distinguish : Fin (suc n) → Type
  distinguish fzero = ⊤
  distinguish (fsuc _) = ⊥
```

Next, we show that `fsuc` is injective. This again follows
the proof in [`Data.Nat.Base`], but some extra care must be
taken to ensure that `pred`{.Agda} is well typed!

[`Data.Nat.Base`]: Data.Nat.Base.html

```agda
fsuc-inj : ∀ {n} {i j : Fin n} → fsuc i ≡ fsuc j → i ≡ j
fsuc-inj {n = suc n} p = ap pred p
  where
    pred : Fin (suc (suc n)) → Fin (suc n)
    pred fzero = fzero
    pred (fsuc i) = i
```

Finally, we pull everything together to show that `Fin`{.Agda} is
`Discrete`{.Agda}. This is not exactly a shock (after all, `Nat`{.Agda}
is discrete), but it's useful nonetheless!

```agda
Discreteᵢ-Fin : ∀ {n} → Discreteᵢ (Fin n)
Discreteᵢ-Fin fzero fzero = yes reflᵢ
Discreteᵢ-Fin fzero (fsuc j) = no λ ()
Discreteᵢ-Fin (fsuc i) fzero = no λ ()
Discreteᵢ-Fin (fsuc i) (fsuc j) with Discreteᵢ-Fin i j
... | yes reflᵢ = yes reflᵢ
... | no ¬i≡j = no λ { reflᵢ → ¬i≡j reflᵢ }

Discrete-Fin : ∀ {n} → Discrete (Fin n)
Discrete-Fin = Discreteᵢ→discrete Discreteᵢ-Fin
```

[Hedberg's theorem] implies that `Fin`{.Agda} is a [set], i.e., it only
has trivial paths.

[Hedberg's theorem]: agda://1Lab.Path.IdentitySystem#Discrete→is-set
[set]: agda://1Lab.HLevel#is-set

```agda
Fin-is-set : ∀ {n} → is-set (Fin n)
Fin-is-set = Discrete→is-set Discrete-Fin

instance
  H-Level-Fin : ∀ {n k} → H-Level (Fin n) (2 + k)
  H-Level-Fin = basic-instance 2 Fin-is-set
```

<!--
```agda
instance
  Number-Fin : ∀ {n} → Number (Fin n)
  Number-Fin {n} .Number.Constraint k = k Nat.< n
  Number-Fin {n} .Number.fromNat k {{e}} = go k n e where
    go : ∀ k n → k Nat.< n → Fin n
    go zero (suc n) e = fzero
    go (suc k) (suc n) (Nat.s≤s e) = fsuc (go k n e)

open import Data.Nat.Base using (0≤x ; s≤s′) public

Fin-elim
  : ∀ {ℓ} (P : ∀ {n} → Fin n → Type ℓ)
  → (∀ {n} → P {suc n} fzero)
  → (∀ {i} (j : Fin i) → P j → P (fsuc j))
  → ∀ {n} (i : Fin n) → P i
Fin-elim P pfzero pfsuc fzero = pfzero
Fin-elim P pfzero pfsuc (fsuc x) = pfsuc x (Fin-elim P pfzero pfsuc x)
```
-->

## Ordering

Keeping with the view that `Fin`{.Agda} represents the type of bounded
natural numbers, we can re-use the ordering on `Nat`{.Agda} to induce an
ordering on `Fin`{.Agda}.  This lets us repurpose any lemmas on [`≤`] to
also operate on `Fin`{.Agda}.

[`≤`]: agda://Data.Nat.Base#_≤_

```agda
_≤_ : ∀ {n} → Fin n → Fin n → Type
i ≤ j = to-nat i Nat.≤ to-nat j
infix 3 _≤_

_<_ : ∀ {n} → Fin n → Fin n → Type
i < j = to-nat i Nat.< to-nat j
infix 3 _<_
```

Next, we define a pair of functions `squish`{.Agda} and `skip`{.Agda},
which are the building blocks for _all_ monotone functions between
`Fin`{.Agda}. `squish i` takes a `j : Fin (suc n)` to a `Fin n` by
mapping both `i` and `i+1` to `i`. Its counterpart `skip i` takes some
`j : Fin n` to a `Fin (suc n)` by skipping over `i` instead.

```agda
squish : ∀ {n} → Fin n → Fin (suc n) → Fin n
squish fzero fzero = fzero
squish fzero (fsuc j) = j
squish (fsuc i) fzero = fzero
squish (fsuc i) (fsuc j) = fsuc (squish i j)

skip : ∀ {n} → Fin (suc n) → Fin n → Fin (suc n)
skip fzero j = fsuc j
skip (fsuc i) fzero = fzero
skip (fsuc i) (fsuc j) = fsuc (skip i j)
```

## As a subset

While `Fin`{.Agda} is very conveniently defined as an indexed family of
types, it can also be defined as a subset of the natural numbers:
Namely, the finite ordinal $[n]$ is the same type as as $\{ x : x < n
\}$. This makes sense! Any set with $n$ elements is equivalent to any
other set with $n$ elements, and a very canonical choice is the first
$n$ values of $\bb{N}$.

```agda
ℕ< : Nat → Type
ℕ< x = Σ[ n ∈ Nat ] (n Nat.< x)

from-ℕ< : ∀ {n} → ℕ< n → Fin n
from-ℕ< {n = suc n} (zero , q) = fzero
from-ℕ< {n = suc n} (suc p , Nat.s≤s q) = fsuc (from-ℕ< (p , q))

to-ℕ< : ∀ {n} → Fin n → ℕ< n
to-ℕ< x = to-nat x , p x where
  p : ∀ {n} (x : Fin n) → suc (to-nat x) Nat.≤ n
  p {n = suc n} fzero = Nat.s≤s Nat.0≤x
  p {n = suc n} (fsuc x) = Nat.s≤s (p x)
```

## Arithmetic

```agda
weaken-≤ : ∀ {m n} → m Nat.≤ n → Fin m → Fin n
weaken-≤ {suc m} {suc n} m≤n fzero = fzero
weaken-≤ {suc m} {suc n} (Nat.s≤s m≤n) (fsuc i) = fsuc (weaken-≤ m≤n i)

fshift : ∀ {n} (m : Nat) → Fin n → Fin (m + n)
fshift zero i = i
fshift (suc m) i = fsuc (fshift m i)

opposite : ∀ {n} → Fin n → Fin n
opposite {n = suc n} fzero = from-nat n
opposite {n = suc n} (fsuc i) = weaken (opposite i)
```
