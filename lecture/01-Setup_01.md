## React Native Movie App

### 1. Setting 1.1 ~ 1.4

#### 1.1 cacheImages

- 앱을 시작하기 위한 준비 과정이 필요합니다. 사용자가 이미지, 폰트, 아이콘 등을 볼 때 로딩하는 과정 없이 바로 보여주기 위해서입니다.
- expo 의 `AppLoading` 컴포넌트를 활용합니다. splash screen 을 보여줍니다.
  - AppLoading 의 props 로 startAsync(처음 시작할 때 수행할 함수), onFinish(끝나고 수행할 함수), onError 를 넣었습니다.
- 이미지를 불러올 cacheImages 함수를 작성했습니다.
  - React native 의 `Image.prefetch()` 기능을 사용합니다. 이미지를 미리 다운로드하는(fetch) 작업입니다. `asset` 으로 asset 에 접근합니다. (expo install expo-asset 로 우선 설치합니다.)
  - Image.prefetch, Asset.fromModule 모두 promise 를 리턴합니다. 따라서 images 배열은 promise 들의 배열입니다.

```javascript
// App.js
const cacheImages = (images) =>
  images.map((image) => {
    if (typeof image === "string") {
      return Image.prefetch(image);
    } else {
      return Asset.fromModule(image).downloadAsync();
    }
  });

export default function App() {
  const [isReady, setIsReady] = useState(false);
  const loadAssets = async () => {
    const images = cacheImages();
  };
  const onFinish = () => setIsReady(true);

  return isReady ? null : (
    <AppLoading
      startAsync={loadAssets}
      onFinish={onFinish}
      onError={console.error}
    />
  );
}
```

#### 1.2 cacheFonts

- `expo install @expo/vector-icons` 로 설치하고, cacheFonts 함수를 추가합니다. fonts 로는 expo-font 를 사용합니다.(`expo add expo-font`)
  - fonts 를 loadAssets 에 넣은 후 promise 를 리턴해 AppLoading 컴포넌트의 startAsync 에 사용하도록 합니다.

```javascript
// App.js
import * as Font from "expo-font";
import { Ionicons } from "@expo/vector-icons";

const cacheFonts = (fonts) =>
  fonts.map((font) => [Font.loadAsync(font), font.loadAsync(font)]);
export default function App() {
  const fonts = cacheFonts([Ionicons.font]);
  const loadAssets = () => {
    // ...
    const fonts = cacheFonts([Ionicons.font]);
    return Promise.all([...images, ...fonts]);
  };
}
```

#### 1.3 Stack Navigation

- 최신 버젼의 react navigate 는 작은 패키지로 조각나서, 설치할 것이 많습니다. `yarn add @react-navigation/native`, `yarn add @react-navigation/stack`, `expo install react-native-gesture-handler react-native-reanimated react-native-screens react-native-safe-area-context @react-native-community/masked-view`
- isReady 가 true 일 때의 컴포넌트를 `NavigationContainer`로 변경하고, navigation 폴더를 만들어 그 안에 stack navigation, tab navigation 을 작성합니다.

```javascript
// Stack.js
import React from "react";
import { createStackNavigator } from "@react-navigation/stack";

const Stack = createStackNavigator();

export default () => (
  <Stack.Navigator>
    <Stack.Screen name="Home" component={Home} />
    <Stack.Screen name="Detail" component={Detail} />
  </Stack.Navigator>
);
```

- screens/Home.js, screens/Detail.js 을 생성해 Stack navigation 에 쓸 스크린을 만듭니다. 지금은 임시로 텍스트만 넣었습니다.
  - 각 스크린은 navigation props 에 접근할 수 있습니다. 이 props 으로 원하는 스크린에 이동하도록 설정할 수 있습니다.

```javascript
// Home.js
import React from "react";
import { view, Text, Button } from "react-native";

export default ({ navigation }) => (
  <View>
    <Text>Home</Text>
    <Button
      title="Go to Detail"
      onPress={() => navigation.navigate("Detail")}
    />
  </View>
);
// Detail.js
export default () => (
  <View>
    <Text>Detail</Text>
  </View>
);
```

#### 1.4 Tabs Navigation

- `yarn add @react-navigation/bottom-tabs`를 설치합니다.
- navigation/Tabs.js 를 작성합니다.

```javascript
import React from "react";
import { createBottomTabNavigator } from "@react-navigation/bottom-tabs";

const Tabs = createBottomTabNavigator();

export default () => (
  <Tabs.Navigator>
    <Tabs.Screen />
  </Tabs.Navigator>
);
```

- screens 의 Home.js 를 삭제하고, Movies.js, Favs.js, Tv.js, Search.js 를 작성합니다. (Stack.js 의 Home 을 navigation/Tabs.js 로 변경합니다.)
  - 이 스크린들은 tab navigation 에 있을 것입니다. 그리고 Tab.js 로 감쌀 것입니다.
- 문제는 tab navigation 의 각 페이지로 이동했을 때, 상단의 제목(헤더)이 바뀌어야 하는데, 상단은 stack navigation 의 영역으로 바꿀 수가 없다는 것입니다. 그래서 Tab.Navigator 에서 부모인 tab navigator 의 제목을 바꿔보도록 합니다.
