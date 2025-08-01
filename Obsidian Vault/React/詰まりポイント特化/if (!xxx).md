```tsx
if (!dbRef.current) { const worker = new Worker(bundle.mainWorker, { type: "module" })
```

これの場合、ifがどういう判定するのかを理解できなかった。
なぜならifが変数名になっているからだ。

しかし、実際は**dbRef.currentの中身になにも入っていなかったら**
という条件だった。

[[useRef]]の特性的にも、
- `dbRef = useRef(null)` で初期化されてるとする。
    
- [[current]] は「中身を保持する場所」。
    
- Reactの再レンダリングでも中身が保持されるけど、初期値はnull。


あとめちゃくちゃ基本的なことやけど。

## **!は否定の意味！！！**
