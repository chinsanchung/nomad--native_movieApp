## React Native Movie App

### 3. Movie Screen

##### 3.0 Home Slider part one

- Movies 페이지 상단에 슬라이더를 설치합니다. 웹을 지원하는 슬라이더 'react-native-web-swiper'를 사용합니다.
  - 슬라이더 컴포넌트는 크기 때문에 Movies.js 파일 하나를 'container precentor pattern'으로 영역을 구분하겠습니다.
  - Movies.js -> MoviesContainer.js, view 를 다루는 MoviesPresenter.js, 둘을 묶는 index.js
- MoviesContainer.js
  - 상태에 loading:false 를 추가하고, useEffect 의 setMoives 에서 true 로 바꿉니다.
  - 렌더링하던 부분을 이제 MoviesPresenter.js 로 바꿉니다.
- MoviesPresenter.js
  - styled-components 로 만든 View, Text 로 작업합니다.
  - Home Slider part two 에서 더 자세히 다룹니다.

#### 3.1 Home Slider part two

- MoviesContainer.js
  - MoviesPresenter 에 `{...movies}` 로 모든 상태를 보냅니다.
- MoviesPresenter.js
  - Swiper 에서 controlsEnabled 를 false 로 해서 버튼을 없앱니다. 그리고 loop timeout 으로 몇 초뒤에 바뀔지 설정합니다.
  - movies.loading props 를 활용, loading 중일 때는 스피너인 'ActivityIndicator'를 넣었습니다.

```javascript
const Container = styled.View`
  flex: 1;
  background-color: black;
  justify-content: center;
`;
export default ({ loading, nowPlaying }) => (
  <Container>
    {loading ? (
      <ActivityIndicator color="white" size="small" />
    ) : (
      <Header>
        <Swiper controlsEnabled={false} loop timeout={3}>
          {nowPlaying.map((movie) => (
            <Section key={movie.id}>
              <Text>{movie.original_title}</Text>
            </Section>
          ))}
        </Swiper>
      </Header>
    )}
  </Container>
);
```

#### 3.2 Slider Background

- 슬라이더 스타일을 지정할 때 코드가 길어지는 것을 막기위해 컴포넌트를 새로 만듭니다. components/Slide.js
  - 참고: component 폴더 구조를 screens 과 동일하게 만들면 검색이 편해집니다.
  - 특정 props 만들 보내기에 PropTypes 를 설치했습니다. 타입스크립트를 쓴다면 필요없습니다.
  - 포스터 배경이미지를 보여주는 스타일 컴포넌트 BG

```javascript
// components/Movies/Slide.js
const { width: WIDTH, height: HEIGHT } = Dimensions.get("screen");
const Container = styled.View`
  width: ${WIDTH}px;
  height: ${HEIGHT / 4}px;
  background-color: red;
`;
const BG = stlyed.Image`height:100%;width:100%;`;

const Slide = ({ id, title, backgroundImage, votes, overview }) => (
  <Container>
    <BG source={{ uri: apiImage(backgroundImage) }} />
  </Container>
);
Slide.propTyles = {
  id: PropTypes.number.isRequired,
  title: PropTypes.string.isRequired,
  backgroundImage: PropTypes.string.isRequired,
  votes: PropTypes.number.isRequired,
  overview: PropTypes.string.isRequired,
};
export default Slide;
```

- 슬라이드를 분해했으니 MoviesPresenter.js 를 간소화합니다. Slide props 로 데이터를 전달합니다.
- 이미지를 받기 위해 api.js 에 apiImage 함수를 만듭니다.

```javascript
export const apiImage = (path) => `https://image.tmdb.org/t/p/w500${path}`;
```

#### 3.3 Slider Content

- 슬라이더를 해당 영역에서만 움직이도록 하려면, 슬라이더의 부모 영역 fragment 를 View 로 바꾸고 너비, 높이를 설정해야 합니다.

```javascript
// MoviesPresenter.js
const { width: WIDTH, height: HEIGHT } = Dimensions.get("screen");
const SliderContainer = styled.View`
  width: ${WIDTH}ps;
  height: ${HEIGHT / 4}px;
`;
export default ({ loading, nowPlaying }) => (
  <Container>
    // 3항연산자 생략
    <>
      <SliderContainer>// Slide 생략</SliderContainer>
    </>
  </Container>
);
```

- Slide.js
  - 데이터를 꾸며 넣어봅니다. title, votes, overview 를 넣고, Content 로 감쌉니다. 폰트 색, 너비 등을 지정해서 3.4 에 추가로 작성합니다.

#### 3.4 Finishing the Slider

- 슬라이더 위에 위치할 components/Poster.js 를 만들어봅니다. 영화, 티비 등 공통적으로 쓰는 컴포넌트입니다.
  - url 하나만 있으면 됩니다. 스타일 컴포넌트로 꾸민 컴포넌트로 작성합니다.
- Movies/Slide.js 에 poster_path 를 연결해 포스터 컴포넌트로 전달합니다.
  - 포스터의 위치릘 css 로 수정하고, overview 의 글자 길이도 줄입니다.
  - 슬라이드를 클릭 시 버튼이 나오도록 TouchableOpacity 컴포넌트를 넣습니다.
  - 참고: Button 은 TouchableOpacity 와 비슷하지만, Button 은 View, Text 를 포함하고 있습니다.
