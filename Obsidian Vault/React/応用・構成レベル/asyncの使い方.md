


**✅ 使い方のテンプレ**
```tsx
async function fetchData() {
  const res = await fetch("https://api.example.com/data")
  const data = await res.json()
  console.log(data)
}

```
[[await]]の右にある処理が終わらないと次の行には進まないってこと。

**✅ Reactでの使い方（[[useState]]の中)**
```tsx
useEffect(() => {
  const getData = async () => {
    const res = await fetch("/api/data")
    const json = await res.json()
    setData(json)
  }

  getData()
}, [])

```
[[useEffect]]の中では直接[[async]]は使えないってことのいい例ね。


**ついでにawaitの具体例**
例えば、こんな感じでAPIにデータをリクエストするプログラミングコードがあるとする。
```tsx
const res = await fetch("https://api.example.com/data")

```

この時に、[[fetch]]の返事が来るまで、下のコードは一切動かない。なぜならそれはawaitがあるから。
つまりこういうコードがかける。
```tsx
const res = await fetch("https://api.example.com/data")
const data = await res.json()
console.log(data) // ← 絶対にundefinedにならない。ちゃんと中身あり

```

