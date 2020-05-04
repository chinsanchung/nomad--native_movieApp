## React Native Movie App

### 2. API Layer

#### 2.0 API Layer part one

- api.js 파일에 axios 로 themoviedb 의 API 를 사용합니다.

```javascript
import axios from "axios";

const TMDB_KEY = "key";

const makeRequest = (path, params) =>
  axios.get(`https://api.themoviedb.org/3${path}`, {
    params: {
      ...params,
      api_key: TMDB_KEY,
    },
  });

export const movieApi = {
  nowPlaying: () => makeRequest(),
  popular: () => makeRequest(),
  upcoming: () => makeRequest(),
  search: (word) => makeRequest(),
  movie: (id) => makeRequest(),
  discover: () => makeRequest(),
};

export const tvApi = {
  today: () => makeRequest(),
  thisWeek: () => makeRequest(),
  topRated: () => makeRequest(),
  popular: () => makeRequest(),
  search: (word) => makeRequest(),
  show: (id) => makeRequest(),
};
```

#### 2.1 API Layer part two

- 각 함수에 makeRequest 를 적용하는데, 주소 url, 그리고 파라미터를 두 번째 인자로 작성합니다. 파라미터는 검색 API 의 조건들을 의미합니다.

```javascript
// 예시
const movieApi = {
  upcoming: () => makeRequest("/movie/upcoming", { region: "kr" }),
};
```

#### 2.2 Divide and Conquer

- api.js 에 getAnything 을 추가합니다. 이 함수는 axios 로 불러온 themoviedb 데이터에 결과 외에 다른 값들을 버리는 것이 목적입니다. return 값은 결과, 에러가 됩니다.
  - getAnything 으로 데이터를 선별했으니, 이제 스크린 페이지들에서 번거롭게 걸러내는 작업을 할 필요가 없어졌습니다.

```javascript
// api.js
const getAnything = async (path, params = {}) => {
  try {
    const {
      data: { results },
    } = await makeRequest(path, params);
    return [result, null];
  } catch (error) {
    return [null, error];
  }
};
```

- 작성한 api 함수들을 적용해봅니다. 우선 screens/Movies.js 입니다.
- getData 함수에 movieApi 의 메소드들을 적용합니다. 불러온 데이터를 상태에 저장해야하는데, 분리시키거나 아니면 큰 상태 하나에 저장하는 방식이 있습니다.
  - 분리시키는 방식은 여러 번 렌더링을 하게 되므로, 큰 상태에 저장하는 방식으로 하겠습니다.

```javascript
// 예시
const getData = async () => {
  const [nowPlaying, error] = await movieApi.nowPlaying();
};
useEffect(() => {
  getData();
}, []);
```

#### 2.3 Bulletproof getAnything

- getAnything 의 문제점은 themoviedb 의 API 들이 동일한 결과를 제공하지 않는다는 것입니다. (리스트는 results, 디테일은 결과 그 자체)
  - getData 함수에 data 를 받고, 리턴문을 수정해 디테일도 반영하게 했습니다.

```javascript
const {
  data: { results },
  data,
} = await makeRequest(path, params);
return [results || data, null];
```

- screens/Movies.js 에 큰 상태를 넣어 불러올 데이터를 저장하게 만듭니다.

```javascript
const [movies, setMovies] = useState({
  nowPlaying: [],
  popular: [],
  upcoming: [],
  nowPlayingError: null,
  popularError: null,
  upcomingError: null,
});
const getData = async () => {
  const [nowPlaying, nowPlayingError] = await movieApi.nowPlaying();
  const [popular, popularError] = await movieApi.popular();
  const [upcoming, upcomingError] = await movieApi.upcoming();

  setMovies({
    nowPlaying,
    popular,
    upcoming,
    nowPlayingError,
    popularError,
    upcomingError,
  });
};
```

#### 2.4 TV and Discovery Data

- TV, Discovery 페이지에도 같은 형태로 데이터를 불러옵니다.
  - 그 전에 Favourites 문구를 Discovery 로 바꿨습니다.
