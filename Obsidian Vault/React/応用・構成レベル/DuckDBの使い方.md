```tsx
const bundle: duckdb.Bundle = {
  [[mainModule(DuckDB)]]: "/duckdb/duckdb-mvp.wasm",                // 本体
  [[mainWorker（DuckDB)]]: "/duckdb/duckdb-browser-mvp.worker.js",   // 処理を別スレッドに任せるJS
  [[pthreadWorker（DuckDB)]]: null                                   // 複数スレッド使うときはここにパス（今は不要）
}

```
これが基本的な使い方になってくる。
