### 📦 1. プロジェクト作成

``` bash
npx create-expo-app my-app -t expo-template-blank-typescript
cd my-app

```


### 🧱 2. UIキット導入（shadcn/ui相当）

```bash
pnpm add nativewind
```
`tailwind.config.js` を作成：
``` bash
npx tailwindcss init
```

``` js
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["./App.{js,jsx,ts,tsx}", "./src/**/*.{js,jsx,ts,tsx}"],
  theme: {
    extend: {},
  },
  plugins: [],
}

```

babel.config.jsを編集：

``` js
module.exports = function(api) {
  api.cache(true);
  return {
    presets: ['babel-preset-expo'],
    plugins: ['nativewind/babel'],
  };
};

```

✅ 2-B. コンポーネントUIキット導入

```bash
npx create-tamagui
# → Expo対応プロジェクトが生成される（React Native + Web 両対応）

```

### 🗂 3. ディレクトリ構成
``` bash
mkdir -p src/screens
mkdir -p src/components
mkdir -p src/navigation
mkdir -p src/lib
```

### 🚀 4. 画面遷移ライブラリ導入
```bash
pnpm add @react-navigation/native
pnpm add react-native-screens react-native-safe-area-context react-native-gesture-handler react-native-reanimated react-native-vector-icons react-native-status-bar-height
pnpm add @react-navigation/native-stack
```
### 💡 5. スターターページ：`src/screens/Home.tsx`
``` tsx
import { View, Text } from 'react-native'
import { Button } from 'react-native-paper'

export default function Home() {
  return (
    <View className="flex-1 items-center justify-center bg-white">
      <Text className="text-lg mb-4">ようこそ！</Text>
      <Button mode="contained" onPress={() => console.log('Pressed')}>
        始める
      </Button>
    </View>
  )
}
```
### 🧭 6. ルーティング定義：`src/navigation/AppNavigator.tsx`
``` tsx
import { NavigationContainer } from '@react-navigation/native'
import { createNativeStackNavigator } from '@react-navigation/native-stack'
import Home from '../screens/Home'

const Stack = createNativeStackNavigator()

export default function AppNavigator() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Home">
        <Stack.Screen name="Home" component={Home} />
      </Stack.Navigator>
    </NavigationContainer>
  )
}
```

### 🧩 7. `App.tsx` を編集
``` tsx
import AppNavigator from './src/navigation/AppNavigator'

export default function App() {
  return <AppNavigator />
}
```
### ✅ 最後に起動
```bash
pnpm expo start --android
```
