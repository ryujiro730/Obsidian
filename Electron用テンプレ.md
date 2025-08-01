```bash
# 前提: Next.js は Vite 非対応。Electron で Next.js を動かすには "Webサーバ読み込み型" または "Exportして静的読み込み型" の2通りある。
# 今回は "next export" を使った静的生成 + Electron 組み込みテンプレで構成する。

# ✅ 1. Next.js プロジェクト作成（これはそのまま）
mkdir my-app
cd my-app
npm install -g pnpm
pnpm create next-app . \
  --ts \
  --app \
  --tailwind \
  --eslint \
  --src-dir \
  --import-alias "@/*"

# ✅ 2. shadcn/ui 初期化
pnpm dlx shadcn-ui@latest init \
  --tailwind-config tailwind.config.ts \
  --tsconfig tsconfig.json \
  --components src/components \
  --utils src/lib/utils.ts

# ✅ 3. よく使うコンポーネント追加
pnpm dlx shadcn-ui@latest add button card input

# ✅ 4. 必要ディレクトリ作成
mkdir -p src/app/dashboard
mkdir -p src/app/auth
mkdir -p src/app/settings
mkdir -p src/components/ui
mkdir -p src/lib
```
# ✅ 5. スターターページ作成（仮のトップページ）
echo 'export default function Page() { return <div>Hello Electron + Next.js</div> }' > src/app/page.tsx

# ✅ 6. next.config.js に export 対応を明示
cat <<EOF > next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'export',
}
module.exports = nextConfig;
EOF

# ✅ 7. Electron 関連ファイル追加
mkdir electron

# main.js: Electronエントリポイント
cat <<EOF > electron/main.js
const { app, BrowserWindow } = require('electron');
const path = require('path');

function createWindow() {
  const win = new BrowserWindow({
    width: 1280,
    height: 800,
    webPreferences: {
      contextIsolation: true,
    },
  });

  win.loadFile(path.join(__dirname, '../out/index.html'));
}

app.whenReady().then(createWindow);
EOF

# ✅ 8. package.json に Electron スクリプト追加
npx json -I -f package.json -e '
this.scripts["export"] = "next build && next export"
this.scripts["electron"] = "pnpm export && electron electron/main.js"
'

# ✅ 9. Electron を devDependency に追加
pnpm add -D electron json

# ✅ 10. 実行コマンド
# [1] Web で確認：
# pnpm dev
# [2] Electronで起動確認：
# pnpm electron

```