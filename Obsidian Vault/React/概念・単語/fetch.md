コンポーネントが表示されたタイミングで、APIからデータを取りに行くというもの。

```tsx
import { useEffect, useState } from "react"

function UserList() {
  const [users, setUsers] = useState([])

  useEffect(() => {
    const fetchData = async () => {
      const res = await fetch("https://api.example.com/users")
      const data = await res.json()
      setUsers(data)
    }

    fetchData()
  }, []) // ← 初回マウント時だけ実行

  return (
    <ul>
      {users.map(user => <li key={user.id}>{user.name}</li>)}
    </ul>
  )
}

```

これどういうことかっていうたら、シンプルにhttpsのところのデータをfetchってしてるだけ。

**注意点**
	fetchは非同期処理やから、必ず[[async]]を使う
	[[useEffect]]の中ではasync関数にできないから、中にasync関数を定義して呼び出すのが定番パターン。