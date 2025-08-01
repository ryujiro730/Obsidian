```
	# 1. プロジェクト作成フォルダを作って移動
	mkdir my-app
	cd my-app
	
	# 2. pnpm をグローバルにインストール（初回のみ）
	npm install -g pnpm
	
	# 3. Next.js + TypeScript + Tailwind + App Router プロジェクト作成
	pnpm create next-app . `
	  --ts `
	  --app `
	  --tailwind `
	  --eslint `
	  --src-dir `
	  --import-alias "@/*"
	
	# 4. shadcn/ui を初期化
	pnpm dlx shadcn-ui@latest init `
	  --tailwind-config tailwind.config.ts `
	  --tsconfig tsconfig.json `
	  --components src/components `
	  --utils src/lib/utils.ts
	
	# 5. よく使うコンポーネントを追加
	pnpm dlx shadcn-ui@latest add button card input
	
	# 6. 必要なディレクトリを作成
	mkdir src\app\dashboard
	mkdir src\app\auth
	mkdir src\app\settings
	mkdir src\components\ui
	mkdir src\lib
	
	# 7. スターターページを作成
	notepad src\app\page.tsx


```

**`page.tsx` の中身（コピー用）**

```
import { Button } from "@/components/ui/button";

export default function Page() {
  return (
    <main className="flex h-screen justify-center items-center">
      <Button className="bg-gradient-to-r from-pink-300 to-yellow-200 text-black px-6 py-2">
        始める
      </Button>
    </main>
  );
}

```

### ✅ 最後に開発サーバー起動

```
pnpm dev

```