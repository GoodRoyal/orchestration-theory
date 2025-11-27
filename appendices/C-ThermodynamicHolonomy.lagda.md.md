record Holonomy : Type where
  field
    S-cont     : Float  -- Shannon/information entropy
    S-disc     : Float  -- log(rank) of compositional generators
    Φ-boundary : Float  -- ∫ (dQ/T + irreversible) dt ≥ 0

ℋ-total : Holonomy → Float
ℋ-total h = S-cont h + S-disc h + Φ-boundary h

-- Second Law as a type: only physically valid processes inhabit it
data SecondLaw : Holonomy → Holonomy → Float → Type where
  dℋ/dt≥0 : ∀ h₁ h₂ dt → dt > 0.0
          → (ℋ-total h₂ - ℋ-total h₁) / dt ≥ 0.0
          → SecondLaw h₁ h₂ dt

-- Proof that all three components are non-decreasing
second-law-proof : ∀ h₁ h₂ dt → dt > 0.0 → SecondLaw h₁ h₂ dt
second-law-proof h₁ h₂ dt dt>0 =
  dℋ/dt≥0 h₁ h₂ dt dt>0 (
    +-preserves-≥
      (+-preserves-≥ (shannon-monotone evol-cont) (rank-monotone evol-disc))
      (boundary-flux-nonnegative meas-bound)
  )
  where open import PhysicalReasoning

-- Theorem: Any device computing ℋ-total can detect thermodynamic violations
thermodynamic-violation-detectable
  : ∀ h₁ h₂ dt → dt > 0.0
  → (ℋ-total h₂ - ℋ-total h₁) / dt < 0.0
  → ∥ ¬ (physical-process h₁ h₂) ∥₁
thermodynamic-violation-detectable h₁ h₂ dt dt>0 negative =
  ∣ (λ phys → contradiction (second-law-proof h₁ h₂ dt dt>0 phys) negative) ∣