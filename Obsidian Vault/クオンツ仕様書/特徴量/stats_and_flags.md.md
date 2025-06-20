### ✪ 定義式

- `mean_diff_n = Close_t - SMA_n`
    
- `corr_close_volume_20 = RollingCorr(Close_t, Volume_t, 20)`
    
- `skew_20 = RollingSkew(Close_t, 20)`
    
- `kurt_20 = RollingKurt(Close_t, 20)`
    
- `position_in_range_20 = (Close_t - Min_t) / (Max_t - Min_t + ε)`
    
- `total_energy_20 = sum(|velocity_t| over 20 periods)`
    
- `price_jump_flag = 1 if |velocity_t| > 3×mean(|velocity_t| over 100 periods) else 0`
    

### ⚙️ 意味・構造

- **mean_diff_n**: 現在価格と過去平均の乖離
    
- **corr_close_volume**: 価格と出来高の同期性
    
- **skew/kurt**: 分布の歪み・尖り → 異常値兆候 or 一方向性検出
    
- **position_in_range**: 過去20本レンジ内のどこにいるか（相対位置）
    
- **total_energy_20**: 市場の“動きの蓄積量”。ボラ高の集約指標
    
- **price_jump_flag**: 異常的な急変をバイナリフラグで表現
    

### 🧐 想定される使い方

|               |                                           |
| ------------- | ----------------------------------------- |
| 戦略タイプ         | 活用方法                                      |
| 順張り or 逆張りの補助 | mean_diff や position_in_range で“行き過ぎ”を検出  |
| ブレイクアウト検出     | corr_close_volume の急上昇 → 出来高主導の動意         |
| 異常値検知         | skew/kurt の急激な変化、price_jump_flag による急変フラグ |

### ⚠️ 限界・注意点

- skew/kurt は欠損多い、外れ値に超敏感
    
- corr_close_volume はダマシも多く、トレンド発生時以外はノイズに近い
    
- price_jump_flag は平均の基準期間次第でヒット率が大きく変動
    

### 🔁 関連特徴量

- `zscore`, `volatility`: 分布・ボラ・乖離の補完関係
    
- `contrarian_pressure`: zscore × volume に近いロジック連動性
    
- `candle_body`, `upper/lower_wick`: 実体の大きさとjumpの共鳴を観測
    

---