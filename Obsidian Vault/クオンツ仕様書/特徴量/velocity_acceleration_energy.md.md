## 📌 定義式

- `velocity_t = Close_t - Close_{t-1}`
- `acceleration_t = velocity_t - velocity_{t-1}`
- `kinetic_energy_t = 0.5 × (velocity_t)^2`

---

## ⚙️ 物理的意味

- **velocity**：価格の瞬間的な変化量（方向あり）
- **acceleration**：変化量の変化。加速 or 減速を検出する指標。
- **kinetic_energy**：方向性を問わず動きの“激しさ”を示す。市場の熱量やボラティリティの代理指標。

---

## 🧠 想定される使い方

| 戦略タイプ | 活用方法 |
|------------|-----------|
| トレンド追従 | velocityが連続的に正 or 負で推移 → トレンド継続の示唆 |
| リバーサル | accelerationの反転（+ → -）がトリガーになることがある |
| 突発検出 | kinetic_energy × volume の同時急騰 → イベント検知や急変予兆 |

---

## ⚠️ 限界・注意点

- 高頻度に0に近づく → 正規化かスムージングが必要な場合あり
- velocityは「差分」であり、スケール依存（絶対値の解釈が通貨ペアごとに異なる）
- 過去の傾向が未来に続く保証はない（非定常性）

---

## 🔁 関連特徴量

- `zscore`: velocityの標準化形に近い、分布的アプローチ
- `momentum_n`: より広範な視野の価格変化（n本スパン）
- `total_energy_20`: kinetic_energyの集積型（パターン強度の判断に使える）
