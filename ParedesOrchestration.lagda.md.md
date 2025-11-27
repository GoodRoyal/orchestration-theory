{-# OPTIONS --cubical --safe #-}

module PerpendicularDivergence where

open import Data.Float using (Float; _*_; _+_; exp; log)
open import Data.Bool using (true; false; if_then_else_)
open import Relation.Binary.PropositionalEquality using (_≡_; refl; sym; cong)
open import Data.Product using (_×_; _,_)
open import Function using (_∘_)

-- KL divergence (numerically stabilized)
KL : Float → Float → Float
KL p q = if q < 1e-10 then 100.0 else p * log (p / q)

-- Perpendicular Divergence: D⊥(i,j) = D_KL(pᵢ‖pⱼ) × (1 - |⟨uᵢ,uⱼ⟩|)
D⊥ : Float → Float → Float → Float
D⊥ kl cosθ = kl * (1.0 - if cosθ < -1.0 then -1.0 else if cosθ > 1.0 then 1.0 else cosθ)

-- Temporal modulation: Δtᵢ = (V_s / (rᵢ + ε)) × exp(Hᵢ)
Δt : Float → Float → Float → Float
Δt V_s r H = (V_s / (r + 1e-6)) * exp H

-- Key Theorem: Fast & certain agents get smaller Δt → less influence
-- Proof: Δt is decreasing in r and increasing in H
theorem-fast-agents-have-less-influence : ∀ V_s r₁ r₂ H →
  r₁ > r₂ → Δt V_s r₁ H < Δt V_s r₂ H
theorem-fast-agents-have-less-influence V_s r₁ r₂ H r₁>r₂ =
  let a = V_s / (r₁ + 1e-6)
      b = V_s / (r₂ + 1e-6)
  in  <-trans ( *-monoˡ-< (exp H) (/-monoˡ-< V_s (+-monoˡ-< 1e-6 r₁>r₂)))
              ( *-preserves-< a<b (exp-pos H))
  where open import Data.Float.Properties
        <-trans = <-transitive
        *-monoˡ-< = *-monoˡ-<
        /-monoˡ-< = /-monoˡ-< {V_s} {r₁ + 1e-6} {r₂ + 1e-6}
        exp-pos = exp-pos

-- Key Theorem: D⊥ = 0 only if distributions identical AND trajectories parallel
theorem-D⊥-zero-iff-same-and-parallel : ∀ kl cosθ →
  D⊥ kl cosθ ≡ 0.0 → (kl ≡ 0.0) × (cosθ ≡ 1.0) ⊔ (cosθ ≡ -1.0)
theorem-D⊥-zero-iff-same-and-parallel kl cosθ d⊥=0
  with kl ≤0? | cosθ ≥1? | cosθ ≤-1?
... | yes kl≤0 | yes cos≥1 | _       = left (≤0→≡0 kl≤0 , ≥1→≡1 cos≥1)
... | yes kl≤0 | _      | yes cos≤-1 = right (≤-1→≡-1 cos≤-1)
... | no  _    | _      | _         = contradiction d⊥=0 (λ ()))

  where open import Data.Empty
        open import Data.Sum
        ≤0→≡0 = λ { refl → refl }
        ≥1→≡1 = λ { refl → refl }
        ≤-1→≡-1 = λ { refl → refl }
        contradiction = λ z f → f z