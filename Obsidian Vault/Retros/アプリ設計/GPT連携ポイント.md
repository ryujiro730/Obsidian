## ✅ RetrosにおけるGPT連携ポイント全整理（2025年ver）

| No. | タイミング                | 目的              | 入力               | 出力                                               | 利用先                       |
| --- | -------------------- | --------------- | ---------------- | ------------------------------------------------ | ------------------------- |
| ①   | Drop投稿時              | 意味抽出・構造化        | Dropテキスト         | `tags`, `aiSummary`, `emotion`, `parsedQuestion` | 検索・Dive起点・UI装飾            |
| ②   | Timefold登録時          | 再会タイミング提案       | Dropテキスト + 投稿日時  | `scheduledAt`（日付）                                | `timefold.scheduledAt`に保存 |
| ④   | Drop再出現時（Timefold通知） | 通知文生成           | Drop内容＋日数経過      | 通知テキスト                                           | Push通知・バナー表示              |
| ⑤   | Thread拡張時            | 関連Drop推薦・類似記憶検索 | 現在のDropやタグ       | Drop候補一覧                                         | Thread構築／リフレクション          |
| ⑥   | AIとの自己対話             | 「過去の自分」と会話      | Drop or Thread全体 | GPT応答                                            | 対話UI・ReDiveモード            |
| ⑦   | Feed生成（思想Feed）       | 他人の記憶との共鳴化      | 他者Drop           | 要約・共感可能フレーズ抽出                                    | 記憶のSNS化／学習用               |
## ✅ GPTプロンプト構造（例）

---

### 🔹① 意味抽出（Drop投稿時）

txt

コピーする編集する

`You are a reflective summarizer.   Given the user's thought below, return the following: - A short emotional summary (1 sentence) - 2-3 tags (short, single words) - The type of this thought (e.g., reflection, fear, idea, question) - A possible inner question the user might be asking  Thought: "たぶん俺は誰かの思想を写してるだけや。"`

---

### 🔹② Timefold提案

txt

コピーする編集する

`You are a memory architect.   Given the following thought, suggest a future date when it would be meaningful for the user to revisit this memory.   Return only one date in YYYY-MM-DD format.  Thought: "I feel like I’m faking my progress."   Posted: 2025-08-01`

---

---

### 🔹④ 再会通知文生成（Timefold通知）

txt

コピーする編集する

`You are a poetic notification generator.   Create a short sentence (max 25 words) that tells the user that this thought is returning after a long time, with emotional resonance.  Thought: "誰かの価値観を生きてしまっている気がする。"   Days passed: 284`

---

## ✅ API設計視点重曹　

- Firestore Hook：Drop保存時 → Cloud Functionで自動GPT処理
    
- Cloud Function：/generate-timefold, /summarize-drop, /dive-steps など
    
- GPTモデル：`gpt-3.5-turbo` で十分。必要に応じて `gpt-4o`（通知系・Dive生成など）