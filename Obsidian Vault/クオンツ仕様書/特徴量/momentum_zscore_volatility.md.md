
## momentum_zscore_volatility.md

### ✪ 定義式

- `momentum_n = Close_t - Close_{t-n}`
    
- `zscore_t = (Close_t - SMA_{20}) / (STD_{20} + ε)` ※ ε = 1e-9
    
- `boltzmann_drift_t = exp(-zscore_t^2)`
    
- `volatility_n = STD(Close_t, n)`
    

### ⚙️ 統計的意味

- **momentum**: 長短期のトレンドの強さを測定
    
- **zscore**: 価格が統計的中央からどれだけ離れているか
    
- **boltzmann_drift**: zscoreからの離心力を指椅 (中央へ戻ろうとする力)
    
- **volatility**: 分散度。市場の不確定性を表す
    

### 🧐 想定される使い方

|   |   |
|---|---|
|戦略タイプ|活用方法|
|トレンドフォロー|momentum_n の正値連続で入場条件|
|逆引フィルタ|zscore が 大きい → boltzmann_drift が低価 → 逆引潮の取引|
|ボラティフィルタ|volatility_n が上昇のみとしてエントリーを有効/無効分岐|

### ⚠️ 限界・注意点

- zscoreは分母が小さいと衝切値化する
    
- momentum_n は無意味な高度のノイズを含む
    
- volatility は大規模トレンドの有無で意味が変わる
    

### 🔁 関連特徴量

- `velocity`, `acceleration`: 短期変化との連携
    
- `contrarian_pressure`: zscore × volume の逆引モーメント
    
- `price_jump_flag`: volatility の高さをトリガー化した指標
    

---