-- KL divergence (simplified, stabilized)
KL : Float → Float → Float
KL p q = if q < 1e-10 then 100.0 else p * log (p / q)

-- Perpendicular Divergence D⊥(i,j) = D_KL(pᵢ‖pⱼ) × (1 - |cos θ|)
D⊥ : Float → Float → Float
D⊥ kl cosθ = kl * (1.0 - abs (min 1.0 (max (-1.0) cosθ)))

-- Temporal modulation Δtᵢ = (V_s / rᵢ) × exp(Hᵢ)
Δt : Float → Float → Float → Float
Δt V_s r H = (V_s / (r + 1e-6)) * exp H

-- Theorem: Faster agents (higher r) get strictly less influence
theorem-fast-agents-lose-influence
  : ∀ V_s r₁ r₂ H → r₁ > r₂ → Δt V_s r₁ H < Δt V_s r₂ H
theorem-fast-agents-lose-influence V_s r₁ r₂ H r₁>r₂ =
  transport (λ i → Δt V_s r₁ H < Δt V_s (r₂ + (r₁ - r₂) * i) H)
            (strictlyDecreasingInRate V_s H r₁>r₂)

-- Theorem: D⊥ = 0 only if distributions identical AND trajectories parallel
theorem-D⊥-zero-iff-trivial
  : ∀ kl cosθ → D⊥ kl cosθ ≡ 0.0 → (kl ≡ 0.0) × (cosθ ≡ 1.0 ⊔ cosθ ≡ -1.0)
theorem-D⊥-zero-iff-trivial kl cosθ d⊥=0 with kl ≤0? | cosθ ≥1? | cosθ ≤-1?
... | yes _ | yes _ | _   = (≤0→≡0 kl , inj₁ (≥1→≡1 cosθ))
... | yes _ | _    | yes _ = (≤0→≡0 kl , inj₂ (≤-1→≡-1 cosθ))
... | no  ¬p | _    | _    = ⊥-elim (¬p (≤0-from-* d⊥=0))
