## 0) スコープ（更新）

- **対象**：GUI戦略定義 → 1クリックBT → 結果サマリー → **公開レポ** → **フォーク即実行**（≤90秒）
    
- **対象外（MVP）**：ギャラリー正式版/Referral/本格SEO/従量課金は**後回し**（PLGは最小ループに集中）。


# MVPの「条件設定」幅（これで十分回る）

**MVPで必ず入れる（＝最小で価値が出る縦の一筋）**

1. 市場・期間
    

- 通貨ペア：主要6（EURUSD/GBPUSD/USDJPY/USDCAD/AUDUSD/NZDUSD）
    
- タイムフレーム：M15 / H1 / H4 の3つ
    
- バックテスト期間：固定レンジ（例：直近2年、UTC固定）  
    （期間固定・仕様簡素化はMVP方針に沿う）
    

2. エントリ（最大3コンポーネントまでの**単一AND**）
    

- コンポーネント種別は以下から**最大3つ**選択：  
    a. **MAクロス**（EMA_fast > EMA_slow / < で反転）  
    b. **RSI閾値**（上抜け/下抜け）  
    c. **ブレイクアウト**（N期間高値/安値更新）
    
- 売買方向：Long / Short を別々に実行（同時は後回し）
    
- 条件ツリーは**1段**（ANDのみ、ORなし）  
    （「インジ数最大3／条件ツリー1段」はMVP制限の通り）
    

3. フィルタ
    

- 曜日フィルタ（Mon–Fri チェックボックス）
    
- 取引時間帯（UTCの開始/終了、任意）  
    （タイムゾーンはUTCに固定してまず出荷）
    

4. リスク/エグジット
    

- **SL**：ATR×k（kは選択式）
    
- **TP**：リスクリワードmR（mは選択式）
    
- **タイムストップ**：最大保持バー数
    
- **オプション**：ブレークイーブン移動 or ATRトレーリング（どちらか1つ）  
    （Exitは最小セットで良い。複雑な部分利確/ピラミッディングはPost-MVP）
    

5. 出力
    

- 指標：勝率 / PF / MaxDD / 取引回数 / 期間別エクイティ
    
- **公開レポ+OG画像**→**フォーク即実行**（≤90秒）を最優先で磨く。
    

**MVPでは外す（Post-MVP）**

- マルチ条件ツリー（AND/OR入れ子）、複合チャートパターン検出
    
- 複雑なセッション/タイムゾーンロジック、マルチシンボル同時最適化
    
- 非公開保存/並列/高度キュー制御のUI（課金機能として後追い）
    
- ギャラリー正式版・Referral・本格SEOの運用部分（MVP対象外）
# 作業順（Backend→UI）

1. **契約を固定（configスキーマ/OpenAPI）**
    

- Strategy JSON（MVP版・固定項目）：pair, timeframe, date_range(UTC固定), direction, entry[≤3], filters(days/session), exit。
    
- Pydanticモデル＆JSON Schema生成。バージョン`mvp-0`を付与。
    

2. **バックテスト・コア**
    

- OHLCVローダ、EMA/RSI/Breakout/ATR実装、シグナル→約定→評価。
    
- 決定性・再現用のseedとidempotencyハッシュ（config+コードハッシュ）。
    

3. **永続化/ジョブ管理**
    

- MinIO: `strategies/{sid}.json`, `results/{run_id}/{equity.parquet, metrics.json}`。
    
- Postgres:
    
    - `strategies(sid pk, version, config jsonb, created_at)`
        
    - `runs(run_id pk, sid fk, status, started_at, finished_at, metrics jsonb, error, idem_key, corr_id)`
        
    - `artifacts(run_id fk, kind, path)`
        
- レート制限/Idempotency-KeyをAPIで強制。
    

4. **ワーカー/キュー**
    

- Celeryで`free/paid/priority`を分離。リトライ/タイムアウト、Correlation-ID貫通。
    

5. **HTTP API**（OpenAPI公開）
    

- `POST /api/run`（body=Strategy JSON、返り値=run_id, status=pending）
    
- `GET /api/runs/{run_id}`（status/metrics/progress）
    
- `GET /api/reports/{run_id}`（公開レポJSON/OG画像URL）
    
- これをcURLでE2E検証。
    

6. **レポート生成**
    

- metrics→サマリ（勝率/PF/MaxDD/回数）と簡易エクイティPNG（OG兼用）。
    

7. **UI（最後に薄く）**
    

- 固定フォーム（項目はMVP範囲のみ）→`/api/run`→ポーリング表示→共有リンク/フォークボタン。
    
- 見た目は最小、体験（≤90秒で結果）を優先。

## 3) 非機能（SLO/セキュリティの改訂）

- **SLO**：`enqueue p95 < 2s` / `run_start p95 < 10s` / `small-run p95 < 30s` / 稼働率 99.9%。
    
- **監視**：Sentry（最低限）。**Correlation-ID** はAPI→Worker→保存まで貫通。
    
- **再現性**：`code_hash / dataset_hash / seed` を保存。結果には**署名付き再現JSON**を添付（後方互換）。
    

## 4) アーキテクチャ（差し替え）

```less
[Next.js] --HTTPS/JWT--> [FastAPI API]
                       |   \-- enqueue --> [Redis] --> [Celery Workers]
                       |                                  |
                       v                                  v
 [Postgres: users/strategies/runs/quota]           [MinIO: data/strategies/results/og]

```
- **Queue分離**：`free / paid / priority` を採用（無料は総スループットの ≤10%）。
    
- **目的**：BaaSの従量爆発を回避し、計算・データ増にリニア対応。
    

> （旧）R2/ClickHouse/Supabase表記は**MinIO/Postgres/DB内イベント**に読み替え。

## 5) データモデル（命名/保管先の統一）

- オブジェクト：`data/{dataset_hash}.(csv|parquet)`, `strategies/{sid}.json`, `results/{sid}/{run_id}.json|parquet`, `og/{sid}.png`（**MinIOに統一**）。
    

## 6) APIサーフェス（補強）

- `POST /api/run`：**Idempotency-Key 必須**、quotaチェック → `runs(status='queued')` → send_task → `202 {run_id}`。
    
- `GET /api/report/{sid}`：最新の**status**と指標を返す（未完了でも status 表示）。
    
- **Rate Limit**：`/fork 10/min/ip`、`/run 30/min/user`（プラン別）。
    

## 8) 仕様制限（MVPの割り切りは維持）

- インジ数最大3／条件ツリー1段、期間は固定レンジ、**保存は公開のみ**（非公開=有料はPost-MVP）。
    

## 9) KPI（閾値の更新）

- **A1**：初回実行到達率 ≥ 70%
    
- **F1**：公開→フォーク ≥ **20%**（←15%から更新）
    
- **TTV p95** ≤ 90秒／**失敗率 ≤ 1%**（SLO準拠）。
    

## 10) スケジュール（2週間版を最新版に馴染ませ）

- **Week 1**：GUI簡易・実行API・結果サマリ・OG/公開・**Idempotency**。
    
- **Week 2**：フォーク自動実行・日次残量ジョブ・**キュー分離**・SLO計測・本番デプロイ。
    

## 11) クオータ（初期値の整合）

- **Free**：**3回/日**（旧5→新3へ）。枯渇時は課金導線へ。
    

---