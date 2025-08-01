Propsは[[component]]に渡す外部の情報ってこと。つまり引数
例えば
```tsx
type Props = {
  selectedSymbol: string
  selectedTimeframe: string
  chartHeight: number
}

export const MainChart = ({ selectedSymbol, selectedTimeframe, chartHeight }: Props) => {
  ...
}```
これらのpropsに入ってるのは外部のコンポーネント達。=の先にあるのはデータ型。
ほぼPythonの変数みたいなもん。

もっとわかりやすく言えば、
```tsx
function App() {
  return (
    <div>
      <UserCard name="山田" age={28} />
      <UserCard name="佐藤" age={35} />
    </div>
  );
}

function UserCard({ name, age }) {
  return <p>{name}さんは{age}歳です</p>;
}```
**これの出力結果は**

山田さんは28歳です
佐藤さんは35歳です

こうなる。