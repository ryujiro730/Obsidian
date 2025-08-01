## 🥇 What Wins Gold in Molecular Property Prediction

### 💡 The gold-winning stack usually looks like this:

|Component|Description|
|---|---|
|**1. Graph Neural Networks (GNN)**|Core model: GCN / GAT / MPNN / GIN — handles raw molecular structure directly|
|**2. 3D Structure (optional)**|If 3D info is available, methods like DimeNet++, SchNet, or SphereNet crush 2D-only models|
|**3. Pretrained Embeddings**|Use models pretrained on massive molecule corpora (e.g., ChemBERTa, MolBERT, GROVER)|
|**4. Data Augmentation**|Random SMILES, subgraph masking, noise injection — increases robustness|
|**5. Ensembling**|GNN + descriptor-based models (LightGBM) + transformer models|
|**6. Target-specific Loss Tuning**|Custom loss weighting for each property (e.g., weighted MAE or uncertainty-weighted loss)|
|**7. Validation Discipline**|Strong CV (scaffold split, stratified per target) — this is what avoids leaderboard traps|


[SMILES] 
   ↓
[RDKit → Graphs] → [MPNN / GATv2 / GIN with edge features]
                               ↓
     [Multi-head regression (Tg, FFV, Tc, Density, Rg)]
                               ↓
               [Custom weighted MAE loss]
                               ↓
          [Ensemble with LightGBM descriptor model]
