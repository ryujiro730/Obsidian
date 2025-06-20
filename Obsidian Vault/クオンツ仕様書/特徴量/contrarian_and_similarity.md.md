### ✪ 定義式

- `contrarian_pressure = -zscore × volume`
    
- `cosine_{col1}_{col2}_w{n} = cosine_similarity({col1}[t-n:t], {col2}[t-n:t])`
    

### ⚙️ 構造的意味

- **contrarian_pressure**: 出来高を伴った逆張り意識を定量化。zscore が高い + volume も高い → 買われすぎ・売られすぎシグナル
    
- **cosine_similarity**: 価格系列間の形状一致度（パターン類似性）を [-1, 1] の範囲で表現。通貨間連動性やスプレッド反転検知に有効
    

### 🧐 想定される使い方

|戦略タイプ|活用方法|
|---|---|
|逆張りシグナル|contrarian_pressure が極端な値を示す地点でリバウンド狙い|
|クロス通貨監視|通貨AとBの Close のコサイン値低下 → 分岐点・裁定タイミングの示唆|
|ペアトレード補助|cosine_similarity により同一パターンを検出 → ロング/ショートバランス構築|

### ⚠️ 限界・注意点

- contrarian_pressure は volume に依存するため、流動性の低い時間帯では無力
    
- cosine_similarity は水準を無視して形状のみを比較 → トレンド無視になる可能性あり
    
- コサイン特徴は対象ペアが多くなると計算量が爆発する
    

### 🔁 関連特徴量

- `zscore`, `volume`: contrarian_pressure の構成要素
    
- `momentum_n`: 類似パターンと momentum の方向一致が鍵
    
- `fft_peak`, `hurst`: 類似波形の周期性補足や相関強度評価に併用可