### ✪ 定義式

- `is_positive = 1 if Close_t > Open_t else 0`
    
- `is_negative = 1 if Close_t < Open_t else 0`
    
- `positive_streak = 連続is_positiveの累積回数`
    
- `negative_streak = 連続is_negativeの累積回数`
    
- `candle_body = abs(Close_t - Open_t)`
    
- `upper_wick = High_t - max(Open_t, Close_t)`
    
- `lower_wick = min(Open_t, Close_t) - Low_t`
    

### ⚙️ 形状的意味

- **is_positive / is_negative**: 陽線 or 陰線をバイナリで判別
    
- **streak系**: 相場の“偏り”や一方向性の継続を捉える
    
- **candle_body**: 実体の大きさ。トレンドの勢いや反発力の目安
    
- **upper/lower_wick**: ヒゲの長さ。買戻し/売り戻し圧力や急反転の兆候
    

### 🧐 想定される使い方

|   |   |
|---|---|
|戦略タイプ|活用方法|
|プライスアクション型|wickとbodyの比率 → ピンバー・包み足の検出|
|一方向継続型|positive_streak > X で順張り加速シナリオ|
|リバーサル判断|陰線連続中の陽線転換・下ヒゲ出現で反転予兆|

### ⚠️ 限界・注意点

- wickはローソク足の足種に依存（短足ほど誤認多い）
    
- streak系はランダムでも偶然連続が出る → 過信禁物
    
- 実体の大きさはボラの影響も含む → 単独使用は非推奨
    

### 🔁 関連特徴量

- `momentum_n`, `velocity`: 実体の方向性と一致するかでトレンド強度補完
    
- `fib_center`, `position_in_range`: ヒゲと位置の整合でサポレジ判断
    
- `price_jump_flag`: 長ヒゲとの併用で急変化合図に昇格