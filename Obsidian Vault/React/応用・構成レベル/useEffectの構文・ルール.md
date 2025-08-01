### ① **[[useEffect]]は「コンポーネント関数のトップレベル」でしか使えない**

```tsx
// ✅ OK
function MyComponent() {
  useEffect(() => {
    console.log("OK")
  }, [])
}

// ❌ NG：条件分岐の中はダメ
function MyComponent() {
  if (true) {
    useEffect(() => {
      console.log("これは動かない")
    }, [])
  }
}
```
トップレベルっていうのは、例えばifとかループのネストの中に入れないってことやね。


### ② **依存配列（第二引数）には“使ってる変数”を正しく入れる**

```tsx
useEffect(() => {
  console.log(name)
}, [name]) // ✅ nameを使ってるから依存に入れる

// ❌ NG：使ってるのに依存に入れてないと警告が出る
useEffect(() => {
  console.log(name)
}, []) // → ESLintに怒られる

```
シンプルにそのまま、第二引数、つまり依存配列には使ってる変数を正しく入れる。
これができてなかったら、どの変数を監視して再発火するかを監視できなくなって、更新されない、更新されすぎみたいなことになるらしい。

### ③ **副作用の中で非同期関数（[[async]]）を直接書けない**
```tsx
// ❌ NG：これは動かない
useEffect(async () => {
  const data = await fetch("/api/data")
}, [])

// ✅ OK：関数を中に定義してから呼ぶ
useEffect(() => {
  const fetchData = async () => {
    const data = await fetch("/api/data")
  }
  fetchData()
}, [])

```
[[useEffect]]の中は同期関数が前提になってるらしいから、asyncは直接かけないらしい。
正解の例も｛｝の中に入ってるからアウトやんって感じやけど、それはアロー関数で外に出してるから大丈夫っていうイメージかな。上の例にしたら返り血を処理できずにバグる。

### ④ **[[クリーンアップ関数]]は return で返す**

```tsx
useEffect(() => {
  const id = setInterval(() => {
    console.log("tick")
  }, 1000)

  return () => {
    clearInterval(id) // ✅ アンマウント時に実行される
  }
}, [])

```
	これがないとタイマーやイベントリスナーが消えずに暴走するらしい。とりあえず目覚まし時計を止める感覚で入れておこう


## ✅ useEffectの基本構文（テンプレ）

```tsx
useEffect(() => {
  // 副作用の処理

  return () => {
    // クリーンアップ処理（省略可）
  }
}, [依存値]) // ← これが命

```
