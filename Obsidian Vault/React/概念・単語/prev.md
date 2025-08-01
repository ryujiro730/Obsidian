前の値ってこと。
previous(前の)の略。
使い方は[[set]]よりもさらに安全に数値をプラスしたいときとかに使うんだよ。

[[useState]]の更新でよく使う
```tsx
const [count, setCount] = useState(0)

const increment = () => {
  setCount(prev => prev + 1)
}

```
これはつまり、setCountの前の値、つまり直近で最新の状態にプラス1するってこと。

基本的に、これを使うときは前の状態を見て計算や処理をしたい場合に使う。

