
## fft_hurst_fib.md

### ✪ 定義式

- `fft_peak_64 = argmax(abs(FFT(Close_{t-64:t} - mean)))`
    
- `hurst_100 = polyfit(log2(lags), log2(tau), 1)[0]`
    
    - ※ lags = 2〜19、tau = std(Close[lag:] - Close[:-lag])
        
- `fib_center_t = (High_t + Low_t)/2 - (High_t - Low_t) × 0.382`
    

### ⚙️ 数学・構造的意味

- **fft_peak**: 周波数領域における支配的な周期を抽出。相場のリズムやサイクル性を可視化
    
- **hurst指数**: 自己相関性。0.5より大きい→トレンド性、0.5より小さい→ランダム性
    
- **fib_center**: フィボナッチリトレースメントの中心点。押し目候補として使える
    

### 🧐 想定される使い方

|   |   |
|---|---|
|戦略タイプ|活用方法|
|サイクル型戦略|fft_peak_64 による周期特定 → スイングや区間抽出に活用|
|トレンド vs レンジ判断|hurst > 0.6 で順張り、< 0.4 で逆張り適正あり|
|フィボナッチ押し目狙い|fib_center との価格交差 or サポレジ転換点で反応|

### ⚠️ 限界・注意点

- fft_peak はノイズに極めて敏感 → smooth前処理推奨
    
- hurst はデータ長に強く依存＆NaN多発 → 欠損耐性注意
    
- fib_center はヒゲの影響を強く受けやすい → 時間足に依存性高
    

### 🔁 関連特徴量

- `momentum_n`: サイクル変化と一致するケースあり
    
- `volatility_n`: hurstが低い状態で高ボラなら逆張り危険
    
- `upper_wick / lower_wick`: fib_centerとの組み合わせでピンバー検出にも応用可