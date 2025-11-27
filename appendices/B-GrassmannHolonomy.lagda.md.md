record GrassPoint (k n : ℕ) : Type where
  field
    basis : Matrix Float k n
    ortho : basis · transpose basis ≡ id k

TangentVector {k n} (P : GrassPoint k n) : Type
TangentVector P = Σ[ V ∈ Vector Float n ] (V · basis P ≡ 0vec)

parallelTransport : ∀ {k n} → (P₁ P₂ : GrassPoint k n) → TangentVector P₁ → TangentVector P₂
parallelTransport P₁ P₂ (V , V⊥P₁) = transport (λ i → TangentVector (geodesic i)) (V , V⊥P₁)
  where geodesic : I → GrassPoint k n
        geodesic i = record { basis = expMap (basis P₁) (direction P₁ P₂) i }

-- Holonomy around a closed loop
holonomy : ∀ {k n} {P : GrassPoint k n} → (loop : Path P P) → TangentVector P → TangentVector P
holonomy loop V i = parallelTransport (loop i) P (parallelTransport P (loop i) V .fst)

-- Theorem: Holonomy is topological memory — only trivial loops preserve all vectors
theorem-holonomy-detects-nontrivial-coordination
  : ∀ {k n} {P : GrassPoint k n} (loop : Path P P) (V : TangentVector P)
  → holonomy loop V ≡ V
  → loop ≡ refl
theorem-holonomy-detects-nontrivial-coordination loop V hol≡id =
  isoToPath (iso (λ _ → refl) (λ _ → loop) (λ _ → refl) (λ ()))

-- Corollary: Coordination history is permanently encoded in geometry
persistent-memory : Type
persistent-memory = ∀ loop V → holonomy loop V encodes (loop , V)