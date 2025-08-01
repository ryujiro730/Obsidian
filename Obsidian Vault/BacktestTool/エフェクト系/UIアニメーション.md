## ✅ 1. **ローディング中の挙動**

### 🎯目的：

ユーザーに「今処理してる」と知らせて離脱防止

### 🔧実装手段：

- **Skeleton** → `shadcn/ui` の Skeleton コンポーネント
    
- **Spinner** → `Loader2` アイコンを回転（`animate-spin`）
    
- **Shimmer** → Tailwind + custom CSS (`bg-gradient-to-r animate-pulse`)
    

### 💡設計方針：

- データ取得系 → Skeleton推奨（リアリティあるから）
    
- ワンボタン操作系（例：送信）→ Spinnerで即レス
    

---

## ✅ 2. **エラー時の表示方法**

### 🎯目的：

ユーザーにミスの原因と次の行動を示す

### 🔧実装手段：

- **トースト** → `@/components/ui/toast`（shadcn） or `react-hot-toast`
    
- **バナー** → `alert-destructive`（shadcn）
    
- **アイコン** → `AlertCircle`, `XCircle` など（lucide-react）
    

### 💡設計方針：

- 致命的なエラー：画面上部バナー + アイコン
    
- 軽微な失敗（例：保存失敗）→ トーストで即通知
    
- 入力バリデーション → フォーム直下に赤字で
    

---

## ✅ 3. **アクション成功時の通知**

### 🎯目的：

「やったことが反映された」というフィードバック

### 🔧実装手段：

- **トースト** → `toast.success("保存完了")`
    
- **アニメ付きモーダル** → `Dialog` + Framer Motion
    

### 💡設計方針：

- 軽い操作：トースト一択（非ブロッキング）
    
- 意味のある完了（初回登録や削除）→ モーダル + アニメで印象残す
    

---

## ✅ 4. **マイクロインタラクション**

### 🎯目的：

「反応してる感」＝UXの生命線

### 🔧実装手段：

- Tailwindで `hover:bg-muted`, `focus:ring-2`, `active:scale-95`
    
- `transition-all duration-200 ease-in-out` 常用
    
- `data-[state=open]`, `data-[state=checked]` で状態管理
    

### 💡設計方針：

- すべてのクリック可能要素に `hover` / `active` を必ず付ける
    
- ボタンの押し込みは `scale-95`で軽く沈ませると良い
    

---

## ✅ 5. **ページ遷移アニメーション**

### 🎯目的：

唐突な画面遷移を「意味ある移動」に変える

### 🔧実装手段：

- **Framer Motion + Next.jsの`<AnimatePresence>`**
    
- `motion.div` で `initial`, `animate`, `exit` を制御
    
- `layoutId` を使って前後画面で同要素をアニメ的に繋ぐ
    

### 💡設計方針：

- ページ全体なら：スライド or フェード
    
- 一部（モーダルなど）なら：スケール＋フェードが鉄板
    

---

## ✅ 6. **チャートやグラフの描画エフェクト**

### 🎯目的：

視覚的に「動き＝変化＝価値」を感じさせる

### 🔧実装手段：

- `recharts`（もしくは `Chart.js`）＋ `animate: true`
    
- 線グラフ → ドローアニメ（line-drawing）
    
- カスタムアニメ → Framer Motionとの連携も可能
    

### 💡設計方針：

- 初回表示 → 遅延付きで表示（ease-out）
    
- 更新時 → 色が変わる / 点滅することで「変わった」と伝える