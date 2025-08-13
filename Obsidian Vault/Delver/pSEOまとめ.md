# SEO 実行計画（pSEO工場ふくむ完全版）

## 0. 目的と方針
- **目的**：ローンチ初月から検索流入の仕込みを開始し、2〜3ヶ月後に自然流入で着火 → 広告依存度を低下。
- **方針**：**Programmatic SEO（pSEO）** を中核に、品質ガード（`index/noindex` 自動切替）と **再現URL** によるCVR最大化。
- **原則**：薄いページは公開しても **noindex**、KPIが揃ったら **index** に自動昇格。

---

## 1. 用語：pSEO工場とは
- **定義**：テンプレート × パラメータ（{pair, timeframe, strategy, params}）で、SEOページを**大量生産**する仕組み。
- **公開タイミング**：システムが未完成でも実施可。  
  - **A）レシピ先行**：結果なし（noindex）でURL育成  
  - **B）ダミーデータでKPI**：既存MVPで実測KPIを入れて index  
  - **C）手動サンプル**：3〜5本の見本記事から開始

---

## 2. 実装フェーズ（今すぐ開始できる順）

### フェーズ0：技術土台（半日）
- ドメイン決定：`www.example.com`（コンテンツ） / `app.example.com`（アプリ）
- Next.js ルーティング：`/backtest/[pair]/[tf]/[strategy]`（**SSG + ISR** / `revalidate: 86400`）
- `robots.txt` / `sitemap.xml`（pSEOセグメント分割）
- **canonical** / `noindex` 切替の仕組み実装
- **OG自動生成**（PF/勝率/MaxDD/Trades を画像に埋め込む）
- 構造化データ：`BreadcrumbList`（後で `Article/Dataset` 追加）

### フェーズ1：下ごしらえ（1–3日）
- **pSEOシードCSV** 準備（主要6通貨 × 3TF × 10戦略 ≒ 180行）
- 記事テンプレ（500–800字以上）・再現手順（ボタン/URL）・免責文
- 品質判定ロジック：`kpi有無 && 本文>500 && 再現JSON有無` を満たすまで **noindex**

### フェーズ2：最小公開（今週）
- **60–100ページ** 公開（noindex 混在でOK）
- KPIが入ったページから **index** へ切替
- Search Console 登録、サイトマップ送信、重要URLは「URL 検査」で手動リクエスト

### フェーズ3：本実装と連動（並走）
- Workerが本番データで回り始めたら、**KPI充足で index を自動化**
- 新レポ生成で **ISR 再生成**（`revalidateTag` 等）

---

## 3. ページテンプレ（定型レイアウト）
1. **H1**：`{PAIR} × {TF} × {STRATEGY} バックテスト`
2. **リード**：何を検証できるか／**再現URL** で即試せる
3. **KPIカード**：PF / 勝率 / MaxDD / Trades（未充足なら空欄 + noindex）
4. **エクイティ画像 or 後日差し替え**（最初は省略可）
5. **再現手順**（ボタン：この条件で即実行 → **90秒** で体験）
6. 戦略ルール（例：SMAクロスの定義・パラメータ）
7. データ条件（期間・足種・スプレッド・手数料・TZ）
8. 免責（過去実績は将来を保証しない／再現条件の明記）

> CTA は常に **「コピーして検証（無料）」** を1点に統一。

---

## 4. pSEO シード（例）
```csv
pair,tf,strategy,short,long,fee_bps,dataset_hash,sid
USDJPY,1h,sma_cross,20,60,5,d1,usdjpysma20-60
EURUSD,1h,sma_cross,20,60,5,d1,eurusdsma20-60
GBPUSD,4h,sma_cross,10,50,5,d1,gbpusdsma10-50
```

## 5. 一括実行スクリプト（KPI埋め用・MVP対応）

```python
# scripts/batch_run.py
import csv, time, requests

API = "http://localhost:8000"

def enqueue(row):
    payload = {
        "sid": row["sid"], "seed": 42,
        "code_hash": f"sma_{row['short']}_{row['long']}",
        "dataset_hash": row["dataset_hash"]
    }
    r = requests.post(f"{API}/api/run", json=payload, timeout=10)
    r.raise_for_status()
    return r.json()["run_id"]

def wait_report(sid, tries=30):
    for _ in range(tries):
        r = requests.get(f"{API}/api/report/{sid}", timeout=10)
        if r.ok and r.json().get("status") == "finished":
            return r.json()
        time.sleep(2)
    return None

with open("seed.csv") as f:
    for row in csv.DictReader(f):
        rid = enqueue(row)
        rep = wait_report(row["sid"])
        print(row["sid"], "OK" if rep else "TIMEOUT")

```
## 6. Next.js 実装ポイント（擬似コード）

```ts
// KPI / 本文 / 再現JSON の有無で品質評価
const qualityOK = !!kpi?.pf && bodyLength > 500 && !!hasReproJson;

export const generateMetadata = () => ({
  robots: qualityOK ? { index: true, follow: true }
                    : { index: false, follow: false },
  alternates: { canonical: canonicalUrl },
  // OG画像は KPI を描画した画像を生成
});

```
- **SSG + ISR**：`export const revalidate = 86400;`（毎日再生成）
    
- **サイトマップ**：品質OKのみ列挙（noindex は除外）
    
- **OG画像**：PF/勝率/MaxDD/Trades を必ず載せる（SNS/検索のCTR改善）
    

---

## 7. KPI としきい値

- **Activation**：サインアップ → 初回実行 ≤ 90秒（到達 **≥ 70%**）
    
- **公開→フォークCVR**：**≥ 20%**（15%未満はUI/文言をA/B）
    
- **SEO**：1ページ/日あたり **0.2–0.5 セッション**が目安（3ヶ月で逓増）
    
- **Paid**（検索Exact/Phrase限定）：**Payback ≤ 2ヶ月**、**CAC ≤ LTV粗利×0.7**
    

---

## 8. 有料広告との連動（最小で強い形）

- **検索広告（Exact/Phraseのみ）**：「バックテスト 無料」「EA 検証」「FX 検証ツール」など意図強キーワードに限定
    
- **LP**：**公開レポ直着地**（KPIカード＋再現ボタン）
    
- **停止条件**：`CAC > LTV粗利×0.7` or `Signup CVR < 2%` → 即停止
    
- **YouTube In-feed**：短尺デモ → 公開レポへ（任意）
    
- 序盤の広告データから **勝ちキーワード** を抽出 → **pSEOで量産強化**
    

---

## 9. 法務・品質ガード（金融系）

- 誇大表現NG、**過去成績は将来を保証しない**を明記
    
- **再現条件（データ期間・手数料等）**を本文に記載
    
- 署名付き**再現JSON**の付与（検証可能性の担保）
    

---

## 10. オペレーション（日次/週次）

### 日次（合計目安：SEO 4h）

- pSEO公開 **15–25 URL/日**（品質OKは index、未満は noindex）
    
- 事例記事 **1本/日**（再現URL＋KPI）
    
- クリエイターDM **10件/日**（＋フォローアップ5件）
    
- Search Console：重要URLの手動リクエスト
    

### 週次

- 追加 **80–120 URL/週**（テンプレ品質維持）
    
- 事例 **2–3本/週**
    
- クリエイター契約 **5–8件/週**（公開3本/月 + 埋め込み1本/月）
    
- 広告：勝ちKW 2–3本残して他は停止、LP改善
    

---

## 11. リスクと回避

- **薄い量産で評価悪化** → noindex で公開し、品質閾値を満たしたら index 化
    
- **重複/正規化問題** → canonical 固定、URL設計の一意性
    
- **流入→体験の断絶** → 公開レポ直着地 + **「コピーして検証（無料）」** をファーストビューに
    
- **虚偽パフォーマンス** → 署名付き再現JSON / データ条件の明示
    

---

## 12. 今日からの ToDo（最短ルート）

-  ルート `/backtest/[pair]/[tf]/[strategy]` を SSG + ISR で用意
    
-  品質判定で `index/noindex` 自動切替
    
-  **seed.csv（30–60行）** を作成 → `batch_run.py` でKPI埋め
    
-  **60–100 URL** を今週公開（Search Console送信）
    
-  広告は検索 Exact/Phrase のみ小額、LPは公開レポ直着地