> 結論：**PLGドリブンで“公開→フォーク→登録→即実行”を中核に据える。**  
> 実装は4本柱（公開レポ、埋め込み、ギャラリー、非公開=有料ゲート）から段階リリース。

---

## 🎯 方針（TL;DR）

- **成果物（公開レポ）にCTAを埋め込み、プロダクトが自ら集客する。**
    
- **可変費は前払いクレジット＋Fair Useでロック。**
    
- **再現性・SLO・法務・課金の穴を10項目で塞ぐ。**
    

---

## 🧱 技術スタック

|要素|役割|
|---|---|
|**Next.js**|`/r/[slug]`公開レポ（ISR）、OG生成（satori/og）、埋め込み、ギャラリー、料金UI、A/B配信|
|**FastAPI**|署名検証、フォーク/実行、クレジット検査、Rate Limit、SLOメトリクス発行|
|**Python+Celery+Redis**|並列検証、冪等実行、優先度キュー|
|**ClickHouse**|イベント/KPI（view/click/fork/signup/run/paywall/ab）|
|**Supabase(Postgres)**|Auth/ユーザー/戦略/フォーク/クレジット/リファラ/サブスク状態|
|**Cloudflare**|Pages/Workers、R2（strategies/results/og/data）|
|**Stripe**|月/年/四半期、Promotion Code、Webhooks（付与/失効）|
|**Sentry/LogRocket**|監視・UXトレース|
|**GitHub Actions**|CI/CD、ISR再生成、バッチ集計/クリーンアップ|

---

## 🔁 成長ループ（PLGコア）

1. **公開レポ**：検証完了→1クリック公開、OG自動生成、URL例：`/r/[slug]?t=<jwt>`
    
2. **フォーク**：**「コピーして検証」**→未ログインでも押せる→登録→**自動インポート即実行**
    
3. **埋め込み**：ブログ/Note/YouTubeカード（Allowlist＋Rate Limit）
    
4. **ギャラリー**：公開許可のみ、**フォーク数/更新日**でランク
    
5. **Referral＝Compute通貨**：紹介両者に付与（決済後追加）
    
6. **Programmatic SEO**：`/backtest/{symbol}/{tf}/{indicator}/{period}` ISR量産
    
7. **Retention**：週次ダイジェスト（あなた vs コミュ中央値）→再シェア
    
8. **ゲート**：**非公開保存=有料**、並列/高速/大容量は段階ロック
    

---

## 💳 価格設計（改定後）

|プラン|料金|契約|自動BT/日|並列|保存|特典/制限|
|---|---|---|---|---|---|---|
|**Free**|¥0|月|3|1|10件|公開レポ可 / 非公開× / 紹介でクレジット|
|**Starter**|**¥5,480**|月 or 年(**-16%**)|20|2|100件|埋め込み◯|
|**Pro**|**¥7,980**|月 or 年(**-16%**)|60|4|1,000件|非公開◯ / 高速◯|
|**Elite**|**¥14,800**|**四半期 or 年**|**2,000回/月 同梱（Fair Use）**|8|無制限|**超過¥0.02/回**（従量）|

**追加販売**

- **Computeクレジット**：1,000回=¥1,500（前払い、繰越12ヶ月）
    
- **Speed Pack**：高速キュー優先 ¥980/月
    
- **Seats（Phase2）**：¥1,200/席/月
    

**課金運用**

- グランドファザー（既存価格維持）、7日返金保証、領収書PDF、自動税計算（JP/US優先）
    

---

## 🔐 法務・リスクディスクレーマ

- 「**過去の実績は将来を保証しない**」「教育目的」「元本割れ/レバレッジリスク」。
    
- 公開レポの**削除ポリシー/再公開禁止条項**、不正時の停止権限を規約に明記。
    

---

## 💾 データ設計（最小スキーマ）

### Supabase（概略）


```sql

create table user_quota(
  user_id uuid primary key references auth.users(id),
  compute_left int not null default 0,
  plan text not null default 'free'
);

create table strategies(
  sid uuid primary key,
  owner uuid references auth.users(id),
  title text, description text, tags text[],
  is_public bool default false,
  allow_fork bool default true,
  fork_of uuid references strategies(sid),
  version int default 1,
  updated_at timestamptz default now()
);

create table runs(
  run_id uuid primary key,
  sid uuid references strategies(sid),
  seed int,
  code_hash text,
  dataset_hash text,
  costs_profile text,
  started_at timestamptz, finished_at timestamptz,
  pf float8, winrate float8, maxdd float8, trades int
);

create table referrals(
  id bigserial primary key,
  user_id uuid not null,
  referred_user_id uuid,
  code text not null,
  reward_state text default 'pending'
);

```


### ClickHouse（イベント）


```sql
CREATE TABLE events (
  ts DateTime,
  event String,            -- view_report, click_fork_from_report, signup_from_fork, fork_success, run_started, paywall_view, embed_impression, embed_click, gallery_view, gallery_fork_click, publish_success ...
  uid String,              -- or anon_id
  sid String,
  run_id String,
  source String,           -- report|embed|gallery|app
  plan String,
  referrer String,
  ab String,
  code_hash String,
  dataset_hash String
) ENGINE = MergeTree ORDER BY (ts, event);
```

### R2（オブジェクト配置）

```bash
strategies/{sid}.json         # 条件・期間・コスト等（再現可能）
results/{sid}/{run_id}.parquet
og/{sid}.png
```


### 署名付きJWT（例）


```json
{ "sid":"...", "ver":1, "checksum":"sha256...", "owner":"...", "exp": 1699999999 }
```

---

## 🔌 API設計（要点）

**Next.js**

- `GET /r/[slug]`（ISR）：公開レポ
    
- `GET /api/og?sid=`：OG生成（フォールバックあり）
    
- `GET /embed/[sid]`：埋め込みiframe（CSP/Originチェック）
    

**FastAPI**

- `POST /api/strategies/publish`：公開＋JWT発行＋`publish_success`
    
- `POST /api/strategies/fork?t=`：署名検証→`sid2`発行→クレジット検査→enqueue
    
- `POST /api/run`：冪等キー＋クレジット減算→Celery enqueue
    
- `POST /api/referral/redeem`：コード消化→付与
    
- `POST /api/embed/token`：Allowlist検証→短寿命トークン発行
    

**Stripe Webhook**

- `checkout.session.completed`：プラン更新＋クレジット初期付与
    
- `invoice.paid`：継続付与
    
- 超過従量（Elite）はまずは月次集計→請求（Phase2でMeteredへ）
    

---

## 🔟 運用で詰まないための10項目（統合）

1. **再現性/バージョニング**：`code_hash/dataset_hash/seed`を`runs`保存、ウォークフォワードはseed固定。
    
2. **データ品質**：欠損/スパイク自動検知、TZ/夏時間固定、コスト（spread/slip/fee/swap）を**プラン別プリセット**。
    
3. **SLO/監視**：`enqueue p95<2s` / `run_start p95<10s` / `small-run p95<30s`、Sentryに**Correlation-ID**貫通。
    
4. **キュー/スロットリング**：`free|paid|priority`分離、無料は**全体の≤10%**、ユーザー単位は**リーキーバケット**。
    
5. **フィーチャーフラグ/A-B**：`fork_autorun`, `embed_card_v2`等。全イベントに`ab`付与。
    
6. **ペイウォール例外**：**公開レポ生成は常に無料**。非公開保存時のみ確実に発火、差分検知で無駄ブロック回避。
    
7. **SEO**：sitemap/canonical/robots、**noindex切替**（非公開化時）、OG失敗時のフォールバック。
    
8. **埋め込み安全**：Origin allowlist、CSP、`postMessage`のorigin検証、短寿命`data-token`＋ドメインバインド。
    
9. **課金/税務/チャージバック**：領収書PDF、JP/US税処理、チャージバック→**即クレジット無効**。
    
10. **法務**：ディスクレーマ、公開レポの権利/削除、停止権限明示。
    

---

## 🖥️ UIフロー（要点）

- **結果画面**：`[公開][リンクコピー][埋め込み]`、公開モーダル（タイトル/説明/タグ/匿名/フォーク許可）
    
- **/r/[slug]**：OG準拠Hero＋KPI、**[コピーして検証]**（未ログイン押下可）
    
- **サインアップ**：1画面（Google/メール）→**自動インポート→即実行**
    
- **フォーク実行シート**：クレジット残高、**[今すぐ実行]**/編集、ロック表示＋アップグレード
    
- **埋め込み生成**：プレビュー（L/D）、幅/高さ、Allowlist、`<script>`コピー
    
- **ギャラリー**：フィルタ・**フォーク数/更新日/PF**並替、カードからフォーク
    
- **ペイウォール**：非公開保存モーダル、並列/高速ロックのバナー
    

---

## 📈 KPI（90日目標）

- **A1**：サインアップ→初回実行 ≤ 90秒（到達率≥70%）
    
- **P1**：公開率≥25%（Run→Publish）
    
- **F1**：公開→フォーク率≥20%
    
- **K係数**：0.25–0.40
    
- **有料CVR**：公開/フォーク体験者≥10%（全体≥5%）
    
- **SLO達成率**：各SLOで≥99%
    
- **返金率**：≤2%、チャージバック率≤0.3%
    

---

## 🚀 ロードマップ（4スプリント）

- **S1**：公開レポ＋OG＋JWT、フォークAPI（自動実行）、ClickHouse最小イベント、キュー分離
    
- **S2**：埋め込み（Allowlist/Rate）、**非公開=有料**、Stripe新価格、領収書PDF
    
- **S3**：ギャラリー、Referral（コード式→付与）、週次ダイジェスト（簡易）
    
- **S4**：Programmatic SEO、A/B基盤、SLOダッシュボード、データ品質検知
    

---

## ✅ Go/No-Go（出荷チェック）

-  **未ログイン→登録→即実行 ≤90秒**（p95実測）
    
-  `view_report→click_fork_from_report→signup_from_fork→fork_success→run_started`が**可視化**
    
-  **同一`sid`再実行で完全一致**（seed/データ固定）
    
-  **無料飽和でも有料SLO維持**（キュー分離）
    
-  冪等リトライで**二重課金/消費なし**
    
-  料金UI＝Stripe価格＝ペイウォール文言が**一致**
    
-  sitemap/OG/canonical/noindexが**自動**
    
-  規約/ディスクレーマ更新済・表示確認
    

---

## まとめ

- **PLGの導線＋価格改定＋運用10項目**で、差別化と収益の両立ができる構成。
    
- これをFigma→Issue化→実装で進めれば、**自走**に乗る。必要ならIssue雛形まで即切る。
    

ChatGPT に質問する