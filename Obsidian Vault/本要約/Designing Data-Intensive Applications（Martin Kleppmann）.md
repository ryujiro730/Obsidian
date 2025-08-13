**結論**：DDIAは「**信頼性（Reliable）×拡張性（Scalable）×保守容易性（Maintainable）**」を満たす**データ設計の原則とパターン集**。  
“どのDBが速いか”ではなく、**ログ中心設計・複製・分割・整合性・再計算**をどう組むかが本質。

---

## 1) 目的と設計原則

- **Reliable**：障害・ネットワーク揺らぎ・人為ミスでも**正しく復元**できる
    
- **Scalable**：負荷が増えても**一貫したSLO**で伸びる（垂直/水平）
    
- **Maintainable**：**進化可能**（スキーマ変更・再計算・置換）で、**観測できる**（ログ/メトリクス/トレース）
    

---

## 2) データモデル & スキーマ進化

- **関係型 vs ドキュメント**
    
    - 多対多・集計が主なら**RDB**、階層で局所利用なら**Doc**が相性良。
        
- **“スキーマレス”は幻想**：**暗黙のスキーマ**がコードに散ると地雷。**明示スキーマ＋互換ルール**で進化させる。
    
- **互換性**
    
    - **後方互換**：旧データを新コードで読める
        
    - **前方互換**：新データを旧コードで落ちない  
        → 追加専用の**“optional fields + 既定値”**が安全。
        

---

## 3) ストレージ & インデックス

- **B-Tree**：ランダム更新に強い、典型RDBの心臓。
    
- **LSM-Tree**（Log-Structured）：追記＋後段で**コンパクション**（SSTable）→書き込み強い。
    
- **二次索引**：**ローカル**（パーティション内）/ **グローバル**（全体）で特性が違う。
    

---

## 4) 取引と隔離レベル

- **ACID**は“魔法”ではない。**どの隔離**かで挙動が変わる。
    
    - Read Committed / **Snapshot Isolation（MVCC）** / **Serializable**（最強・コスト高）
        
    - **書き込みスキュー**はSnapshotでも起こる → **SELECT…FOR UPDATE**や**チェック制約**で潰す。
        

---

## 5) 複製（Replication）

- **Single-Leader**：書き込みはリーダー、読みはフォロワ（一般的）。**ラグ**対策が要。
    
- **Multi-Leader**：各サイトで書けるが**競合解決**が必要。
    
- **Leaderless**（Dynamo型）：**クォーラム**（R+W> N）で可用性↑、**整合回復**（read repair/anti-entropy）が前提。
    
- 一貫性パターン：**Read-your-writes**、**Monotonic reads**、**Causal**（ユーザー体験に効く）。
    

---

## 6) 分割（Sharding/Partitioning）

- **レンジ分割**：範囲クエリに強いがホットスポット注意
    
- **ハッシュ分割**：分散綺麗、範囲弱い
    
- **再配置**：**コンシステントハッシュ**や**動的再バランス**
    
- 二次索引は**パーティション付き** or **グローバル**（後者は集約コスト↑）
    

---

## 7) 一貫性・合意・分散トランザクション

- **線形化（Linearizability）**：単一オブジェクトの“絶対いま”整合。遅い/高価。
    
- **2PC**：**ブロッキング**で部分失敗に弱い。**SAGA**や**アウトボックス**で代替。
    
- **合意（Consensus）**：Raft/Paxosで**リーダ選出＋ログ合意**。**メタデータ管理**に使う。
    

---

## 8) バッチ vs ストリーム

- **バッチ（再計算）**：MapReduce/Sparkで**全履歴から再生成**（真理表の再構築）。
    
- **ストリーム**：ログを**無限配列**として**状態ful演算**（集計・ウィンドウ）。
    
    - **イベント時刻** vs **処理時刻**、**ウォーターマーク**
        
    - 配信保証：**At-most-once / At-least-once / Exactly-once（実現は“効果の一意化”）**
        

---

## 9) ログ中心設計（心臓部）

- **単一の真実＝追加専用ログ**
    
    - 変更は**イベント**として追記→**派生ビュー**（マテビュー/検索/分析DB）を**非同期更新**
        
- **デュアルライト禁止**：2系統へ同時書き込みは**競走で破綻**  
    → **トランザクショナル・アウトボックス** + **CDC** + **リプレイ可能なログ**で統一
    

---

## 10) 進化と移行

- **ローリングアップグレード**：旧新の**共存期間**を設計
    
- **バックフィル**：履歴再計算の**リソース窓**を確保
    
- **Feature Flag**：データの互換性と**切り戻し**を担保
    

---

## 11) 反パターン

- “**ネットワークは信頼できる**”前提（**タイムアウト≠失敗**）
    
- **分散ロック濫用**（スプリットブレイン）
    
- **人手再送**で重複多発（**冪等化なし**）
    
- **プロセス外にしか真実がない**（ログ不在）
    

---

# あなたのプロジェクトに直結する実装指針

## A) **デュアルライト禁止 → アウトボックス + CDC**

**目的**：Supabase(Postgres)とClickHouseへ“同じ事実”を**確実に**届ける。

- **Postgres（Supabase）**
    
    
    
    ```sql
```create table outbox (
  id bigserial primary key,
  topic text not null,           -- events.run_started など
  key text not null,             -- sid/run_id 等の一意キー
  payload jsonb not null,        -- 事件の内容
  occurred_at timestamptz not null default now(),
  delivered bool not null default false
);
-- 生成側トランザクションの最後にINSERT（アプリのDBトランザクションに内包）
```

    
- **ワーカー**（Celery or Worker）
    
    1. `delivered=false`を**ポーリング** or CDC購読
        
    2. ClickHouseに**追記**
        
    3. 成功後に`delivered=true`（**冪等**：`key`でUPSERT or 重複無視）
        
- **利点**：アプリ→DBの**1コミット**で“事実が確定”。後段は**少なくとも一度**で流せばOK。
    

## B) **冪等性（Idempotency）を全APIに**

- `run`/`publish`/`fork`に**冪等キー**（`idempotency_key` or `sid+params+seed`）
    
- **重複防止テーブル**
    
    ```sql
create table idempotency_keys(
  key text primary key,
  created_at timestamptz default now()
);
```
    
- 受信時に`INSERT ON CONFLICT DO NOTHING`→既存なら**同じレスポンス**を返す。
    

## C) **キュー分離 + バックプレッシャ**

- Celery：`free` / `paid` / `priority` の**別キュー**＋別ワーカー上限
    
- **リーキーバケット**で**/run**のレート制限（プラン別）
    
- **失敗の再試行**は指数バックオフ＋**DLQ**（死隊列）
    

## D) **ClickHouseの分割設計（例）**
```sql
create table events
(
  ts DateTime,
  event LowCardinality(String),
  uid Nullable(String),
  sid Nullable(String),
  run_id Nullable(String),
  source LowCardinality(String),
  ab LowCardinality(String)
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(ts)
ORDER BY (event, ts, uid)
TTL ts + INTERVAL 180 DAY DELETE;
```


- **集計系**は別テーブル（マテビュー）で**前処理**。
    

## E) **R2（オブジェクトストレージ）の扱い**

- オブジェクトは**追記不可＝新規書き/新バージョン**で扱う（**上書き回避**）
    
- キー設計：`results/{sid}/{run_id}.parquet`（**不変**）＋`latest/{sid}.json`（**キャッシュ制御**）
    
- **整合性前提が弱い**ものとして扱い、**書き込み後の即読みに依存しない**（UIは“完了→少し待つ”UX）。
    

## F) **スキーマ進化 & 互換**

- `strategies/{sid}.json`に**`version`**と**既定値**
    
- イベント`payload`は**追加のみ**を原則に（削除/型変更は互換崩壊）
    
- **ロールアウト順**：読み手（ClickHouse集計/フロント）を先に互換対応→書き手（API）更新
    

## G) **“Exactly-once 風”の実現**

- 物理的には**At-least-once + 冪等効果**
    
    - Outbox→（少なくとも一度）→ClickHouse
        
    - ClickHouse側は`(event, key)`で**重複除去** or **AggregatingMV**で吸収
        

## H) **再計算の道を常に残す**

- 週間ランキング等は**日次バッチ**（Spark/ClickHouseでも可）で**再生成可能**に
    
- 履歴は**append-only**を保持、**TTL**で古い原始ログを整理
    

---

## すぐ使える運用チェックリスト

-  **アウトボックス**導入（デュアルライト撲滅）
    
-  `/run` `/publish` `/fork` **全APIに冪等キー**
    
-  Celery **別キュー**＋DLQ＋バックオフ
    
-  ClickHouse **PARTITION/ORDER**と**TTL**設定
    
-  R2は**不変キー**＋**最新ポインタ**の二層
    
-  スキーマは**追加互換**で運用、削除/型変更は移行手順付き
    
-  **再計算手順**（バッチ）をドキュメント化
    

---

### まとめ

DDIAの肝は「**真実はログ（1か所）に、派生は後段で**」。  
あなたの構成（FastAPI/Celery/Redis + Supabase + ClickHouse + R2）なら、  
**アウトボックス＋冪等＋別キュー＋再計算**を入れれば、**壊れにくく進化しやすい**土台が完成する。