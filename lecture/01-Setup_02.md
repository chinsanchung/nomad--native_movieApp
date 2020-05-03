## React Native Movie App

### 1. Setting 1.5 ~ 1.8

#### 1.5 Updating Header part One

- react navigation 을 이용해 Tabs.Navigator 의 부모인 Stack.Screen 의 name 을 바꿔봅니다.
- 각 스크린이 가지는 navigation props 는 스크린 조종뿐만 아니라 부모 navigatior 와도 소통하게 해줍니다.
  (지금 tab navigation 은 stack navigation 의 자식이니, tab 의 navigation props 를 활용해 부모 navigation 을 수정할 수 있습니다.)
- Tabs.js 에 navigation.setOptions 를 활용해 부모 Stack 의 이름을 바꿔봅니다.

```javascript
// Tabs.js
export default ({ navigation }) => {
  navigation.setOptions({ title: "Hello" });
  // return ...
};
```

- 참고: 스크린은 navigation 뿐만 아니라 route props 도 가집니다. route 는 현재 위치(route)에 대해 알려줍니다. 이 점을 활용해 위치에 따라 헤더의 title 을 바꿀 수 있습니다.

#### 1.6 Updating Header part Two

- useLayoutEffect 의 dependency 에 route 를 넣어 위치가 바뀔 때마다 제목을 바꾸도록 합니다.
  - `route?.state?.routeNames[route.state.index]`: 처음에는 routeNames 가 없어 에러가 뜰 것입니다. 물음표(optional chaining)으로 해결합니다. optional chaining 은 if 문과 비슷하게 처리해주는 react navigation 의 신기능입니다. or 연산자로 처음 `route?.state?.routeNames[route.state.index]`이 (undefined)일 때 Movies 로 설정합니다.
- 참고: useEffect 과 달리 useLayoutEffect 은 레이아웃 변경(렌더링 작업)이 끝난 후에 작동합니다. 그래서 useLayoutEffect 을 사용했습니다.

```javascript
// Tabs.js
const getHeaderName = (route) =>
  route?.state?.routeNames[route.state.index] || "Movies";

export default ({ navigation, route }) => {
  useLayoutEffect(() => {
    navigation.setOptions({ title: getHeaderName(route) });
  }, [route]);
};
```

#### 1.7 Styling Nav

- react navigation 의 setOptions 는 강력합니다. title 뿐만 아니라, headerShown 으로 특정 페이지의 헤더를 가리거나, headerStyle 로 특정 페이지 헤더의 스타일을 바꿀 수도 있습니다.

```javascript
// 예
navigation.setOptions({
  headerStyle: {
    backgroundColor: name === "TV" ? "blue" : "white",
  },
});
```

- Stack.js 속 Stack.Navigator 의 props 'screenOption' 으로 스크린에 대한 스타일을 정할 수 있습니다.
  - Stack.Navigator 에 넣으면 스크린 전체, Stack.Screen 에 넣으면 한 스크린에서 스타일을 설정합니다.
  - headerTintColor: tint 는 기본적으로 보이는 강조색입니다.
- Stack.Navigator 의 모드를 card, modal 둘 중 하나로 할 수 있습니다.

```javascript
// Stack.js
<Stack.Navigator
  screenOptions={{
    headerStyle: {
      backgroundColor: "black",
      borderBottomColor: "black",
      shodowColor: "black",
    },
    headerTintColor: "white",
    headerBackTitleVisible: false,
  }}
></Stack.Navigator>
```

- App.js 에서 상태 바의 스타일을 light-content 로 해서 검정 배경에서도 글이 보이도록 만듭니다. `<StatusBar barStyle='light-content' />`

#### 1.8 Adding the Icons

- tabBarOptions 으로 tab 의 스타일을 지정하고, screenOption/tabBarIcon 으로 아이콘을 설정합니다.

  - tabBarIcon 은 함수 형태여야합니다. focused 는 boolean 으로 컬러를 설정했고, route 를 활용해 페이지별 이름, 아이콘을 구분했습니다.
  - iconName 변수는 플랫폼을 구분해서, ios 일 경우 ios 접두어가 붙은 아이콘을 쓰고, 아니면 md 접두어가 붙은 아이콘을 쓰도록 합니다.

```javascript
import { Ionicons } from "@expo/vector-icons";
import { Platform } from "react-native";

<Tabs.Navigator
  tabBarOptions={{
    showLabel: false,
    style: { backgroundColor: "black", borderTopColor: "black" },
  }}
  screenOptions={({ route }) => ({
    tabBarIcon: ({ focused }) => {
      let iconName = Platform.OS === "ios" ? "ios-" : "md-";
      if (route.name === "Movies") iconName += "film";
      else if (route.name === "TV") iconName += "tv";
      else if (route.name === "Search") iconName += "search";
      else if (route.name === "Favourites") iconName += "heart";
      return (
        <IonIcons
          name={iconName}
          color={focused ? "white" : "grey"}
          size={26}
        />
      );
    },
  })}
></Tabs.Navigator>;
```
